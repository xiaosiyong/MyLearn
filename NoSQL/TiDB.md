# TiDB相关
### 背景：当前数据库面临的问题
在缩放、一致性、大数据分析、与云基础架构集成等方面均存在诸多问题，现有的数据库解决方案和大数据分析引擎解决方案基本处于割裂的状态，由于Oracle、MySQL数据库并不是面向分布式环境所涉及，因此即使勉强通过分库、分表或中间件的方式，在数据库层面做了分片，从本质上看也只是复制了相同的堆栈，而非针对分布式系统进行存储和计算优化，这正是进行跨业务查询或跨物理机查询和写入十分繁琐的本质原因。NoSQL虽然解决了数据库弹性扩展的难题，但是却放弃了数据的强一致性以及对ACID事务的支持，带来了新的问题。

### TiDB 架构

![15f15969c7cd187a99c2c2072db2ad93](../images/15f15969c7cd187a99c2c2072db2ad93.jpg)

TiDB 由分布式SQL层(TiDB)，分布式KV存储引擎(TiKV)以及管理整个集群的PD模块组成。无线水平扩展是TiDB的一大特点。

## TiDB设计的思路

### 1、为什么SQL层和存储层分开

- 对运维的友好性(有bug升级方便)
- 成本(存储和计算依赖的资源不一样)
- 更加灵活，可以使用不同的开发语言，对于无状态的计算层，选择Go语言，而对于存储层TiKV，选用Rust，稳定，无垃圾回收，不会造成抖动

### 2、一些特性

- 高度兼容MySQL，无缝迁移 
- 水平弹性扩展，加节点扩容
- 分布式事务，100%支持标准的ACID事务
- 金融级高可用，相比于传统主从（M-S）复制方案，基于Raft的多数选举协议可以提供金融级别的100%数据强一致性，大多数副本不丢失的前提下，故障自动恢复。
- HTAP解决方案（Hybrid Transactional analytical processing）
- 云原生SQL数据库、支持公有云、私有云、混合云，部署，配置，维护简单

核心特性：**水平扩展与高可用、100% OLTP，80%OLAP**

### 3、一致性

TiKV 利用 Raft 来做数据复制，每个数据变更都会落地为一条 Raft 日志，通过 Raft 的日志复制功能，将数据安全可靠地同步到 Group 的多数节点中。通过单机的 RocksDB，我们可以将数据快速地存储在磁盘上；通过 Raft，我们可以将数据复制到多台机器上，以防单机失效。数据的写入是通过 Raft 这一层的接口写入，而不是直接写 RocksDB。通过实现 Raft，我们拥有了一个分布式的 KV，现在再也不用担心某台机器挂掉了。

![一致性](../images/v2-66edd7577dcb6b9c8b69a134fa2c89d2_hd.jpg)

### 4、Region

对KV系统，数据分散在多台机器两种典型的方案，一是按Key做Hash，根据Hash值选存储节点，另一种是分Range，

连续的Key都保存在一个存储节点。TiKV默认是64M。每个Region是一个左闭右开的区间。。

1、以Region为单位，将数据分散在集群的节点上，尽量保证每个节点服务上的Region数量差不多

2、以Region为单位做Raft的复制和成员管理

TiKV 是以 Region 为单位做数据的复制，也就是一个 Region 的数据会保存多个副本，我们将每一个副本叫做一个 Replica。Replica 之间是通过 Raft 来保持数据的一致（终于提到了 Raft），一个 Region 的多个 Replica 会保存在不同的节点上，构成一个 Raft Group。其中一个 Replica 会作为这个 Group 的 Leader，其他的 Replica 作为 Follower。所有的读和写都是通过 Leader 进行，再由 Leader 复制给 Follower。

![](../images/image.png)

## TiDB的应用场景

### 1、MySQL 分片与合并

对于已经在用MySQL的业务，分库、分表、分片、中间件是常用手段，随着分片的增多，跨分片查询是一大难题。TiDB在业务层兼容MySQL的访问协议，PingCAP做了一个数据同步的工具——Syncer，它可以把TiDB作为一个MySQL Slave，将TiDB作为现有数据库的从库接在主MySQL库的后方，在这一层将数据打通，可以直接进行复杂的跨库、跨表、跨业务的实时SQL查询。过去的数据库都是一主多从，有了TiDB以后，可以反过来做到多主一从。

### 2、直接替换MySQL

在一个TiDB的数据库上，所有业务场景不需要做分库分表，所有的分布式工作都由数据库层完成。TiDB兼容MySQL协议，所以可以直接替换MySQL，而且基本做到了开箱即用，完全不用担心传统分库分表方案带来繁重的工作负担和复杂的维护成本，友好的用户界面让常规的技术人员可以高效地进行维护和管理。另外，TiDB具有NoSQL类似的扩容能力，在数据量和访问流量持续增长的情况下能够通过水平扩容提高系统的业务支撑能力，并且响应延迟稳定。

### 3、数据仓库

TiDB本身是一个分布式系统，可作为数据仓库使用。当TiDB的SQL出现瓶颈的时候，可用TiSpark，可以直接用Spark SQL 实时在TiKV上做大数据分析。

### 4、作为其他系统的模块 

TiDB是传统的存储跟计算分离的项目，底层的Key-Value层，可单独作为HBase的Replacement，同时支持跨行事务。TiDB对外提供两个API接口，一个是ACID Transaction的API，用于支持跨行事务；另一个是Raw API，可以做单行的事务，提升了整个的性能，但不提供跨行事务的ACID支持。

**TiDB、TiKV、PD**

TiDB 对每个表分配一个 TableID，每一个索引都会分配一个 IndexID，每一行分配一个 RowID（如果表有整数型的 Primary Key，那么会用 Primary Key 的值当做 RowID），其中 TableID 在整个集群内唯一，IndexID/RowID 在表内唯一，这些 ID 都是 int64 类型

Key-Value存储：

1、这是一个巨大的 Map，也就是存储的是 Key-Value pair

2、这个 Map 中的 Key-Value pair 按照 Key 的二进制顺序有序，也就是我们可以 Seek 到某一个 Key 的位置，然后不断的调用 Next 方法以递增的顺序获取比这个 Key 大的 Key-Value。TiKV把数据保存在RocksDB中，数据落地由RocksDB负责。RocksDB是单机的Key-Value Map。通过优化后的Raft协议，实现高效可靠的本地存储方案。

Raft是一个一致性协议，提供几个重要的功能：

1、Leader选举

2、成员变更

3、日志复制

TiKV利用Raft来做数据复制，每个数据变更都会落地为一条Raft日志，通过Raft复制功能，将数据安全可靠地同步到Group多数节点。通过单机的RocksDB，将数据快速存储在磁盘，通过Raft，数据复制到多台机器，防止单机失效。

**MVCC KEY+版本号**

没有MVCC之前

Key1-->Value

有了MVCC之后：

Key1-Version3 -->Value

Key1-Version2 -->Value 

**定位问题语句**

并不是所有 SLOW_QUERY 的语句都是有问题的。会造成集群整体压力增大的，是那些 process_time 很大的语句。wait_time 很大，但 process_time 很小的语句通常不是问题语句，是因为被问题语句阻塞，在执行队列等待造成的响应时间过长。

**Explain 查看执行计划**

Task 简介

目前 TiDB 的计算任务隶属于两种不同的 task：cop task 和 root task。cop task 是指使用 TiKV 中的 coprocessor 执行的计算任务，root task 是指在 TiDB 中执行的计算任务。

SQL 优化的目标之一是将计算尽可能地下推到 TiKV 中执行。TiKV 中的 coprocessor 能支持大部分 SQL 内建函数（包括聚合函数和标量函数）、SQL LIMIT操作、索引扫描和表扫描。但是，所有的 Join 操作都只能作为 root task 在 TiDB 上执行。

MySQL执行计划返回的是正在执行的查询计划，

TiDB返回的是最后执行的查询计划。



TiKV使用Raft一致性算法保证数据的安全，默认提供三个副本支持，三个副本形成了一个Raft Group。

当Client需要写入某个数据的时候，Client将操作发送给Raft Leader，在TiKV里称为Propose，Leader将操作编码成一个entry，写入到自己的Raft Log里，我们称为Append。Leader通过Raft算法将Entry复制到其他Follower上面，这个我们叫做Replicate。Follower收到Entry之后会进行Append操作，顺带告诉Leader Append成功。当Leader发现这个Entry被大多数节点Append，就认为这个Entry已经是Committed的了，然后就可以将Entry里面的操作解码出来，执行并且应用到状态机里面，这个我们称为Apply。在TiKV里，提供了Lease Read，对于Read请求，直接发给Leader，如果Leader确认自己的lease未过期，直接提供Read服务，这样就不用走一次Raft了。如果发现lease过期了，就会走一次Raft进行续租，再提供Read服务。

因为一个 Raft Group 处理的数据量有限，所以我们会将数据切分成多个 Raft Group，我们叫做 Region。切分的方式是按照 range 进行切分，也就是我们会将数据的 key 按照字节序进行排序，也就是一个无限的 sorted map，然后将其切分成一段一段（连续）的 key range，每个 key range 当成一个 Region。

两个相邻的 Region 之间不允许出现空洞，也就是前面一个 Region 的 end key 就是后一个 Region 的 start key。Region 的 range 使用的是前闭后开的模式  [start, end)，对于 key start 来说，它就属于这个 Region，但对于 end 来说，它其实属于下一个 Region。

## SQL Key Mapping

我们在 TiKV 上面构建了一个分布式数据库 TiDB，它是一个[关系型数据库](https://cloud.tencent.com/product/cdb-overview?from=10680)，所以大家需要关注的是一个关系型的 table 是如何映射到 key-value 上面的。假设我们有如下的表结构：

```js
CREATE TABLE t1 {
	id BIGINT PRIMARY KEY,
	name VARCHAR(1024),
	age BIGINT,
	content BLOB,
	UNIQUE(name),
	INDEX(age),
}
```

上面我们创建了一张表 t1，里面有四个字段，id 是主键，name 是唯一索引，age 是一个索引。那么这个表里面的数据是如何对应到 TiKV 的呢？

在 TiDB 里面，任何一张表都有一个唯一的 ID，譬如这里是 11，任何的索引也有唯一的 ID，上面 name 就是 12，age 就是 13。我们使用前缀 t 和 i 来区分表里面的 data 和 index。对于上面表 t1 来说，假设现在它有两行数据，分别是 (1, “a”, 10, “hello”) 和 (2, “b”, 12, “world”)，在 TiKV 里面，每一行数据会有不同的 key-value 对应。如下：

```js
PK
t_11_1 -> (1, “a”, 10, “hello”)
t_11_2 -> (2, “b”, 12, “world”)

Unique Name
i_12_a -> 1
i_12_b -> 2

Index Age
i_13_10_1 -> nil
i_13_12_2 -> nil
```

因为 PK 具有唯一性，所以我们可以用 t + Table ID + PK 来唯一表示一行数据，value 就是这行数据。对于 Unique 来说，也是具有唯一性的，所以我们用 i + Index ID + name 来表示，而 value 则是对应的 PK。如果两个 name 相同，就会破坏唯一性约束。当我们使用 Unique 来查询的时候，会先找到对应的 PK，然后再通过 PK 找到对应的数据。

对于普通的 Index 来说，不需要唯一性约束，所以我们使用 i + Index ID + age + PK，而 value 为空。因为 PK 一定是唯一的，所以两行数据即使 age 一样，也不会冲突。当我们使用 Index 来查询的时候，会先 seek 到第一个大于等于 i + Index ID + age 这个 key 的数据，然后看前缀是否匹配，如果匹配，则解码出对应的 PK，再从 PK 拿到实际的数据。

TiDB 在操作 TiKV 的时候需要保证操作 keys 的一致性，所以需要使用 TxnKV 模式。

