# Kafka相关

![kafka](../images/kafka学习导图.jpg)



## Kafka是什么？

Apache Kafka 是一款开源的消息引擎系统，消息引擎系统是一组规范。企业利用这组规范在不同系统之间传递语义准确的消息，实现松散耦合的异步式数据传递。

Kafka的消息编码格式：**纯二进制的字节序列**

消息引擎常见的传输协议，两种：

- **点对点模型** 也叫消息队列模型。A系统发的消息只能被B系统接收，其他任何系统都不能读取A发送的消息。
- **发布/订阅模型** 有一个主题(Topic)的概念，该模型有发送方和接收方，也称Publisher和Subscriber。多对多的关系。

Kafka同时支持这两种消息引擎模型。

Kafka的三层消息架构：

- 第一层是 主题层，每个主题可以配置M个分区，每个分区可以配置N个副本；
- 第二层是分区层，每个分区的N个副本中只能有一个充当领导者角色，对外提供服务；其他N-1个副本是追随者副本，只是提供数据冗余之用；
- 第三层是消息层，分区中包含若干条消息，每条消息的位移从0开始，依次递增。
- 最后，客户端程序只能与分区的领导者副本进行交互。

Kafka使用消息日志(Log)来保存数据，一个日志就是磁盘上一个只能追加写(Append-Only)消息的物理文件。因为只能追加写，所以避免了缓慢的随机I/O操作，改为性能较好的顺序I/O写操作，这也是实现Kafka高吞吐量特性的一个重要手段。如果不听地向一个日志写入消息，最终也会耗尽所有的磁盘空间，因此Kafka必然要定期的删除消息以回收磁盘。通过日志段(Log Segment)机制。Kafka底层，一个日志细分成多个日志段，消息被追加写到最新的日志段中，当写满一个日志段后，Kafka会自动切分出一个新的日志段，并将老的日志段封存起来。Kafka在后台有定时任务会定期地检查老的日志段是否能被删除，从而实现回收磁盘空间的目的。

Kafka的点对点消息模型，指同一条消息只能被下游的一个消费者消费，其他消费者不能染指。Kafka中实现这种P2P模型的方法是引入了消费者组(Consumer Group)。所谓的消费者组，指多个消费者实例共同组成一个组来消费一组主题。主题中的每个分区都只会被组内的一个消费者实例消费，其他消费者实例不能消费它，这样加速了消费端的吞吐量。

 如果producer指定了要发送的目标分区，消息自然是去到那个分区；否则就按照producer端参数partitioner.class指定的分区策略来定；如果你没有指定过partitioner.class，那么默认的规则是：看消息是否有key，如果有则计算key的murmur2哈希值%topic分区数；如果没有key，按照轮询的方式确定分区。

## Kafka相关概念

- Broker：Kafka 集群中包含的服务器。
- Producer：消息生产者。
- Consumer：消息消费者。
- Consumer Group：每个 Consumer 都属于一个 Consumer Group，每条消息只能被 Consumer Group 中的一个 Consumer 消费，但可以被多个 Consumer Group 消费。
- Topic：消息的类别。每条消息都属于某个 Topic，不同的 Topic 之间是相互独立的，即 Kafka 是面向 Topic 的。
- Partition：每个 Topic 分为多个 Partition，Partition 是 Kafka 分配的单位。Kafka 物理上的概念，相当于一个目录，目录下的日志文件构成这个 Partition。
- Replica：Partition 的副本，保障 Partition 的高可用。
- Leader：Replica 中的一个角色， Producer 和 Consumer 只跟 Leader 交互。
- Follower：Replica 中的一个角色，从 Leader 中复制数据。
- Controller：Kafka 集群中的其中一个服务器，用来进行 Leader Election 以及各种 Failover。
- Zookeeper：Kafka 通过 Zookeeper 来存储集群的 Meta 信息。

### Topic and Logs

Message 是按照 Topic 来组织的，每个 Topic 可以分成多个 Partition(对应 server.properties/num.partitions)。Partition 是一个顺序的追加日志，属于顺序写磁盘(顺序写磁盘效率比随机写内存要高，保障 Kafka 吞吐率)。其结构如下：server.properties/num.partitions 表示文件 server.properties 中的 num.partitions 配置项，下同。

![kafka-message](../images/kafka-message.jpg)

Partition 中的每条记录(Message)包含三个属性：Offset，messageSize 和 Data。其中 Offset 表示消息偏移量;messageSize 表示消息的大小;Data 表示消息的具体内容。Partition 是以文件的形式存储在文件系统中，位置由 server.properties/log.dirs 指定，其命名规则为 - 。比如，Topic 为"page_visits"的消息，分为 5 个 Partition，其目录结构为：

![kafka-msg-dir](../images/kafka-msg-dir.jpg)

Partition 可能位于不同的 Broker 上，Partition 是分段的，每个段是一个 Segment 文件。Segment的常用配置有：

~~~shell
#server.properties 
 
#segment文件的大小，默认为 1G 
log.segment.bytes=1024*1024*1024 
#滚动生成新的segment文件的最大时长 
log.roll.hours=24*7 
#segment文件保留的最大时长，超时将被删除 
log.retention.hours=24*7 
~~~

Partition 目录下包括了数据文件和索引文件，下图是某个 Partition 的目录结构：

![kafka-partition](../images/kafka-partition.jpg)

ndex 采用稀疏存储的方式，它不会为每一条 Message 都建立索引，而是每隔一定的字节数建立一条索引，避免索引文件占用过多的空间。缺点是没有建立索引的 Offset 不能一次定位到 Message 的位置，需要做一次顺序扫描，但是扫描的范围很小。

索引包含两个部分(均为 4 个字节的数字)，分别为相对 Offset 和 Position。相对 Offset 表示 Segment 文件中的 Offset，Position 表示 Message 在数据文件中的位置。

总结：Kafka 的 Message 存储采用了分区(Partition)，磁盘顺序读写，分段(LogSegment)和稀疏索引这几个手段来达到高效性。

### Partition and Replica

一个 Topic 物理上分为多个 Partition，位于不同的 Broker 上。如果没有 Replica，一旦 Broker 宕机，其上所有的 Patition 将不可用。每个 Partition 可以有多个Replica(对应server.properties/default.replication.factor)，分配到不同的 Broker 上。其中有一个 Leader 负责读写，处理来自 Producer 和 Consumer 的请求;其他作为 Follower 从 Leader Pull 消息，保持与 Leader 的同步。

如何分配 Partition 和 Replica 到 Broker 上?步骤如下：

- 将所有 Broker(假设共 n 个 Broker)和待分配的 Partition 排序。
- 将第 i 个 Partition 分配到第(i mod n)个 Broker 上。
- 将第 i 个 Partition 的第 j 个 Replica 分配到第((i + j) mode n)个 Broker 上。

根据上面的分配规则，若 Replica 的数量大于 Broker 的数量，必定会有两个相同的 Replica 分配到同一个 Broker 上，产生冗余。因此 Replica 的数量应该小于或等于 Broker 的数量。

### Leader选举

Kafka 在 Zookeeper 中(/brokers/topics/[topic]/partitions/[partition]/state)动态维护了一个 ISR(in-sync replicas)。ISR 里面的所有 Replica 都"跟上"了 Leader，Controller 将会从 ISR 里选一个做 Leader。

具体流程如下：

- Controller 在 Zookeeper 的 /brokers/ids/[brokerId] 节点注册 Watcher，当 Broker 宕机时 Zookeeper 会 Fire Watch。
- Controller 从 /brokers/ids 节点读取可用 Broker。
- Controller 决定 set_p，该集合包含宕机 Broker 上的所有 Partition。
- 对 set_p 中的每一个 Partition，从/brokers/topics/[topic]/partitions/[partition]/state 节点读取 ISR，决定新 Leader，将新 Leader、ISR、controller_epoch 和 leader_epoch 等信息写入 State 节点。
- 通过 RPC 向相关 Broker 发送 leaderAndISRRequest 命令。

当 ISR 为空时，会选一个 Replica(不一定是 ISR 成员)作为 Leader;当所有的 Replica 都歇菜了，会等任意一个 Replica 复活，将其作为 Leader。ISR(同步列表)中的 Follower 都"跟上"了Leader，"跟上"并不表示完全一致，它由 server.properties/replica.lag.time.max.ms 配置。表示 Leader 等待 Follower 同步消息的最大时间，如果超时，Leader 将 Follower 移除 ISR。配置项 replica.lag.max.messages 已经移除。

### Replica同步

Kafka 通过"拉模式"同步消息，即 Follower 从 Leader 批量拉取数据来同步。具体的可靠性，是由生产者(根据配置项 producer.properties/acks)来决定的。

In Kafka 0.9，request.required.acks=-1 which configration of producer is replaced by acks=all, but this old config is remained in docs.在 0.9 版本，生产者配置项 request.required.acks=-1 被 acks=all 取代，但是老的配置项还保留在文档中。PS：最新的文档 2.2.x request.required.acks 已经不存在了。

![kafka-ack](../images/kafka-ack.jpg)

在 Acks=-1 的时候，如果 ISR 少于 min.insync.replicas 指定的数目，将会抛出 NotEnoughReplicas 或 NotEnoughReplicasAfterAppend 异常。

### Producer如何发送消息

Producer 首先将消息封装进一个 ProducerRecord 实例中。

![kafka-producer](../images/kafka-producer.jpg)

消息路由：

- 发送消息时如果指定了 Partition，则直接使用。
- 如果指定了 Key，则对 Key 进行哈希，选出一个 Partition。这个 Hash(即分区机制)由 producer.properties/partitioner.class 指定的类实现，这个路由类需要实现 Partitioner 接口。
- 如果都未指定，通过 Round-Robin 来选 Partition。

消息并不会立即发送，而是先进行序列化后，发送给 Partitioner，也就是上面提到的 Hash 函数，由 Partitioner 确定目标分区后，发送到一块内存缓冲区中(发送队列)。Producer 的另一个工作线程(即 Sender 线程)，则负责实时地从该缓冲区中提取出准备好的消息封装到一个批次内，统一发送到对应的 Broker 中。

![kafka-sendmsg](../images/kafka-sendmsg.png)

### Consumer如何消费消息

每个 Consumer 都划归到一个逻辑 Consumer Group 中，一个 Partition 只能被同一个 Consumer Group 中的一个 Consumer 消费，但可以被不同的 Consumer Group 消费。若 Topic 的 Partition 数量为 p，Consumer Group 中订阅此 Topic 的 Consumer 数量为 c， 则：

1. p < c: 会有 c - p 个 consumer闲置，造成浪费 
2. p > c: 一个 consumer 对应多个 partition 
3. p = c: 一个 consumer 对应一个 partition 

应该合理分配 Consumer 和 Partition 的数量，避免造成资源倾斜，最好 Partiton 数目是 Consumer 数目的整数倍。

#### 如何将 Partition 分配给 Consumer

生产过程中 Broker 要分配 Partition，消费过程这里，也要分配 Partition 给消费者。类似 Broker 中选了一个 Controller 出来，消费也要从 Broker 中选一个 Coordinator，用于分配 Partition。当 Partition 或 Consumer 数量发生变化时，比如增加 Consumer，减少 Consumer(主动或被动)，增加 Partition，都会进行 Rebalance。

其过程如下：

- Consumer 给 Coordinator 发送 JoinGroupRequest 请求。这时其他 Consumer 发 Heartbeat 请求过来时，Coordinator 会告诉他们，要 Rebalance了。其他 Consumer 也发送 JoinGroupRequest 请求。
- Coordinator 在 Consumer 中选出一个 Leader，其他作为 Follower，通知给各个 Consumer，对于 Leader，还会把 Follower 的 Metadata 带给它。
- Consumer Leader 根据 Consumer Metadata 重新分配 Partition。
- Consumer 向 Coordinator 发送 SyncGroupRequest，其中 Leader 的 SyncGroupRequest 会包含分配的情况。Coordinator 回包，把分配的情况告诉 Consumer，包括 Leader。

#### Consumer Fetch Message

Consumer 采用"拉模式"消费消息，这样 Consumer 可以自行决定消费的行为。Consumer 调用 Poll(duration)从服务器拉取消息。拉取消息的具体行为由下面的配置项决定：

~~~shell
#consumer.properties 
 
#消费者最多 poll 多少个 record 
max.poll.records=500 
 
#消费者 poll 时 partition 返回的最大数据量 
max.partition.fetch.bytes=1048576 
 
#Consumer 最大 poll 间隔 
#超过此值服务器会认为此 consumer failed  
#并将此 consumer 踢出对应的 consumer group  
max.poll.interval.ms=300000 
~~~

在 Partition 中，每个消息都有一个 Offset。新消息会被写到 Partition 末尾(最新的一个 Segment 文件末尾)， 每个 Partition 上的消息是顺序消费的，不同的 Partition 之间消息的消费顺序是不确定的。若一个 Consumer 消费多个 Partition, 则各个 Partition 之前消费顺序是不确定的，但在每个 Partition 上是顺序消费。若来自不同 Consumer Group 的多个 Consumer 消费同一个 Partition，则各个 Consumer 之间的消费互不影响，每个 Consumer 都会有自己的 Offset。

![kafka-partition-msg](../images/kafka-partition-msg.jpg)

Consumer A 和 Consumer B 属于不同的 Consumer Group。Cosumer A 读取到 Offset=9， Consumer B 读取到 Offset=11，这个值表示下次读取的位置。也就是说 Consumer A 已经读取了 Offset 为 0~8 的消息，Consumer B 已经读取了 Offset 为 0～10 的消息。下次从 Offset=9 开始读取的 Consumer 并不一定还是 Consumer A 因为可能发生 Rebalance。

### Offset如何保存

Consumer 消费 Partition 时，需要保存 Offset 记录当前消费位置。Offset 可以选择自动提交或调用 Consumer 的 commitSync() 或 commitAsync() 手动提交，相关配置为：

~~~shell
#是否自动提交 offset 
enable.auto.commit=true 
 
#自动提交间隔。enable.auto.commit=true 时有效 
auto.commit.interval.ms=5000 
~~~

Offset 保存在名叫 __consumeroffsets 的 Topic 中。写消息的 Key 由 GroupId、Topic、Partition 组成，Value 是 Offset。一般情况下，每个 Key 的 Offset 都是缓存在内存中，查询的时候不用遍历 Partition，如果没有缓存，第一次就会遍历 Partition 建立缓存，然后查询返回。

__consumeroffsets 的 Partition 数量由下面的 Server 配置决定：

~~~
offsets.topic.num.partitions=50 
~~~

Offset 保存在哪个分区上，即 __consumeroffsets 的分区机制，可以表示为：

~~~
groupId.hashCode() mode groupMetadataTopicPartitionCount 
~~~

groupMetadataTopicPartitionCount 是上面配置的分区数。因为一个 Partition 只能被同一个 Consumer Group 的一个 Consumer 消费，因此可以用 GroupId 表示此 Consumer 消费 Offeset 所在分区。

### 消息系统可能会遇到哪些问题

Kafka 支持 3 种消息投递语义：

- at most once：最多一次，消息可能会丢失，但不会重复（获取数据 -> commit offset -> 业务处理）

- at least once：最少一次，消息不会丢失，可能会重复（获取数据 -> 业务处理 -> commit offset。）

- exactly once：只且一次，消息不丢失不重复，只且消费一次(0.11 中实现，仅限于下游也是 Kafka)

#### 如何保证消息不被重复消费?(消息的幂等性)

对于更新操作，天然具有幂等性。对于新增操作，可以给每条消息一个唯一的 id，处理前判断是否被处理过。这个 id 可以存储在 Redis 中，如果是写数据库可以用主键约束。

#### 如何保证消息的可靠性传输?(消息丢失的问题)

根据 Kafka 架构，有三个地方可能丢失消息：Consumer，Producer 和 Server。

消费端弄丢了数据：当 server.properties/enable.auto.commit 设置为 True 的时候，Kafka 会先 Commit Offset 再处理消息，如果这时候出现异常，这条消息就丢失了。

因此可以关闭自动提交 Offset，在处理完成后手动提交 Offset，这样可以保证消息不丢失;但是如果提交 Offset 失败，可能导致重复消费的问题， 这时保证幂等性即可。

Kafka 弄丢了消息：如果某个 Broker 不小心挂了，此时若 Replica 只有一个，Broker 上的消息就丢失了。若 Replica>1，给 Leader 重新选一个 Follower 作为新的 Leader，如果 Follower 还有些消息没有同步，这部分消息便丢失了。

可以进行如下配置，避免上面的问题：

- 给 Topic 设置 replication.factor 参数：这个值必须大于 1，要求每个 Partition 必须有至少 2 个副本。
- 在 Kafka 服务端设置 min.insync.replicas 参数：这个值必须大于 1，这个是要求一个 Leader 至少感知到有至少一个 Follower 还跟自己保持联系，没掉队，这样才能确保 Leader 挂了还有一个 Follower 吧。
- 在 Producer 端设置 acks=all：这个是要求每条数据，必须是写入所有 Replica 之后，才能认为是写成功了。
- 在 Producer 端设置 retries=MAX(很大很大很大的一个值，无限次重试的意思)：这个是要求一旦写入失败，就无限重试，卡在这里了。

Producer弄丢了消息：在 Producer 端设置 acks=all，保证所有的 ISR 都同步了消息才认为写入成功。

## 压缩算法

Kafka的消息层次分为两层，消息集合(message set)以及消息(message)。一个消息集合中包含若干条日志项(record item)，而日志项才是封装消息的地方。Kafka底层的消息日志由一些列消息集合日志项组成，通常不会直接操作具体的一条条消息，总是在消息集合这个层面上进行写入操作。

Kafka中，压缩可能发生在两个地方：生产者端和Broker端。生产者端配置props.put("compression.type", "gzip");这样Producer启动后生产的每个消息集合都是经GZIP压缩过的，能很好的节省网络传输带宽以及Kafka Broker端的磁盘占用。一般情况Broker从Producer端接收到消息后原封不动的保存而不会对其进行任何修改，但两种情况除外：

- Broker端指定了和Producer端不同的压缩算法，比如，Producer指定了GZIP压缩算法，Broker指定了Snappy算法。Broker端也有一个参数叫compression.type,参数的默认值是producer，表示会尊重producer端的压缩算法，可一旦指定了不同的type，就会发生预料之外的压缩、解压操作，通常Broker端CPU飙升。
- Broker端发生了消息格式转换。所谓的消息格式转换主要是为了兼容老版本的消费者程序。还记得之前说过的 V1、V2 版本吧？在一个生产环境中，Kafka 集群中同时保存多种版本的消息格式非常常见。为了兼容老版本的格式，Broker 端会对新版本消息执行向老版本格式的转换。这个过程中会涉及消息的解压缩和重新压缩。一般情况下这种消息格式转换对性能是有很大影响的，除了这里的压缩之外，它还让 Kafka 丧失了引以为豪的 Zero Copy 特性。

通常来说解压缩发生在消费者程序中，也就是说 Producer 发送压缩消息到 Broker 后，Broker 照单全收并原样保存起来。当 Consumer 程序请求这部分消息时，Broker 依然原样发送出去，当消息到达 Consumer 端后，由 Consumer 自行解压缩还原成之前的消息。

**Producer 端压缩、Broker 端保持、Consumer 端解压缩。**

Kafka对已提交的消息做有限度的持久化保证。什么是已提交的消息？当 Kafka 的若干个 Broker 成功地接收到一条消息并写入到日志文件后，它们会告诉生产者程序这条消息已成功提交。此时，这条消息在 Kafka 看来就正式变为“已提交”消息了。目前 Kafka Producer 是异步发送消息的，也就是说如果你调用的是 producer.send(msg) 这个 API，那么它通常会立即返回，但此时你不能认为消息发送已成功完成。

这种发送方式有个有趣的名字，叫“fire and forget”，翻译一下就是“发射后不管”。这个术语原本属于导弹制导领域，后来被借鉴到计算机领域中，它的意思是，执行完一个操作后不去管它的结果是否成功。调用 producer.send(msg) 就属于典型的“fire and forget”，因此如果出现消息丢失，我们是无法知晓的。Producer 永远要使用带有回调通知的发送 API，也就是说不要使用 producer.send(msg)，而要使用 producer.send(msg, callback)。不要小瞧这里的 callback（回调），它能准确地告诉你消息是否真的提交成功了。一旦出现消息提交失败的情况，你就可以有针对性地进行处理。

Consumer 程序有个“位移”的概念，表示的是这个 Consumer 当前消费到的 Topic 分区的位置。下面这张图来自于官网，它清晰地展示了 Consumer 端的位移数据。

![kafka](../images/kafkaconsumer.png)

比如对于 Consumer A 而言，它当前的位移值就是 9；Consumer B 的位移值是 11。这里的“位移”类似于我们看书时使用的书签，它会标记我们当前阅读了多少页，下次翻书的时候我们能直接跳到书签页继续阅读。正确使用书签有两个步骤：第一步是读书，第二步是更新书签页。如果这两步的顺序颠倒了，就可能出现这样的场景：当前的书签页是第 90 页，我先将书签放到第 100 页上，之后开始读书。当阅读到第 95 页时，我临时有事中止了阅读。那么问题来了，当我下次直接跳到书签页阅读时，我就丢失了第 96～99 页的内容，即这些消息就丢失了。同理，Kafka 中 Consumer 端的消息丢失就是这么一回事。要对抗这种消息丢失，办法很简单：<strong>维持先消费消息（阅读），再更新位移（书签）的顺序</strong>即可。这样就能最大限度地保证消息不丢失。

#### 使用总结：

- 不要使用 producer.send(msg)，而要使用 producer.send(msg, callback)。记住，一定要使用带有回调通知的 send 方法。
- 设置 acks = all。acks 是 Producer 的一个参数，代表了你对“已提交”消息的定义。如果设置成 all，则表明所有副本 Broker 都要接收到消息，该消息才算是“已提交”。这是最高等级的“已提交”定义。
- 设置 retries 为一个较大的值。这里的 retries 同样是 Producer 的参数，对应前面提到的 Producer 自动重试。当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了 retries > 0 的 Producer 能够自动重试消息发送，避免消息丢失。
- 设置 unclean.leader.election.enable = false。这是 Broker 端的参数，它控制的是哪些 Broker 有资格竞选分区的 Leader。如果一个 Broker 落后原先的 Leader 太多，那么它一旦成为新的 Leader，必然会造成消息的丢失。故一般都要将该参数设置成 false，即不允许这种情况的发生。
- 设置 replication.factor >= 3。这也是 Broker 端的参数。其实这里想表述的是，最好将消息多保存几份，毕竟目前防止消息丢失的主要机制就是冗余。
- 设置 min.insync.replicas > 1。这依然是 Broker 端参数，控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。在实际环境中千万不要使用默认值 1。
- 确保 replication.factor > min.insync.replicas。如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。推荐设置成 replication.factor = min.insync.replicas + 1。
- 确保消息消费完成再提交。Consumer 端有个参数 enable.auto.commit，最好把它设置成 false，并采用手动提交位移的方式。就像前面说的，这对于单 Consumer 多线程处理的场景而言是至关重要的。

#### 连接管理：

Kafka所有通信是基于TCP的，包括生产者、消费者还是Broker之间的通信。<strong>在创建 KafkaProducer 实例时，生产者应用会在后台创建并启动一个名为 Sender 的线程，该 Sender 线程开始运行时首先会创建与 Broker 的连接</strong>。 bootstrap.servers 参数。它是 Producer 的核心参数之一，指定了这个 Producer 启动时要连接的 Broker 地址。请注意，这里的“启动时”，代表的是 Producer 启动时会发起与这些 Broker 的连接。因此，如果你为这个参数指定了 1000 个 Broker 连接信息，那么很遗憾，你的 Producer 启动时会首先创建与这 1000 个 Broker 的 TCP 连接。在实际使用过程中，并不建议把集群中所有的 Broker 信息都配置到 bootstrap.servers 中，通常你指定 3～4 台就足以了。因为 Producer 一旦连接到集群中的任一台 Broker，就能拿到整个集群的 Broker 信息，故没必要为 bootstrap.servers 指定所有的 Broker。Producer 向某一台 Broker 发送了 METADATA 请求，尝试获取集群的元数据信息——这就是前面提到的 Producer 能够获取集群所有Broker地址的方法。

#### 幂等性

简单来说，幂等性 Producer 和事务型 Producer 都是 Kafka 社区力图为 Kafka 实现精确一次处理语义所提供的工具，只是它们的作用范围是不同的。幂等性 Producer 只能保证单分区、单会话上的消息幂等性；而事务能够保证跨分区、跨会话间的幂等性。从交付语义上来看，自然是事务型 Producer 能做的更多。Kafka中，Producer默认不是幂等的，但可以创建幂等性Producer。 props.put(“enable.idempotence”, ture)保证幂等，enable.idempotence被设置成true后，Producer自动升级成幂等性Producer。底层具体的原理很简单，就是经典的用空间去换时间的优化思路，即在 Broker 端多保存一些字段。当 Producer 发送了具有相同字段值的消息后，Broker 能够自动知晓这些消息已经重复了，于是可以在后台默默地把它们“丢弃”掉。首先，它只能保证单分区上的幂等性，即一个幂等性 Producer 能够保证某个主题的一个分区上不出现重复消息，它无法实现多个分区的幂等性。其次，它只能实现单会话上的幂等性，不能实现跨会话的幂等性。这里的会话，你可以理解为 Producer 进程的一次运行。当你重启了 Producer 进程之后，这种幂等性保证就丧失了。

Kafka 自 0.11 版本开始也提供了对事务的支持，目前主要是在 read committed 隔离级别上做事情。它能保证多条消息原子性地写入到目标分区，同时也能保证 Consumer 只能看到事务成功提交的消息。事务型 Producer 能够保证将消息原子性地写入到多个分区中。这批消息要么全部写入成功，要么全部失败。另外，事务型 Producer 也不惧进程的重启。Producer 重启回来后，Kafka 依然保证它们发送消息的精确一次处理。

设置事务型 Producer 的方法也很简单，满足两个要求即可：

- 和幂等性 Producer 一样，开启 enable.idempotence = true。
- 设置 Producer 端参数 transctional. id。最好为其设置一个有意义的名字。

如：

Producer.initTransaction();

Try  {

​			producer.beginTransaction();

​			producer.send(record1);

​			producer.send(record2);

​			producer.commitTransaction();

}catch(KafkaException e){

​			producer.abortTransaction();

}

和普通 Producer 代码相比，事务型 Producer 的显著特点是调用了一些事务 API，如 initTransaction、beginTransaction、commitTransaction 和 abortTransaction，它们分别对应事务的初始化、事务开始、事务提交以及事务终止。这段代码能够保证 Record1 和 Record2 被当作一个事务统一提交到 Kafka，要么它们全部提交成功，要么全部写入失败。实际上即使写入失败，Kafka 也会把它们写入到底层的日志中，也就是说 Consumer 还是会看到这些消息。因此在 Consumer 端，读取事务型 Producer 发送的消息也是需要一些变更的。修改起来也很简单，设置 isolation.level 参数的值即可。当前这个参数有两个取值：

- read_uncommitted：这是默认值，表明 Consumer 能够读取到 Kafka 写入的任何消息，不论事务型 Producer 提交事务还是终止事务，其写入的消息都可以读取。很显然，如果你用了事务型 Producer，那么对应的 Consumer 就不要使用这个值。
- read_committed：表明 Consumer 只会读取事务型 Producer 成功提交事务写入的消息。当然了，它也能看到非事务型 Producer 写入的所有消息。

#### 消费者组

Consumer Group是Kafka提供的可扩展且具有容错性的消费者机制。理想情况下，Consumer实例的数量等于该Group订阅主题的分区总数。举个简单的例子，假设一个 Consumer Group 订阅了3个主题，分别是A、B、C，它们的分区数依次是1、2、3，那么通常情况下，为该Group设置6个Consumer实例是比较理想的情形，能最大限度的实现高伸缩性。新版本的Consumer Group将位移保存在Broker端的内部主题中。消费者组可以Rebalance，Rebalace是一种协议，规定了一个Consumer Group下的所有Consumer如何达成一致，来分配订阅Topic的每个分区。比如某个Group下有20个Consumer实例，订阅了一个具有100个分区的Topic，通常情况下每个Consumer分配5个分区。触发Rebalance的条件：

- 组成员数发生变更
- 订阅主题数发生变更
- 订阅主题的分区数发生变更

Rebalance 就是让一个 Consumer Group 下所有的 Consumer 实例就如何消费订阅主题的所有分区达成共识的过程。Rebalance过程中，所有Consumer实例共同参与，在协调者组件的帮助下，完成订阅主题分区的分配。在整个过程中，所有实例不能消费消息，对TPS影响很大。所谓协调者，在 Kafka 中对应的术语是 Coordinator，它专门为 Consumer Group 服务，负责为 Group 执行 Rebalance 以及提供位移管理和组成员管理等。Broker启动时，会创建和开启相应的Coordinator组件。也就是说，所有Broker都有各自的Coordinator组件。目前确定Coordinator所在Broker的算法有两步：

1）确定由位移主题的哪个分区来保存该Group数据：partitionId=Math.abs(groupId.hashCode() % offsetsTopicPartitionCount)。

2）找出该分区 Leader 副本所在的 Broker，该 Broker 即为对应的 Coordinator。

#### 避免不必要的Rebalance

1）未能及时发送心跳，导致Consumer被"踢出"Group引发。推荐值：**session.timeout.ms>=3*heartbeat.interval.ms.**

2）Consumer消费时间过长。**max.poll.interval.ms**设置得大一点。

Consumer的位移数据作为一条条普通的Kafka消息，提交到__consumer_offsets中，用__consumer_offsets的主要作用是保存kafka消费者的位移信息。位移主题的Key中保留了3部分内容：**<Group ID,主题名，分区号>**。当Kafka急群众的第一个Consumer程序启动时，Kafka会自动创建位移主题。**如果位移主题是Kafka自动创建的，那么该主题的分区数是50，副本数是3。**

#### 自动提交位移和手动提交位移

Consumer端有个参数叫enable.auto.commit,如果为true，Consumer后台默默的定期提交位移，提交间隔由一个专属的参数auto.commit.interval.ms来控制。自动提交位移时存在的问题，只要Consumer一直启动着，就会无限期的向位移主题写入消息。我们来举个极端一点的例子。假设 Consumer 当前消费到了某个主题的最新一条消息，位移是 100，之后该主题没有任何新消息产生，故 Consumer 无消息可消费了，所以位移永远保持在 100。由于是自动提交位移，位移主题中会不停地写入位移 =100 的消息。显然 Kafka 只需要保留这类消息中的最新一条就可以了，之前的消息都是可以删除的。这就要求 Kafka 必须要有针对位移主题消息特点的消息删除策略，否则这种消息会越来越多，最终撑爆整个磁盘。

Kafka使用**Compact策略**来删除位移主题中的过期消息，避免该主题无限期膨胀。对于同一个Key的两条消息M1和M2，如果M1的发送时间早于M2，那么M1就是过期消息。Compact的过程就是扫描日志的所有消息，剔除过期的消息，把剩下的整理在一起。如图：

![compact](../images/compact.jpeg)

图中位移为0、2和3的消息的Key都是K1。Compact之后，分区只需要保存位移为3的消息就行。**Kafka提供了专门的后台线程 Log Cleaner，定期巡检待Compact的主题，看看是否存在可以删除的数据**。

#### Consumer消费位移

记录了要消费的下一条消息的位移，是下一条，而不是目前最新消费消息的位移。**Consumer向Kafka汇报自己位移数据的过程叫提交位移**。因为Consumer能同时消费多个分区的数据，所以位移的提交实际上是在分区粒度上进行的，即**Consumer需要为分配给它的每个分区提交各自的位移数据**。

从用户角度，有手动提交和自动提交，从Consumer端的角度，分为同步提交和异步提交。

自动提交：一旦设置了 enable.auto.commit 为 true，Kafka 会保证在开始调用 poll 方法时，提交上次 poll 返回的所有消息。从顺序上来说，poll 方法的逻辑是先提交上一批消息的位移，再处理下一批消息，因此它能保证不出现消费丢失的情况。但自动提交位移的一个问题在于，<strong>它可能会出现重复消费</strong>。在默认情况下，Consumer 每 5 秒自动提交一次位移。现在，我们假设提交位移之后的 3 秒发生了 Rebalance 操作。在 Rebalance 之后，所有 Consumer 从上一次提交的位移处继续消费，但该位移已经是 3 秒前的位移数据了，故在 Rebalance 发生前 3 秒消费的所有数据都要重新再消费一次。虽然你能够通过减少 auto.commit.interval.ms 的值来提高提交频率，但这么做只能缩小重复消费的时间窗口，不可能完全消除它。这是自动提交机制的一个缺陷。

手动提交：commitSync()，被调用时，Consumer处于阻塞状态，直至远端的Broker返回结果，这个状态才结束，这样会导致系统的瓶颈。如果拉长提交间隔，下次Consumer重启后，会有更多的消息被重新消费。KafkaConsumer#commitAsync()  异步提交。异步提交的问题是出现问题时不会自动重试。因为是异步操作，如果提交失败后自动重试，重试时提交的位移值可能已经"过期"，因此重试没有意义。显然，如果是手动提交，我们需要将 commitSync 和 commitAsync 组合使用才能到达最理想的效果，原因有两个：

- 我们可以利用 commitSync 的自动重试来规避那些瞬时错误，比如网络的瞬时抖动，Broker 端 GC 等。因为这些问题都是短暂的，自动重试通常都会成功，因此，我们不想自己重试，而是希望 Kafka Consumer 帮我们做这件事。
- 我们不希望程序总处于阻塞状态，影响 TPS。

~~~java
   try {
           while(true) {
                        ConsumerRecords<String, String> records = 
                                    consumer.poll(Duration.ofSeconds(1));
                        process(records); // 处理消息
                        commitAysnc(); // 使用异步提交规避阻塞
            }
} catch(Exception e) {
            handle(e); // 处理异常
} finally {
            try {
                        consumer.commitSync(); // 最后一次提交使用同步阻塞式提交
	} finally {
	     consumer.close();
}
}


//分批提交位移
private Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
int count = 0;
……
while (true) {
            ConsumerRecords<String, String> records = 
	consumer.poll(Duration.ofSeconds(1));
            for (ConsumerRecord<String, String> record: records) {
                        process(record);  // 处理消息
                        offsets.put(new TopicPartition(record.topic(), record.partition()),
                                   new OffsetAndMetadata(record.offset() + 1)；
                       if（count % 100 == 0）
                                    consumer.commitAsync(offsets, null); // 回调处理逻辑是 null
                        count++;
	}
}

~~~

位移提交：

![suboffset](../images/suboffset.jpeg)

#### CommitFailedException

所谓 CommitFailedException，顾名思义就是 Consumer 客户端在提交位移时出现了错误或异常，而且还是那种不可恢复的严重异常。从源代码方面来说，CommitFailedException 异常通常发生在手动提交位移时，即用户显式调用 KafkaConsumer.commitSync() 方法时。从使用场景来说，有两种典型的场景可能遭遇该异常。

第一种：当消息处理的总时间超过预设的 max.poll.interval.ms 参数值时，Kafka Consumer 端会抛出 CommitFailedException 异常。这是该异常最“正宗”的登场方式。你只需要写一个 Consumer 程序，使用 KafkaConsumer.subscribe 方法随意订阅一个主题，之后设置 Consumer 端参数 max.poll.interval.ms=5 秒，最后在循环调用 KafkaConsumer.poll 方法之间，插入 Thread.sleep(6000) 和手动提交位移，就可以成功复现这个异常了。防止出现这种异常，可以尝试以下操作：

- 缩短单条消息处理的时间，比如，之前下游系统消费一条消息的时间是 100 毫秒，优化之后成功地下降到 50 毫秒，那么此时 Consumer 端的 TPS 就提升了一倍。
- 增加 Consumer 端允许下游系统消费一批消息的最大时长。
- 减少下游系统一次性消费的消息总数。。这取决于 Consumer 端参数 max.poll.records 的值。当前该参数的默认值是 500 条，表明调用一次 KafkaConsumer.poll 方法，最多返回 500 条消息。可以说，该参数规定了单次 poll 方法能够返回的消息总数的上限。如果前两种方法对你都不适用的话，降低此参数值是避免 CommitFailedException 异常最简单的手段。
- 下游系统使用多线程来加速消费

首先，你需要弄清楚你的下游系统消费每条消息的平均延时是多少。比如你的消费逻辑是从 Kafka 获取到消息后写入到下游的 MongoDB 中，假设访问 MongoDB 的平均延时不超过 2 秒，那么你可以认为消息处理需要花费 2 秒的时间。如果按照 max.poll.records 等于 500 来计算，一批消息的总消费时长大约是 1000 秒，因此你的 Consumer 端的 max.poll.interval.ms 参数值就不能低于 1000 秒。如果你使用默认配置，那默认值 5 分钟显然是不够的，你将有很大概率遭遇 CommitFailedException 异常。将 max.poll.interval.ms 增加到 1000 秒以上的做法就属于上面的第 2 种方法。除了调整 max.poll.interval.ms 之外，你还可以选择调整 max.poll.records 值，减少每次 poll 方法返回的消息数。还拿刚才的例子来说，你可以设置 max.poll.records 值为 150，甚至更少，这样每批消息的总消费时长不会超过 300 秒（150*2=300），即 max.poll.interval.ms 的默认值 5 分钟。这种减少 max.poll.records 值的做法就属于上面提到的方法 3。

第二种不太常见的场景

Kafka Java Consumer 端还提供了一个名为 Standalone Consumer 的独立消费者。它没有消费者组的概念，每个消费者实例都是独立工作的，彼此之间毫无联系。不过，你需要注意的是，独立消费者的位移提交机制和消费者组是一样的，因此独立消费者的位移提交也必须遵守之前说的那些规定，比如独立消费者也要指定 group.id 参数才能提交位移。你可能会觉得奇怪，既然是独立消费者，为什么还要指定 group.id 呢？没办法，谁让社区就是这么设计的呢？总之，消费者组和独立消费者在使用之前都要指定 group.id。如果你的应用中同时出现了设置相同 group.id 值的消费者组程序和独立消费者程序，那么当独立消费者程序手动提交位移时，Kafka 就会立即抛出 CommitFailedException 异常，因为 Kafka 无法识别这个具有相同 group.id 的消费者实例，于是就向它返回一个错误，表明它不是消费者组内合法的成员。

#### 多线程开发消费者

1、消费者程序启动多个线程，每个线程维护专属的KafkaConsumer实例，负责完整的消息获取、处理流程，如下图所示：

![multiplec1](../images/multiplec1.png)

2、消费者程序使用单或多线程获取消息，同时创建多个消费线程执行消息处理逻辑。获取消息的线程可以是一个，也可是多个，每个线程维护专属的KafkaConsumer实例，处理消息则交由特定的线程池来做。如下图：

![multiplec2](../images/multiplec2.png)

优缺点对比：

![solutioncompare](../images/solutioncompare.jpeg)

#### Kafka消费者TCP连接

Kafka网络传输是基于TCP协议的，而不是UDP协议。和生产者不同的是，构建KafkaConsumer实例时不会创建任何TCP连接。当执行完new KafkaConsumer(properties)后并没有socket连接被创建出来。这一点与Java生产者有区别，生产者入口类KafkaProducer在构建实例的时候，会在后台默默启动一个Sender线程，负责Socket连接的创建。**TCP连接是在调用KafkaConsumer.poll方法时创建的**，poll方法内部有3个时机可以创建TCP连接。

1、发起FindCoordinator请求时

消费者端的协调者(Coordinator)会驻留在Broker端的内存中，负责消费者组的成员管理和各个消费者的位移提交管理。当消费者程序首次启动调用Poll方法时，需要向kafka集群发送一个名为FindCoordinator的请求，希望kafka集群告诉它哪个broker是管理它的协调者。理论上消费者可以向任意broker发送请求，但实际上，消费者程序会向集群中当前负载最小的那台broker发请求，如何评估负载呢？看待消费者连接的所有broker中，谁的待发送请求最少。

2、连接协调者时

Broker 处理完上一步发送的 FindCoordinator 请求之后，会返还对应的响应结果（Response），显式地告诉消费者哪个 Broker 是真正的协调者，因此在这一步，消费者知晓了真正的协调者后，会创建连向该 Broker 的 Socket 连接。只有成功连入协调者，协调者才能开启正常的组协调操作，比如加入组、等待组分配方案、心跳请求处理、位移获取、位移提交等。

3、消费数据时

消费者会为每个要消费的分区创建与该分区领导者副本所在 Broker 连接的 TCP。举个例子，假设消费者要消费 5 个分区的数据，这 5 个分区各自的领导者副本分布在 4 台 Broker 上，那么该消费者在消费时会创建与这 4 台 Broker 的 Socket 连接。

消费者程序会创建3类TCP连接：**1、确定协调者和获取集群元数据 2、连接协调者、令其执行组成员管理操作 3、执行实际的消息获取**，当第三类连接成功创建后，消费者程序就会放弃第一类TCP连接。

#### TCP连接的关闭

主动关闭和kafka自动关闭，主动是指**手动调用KafkaConsumer.close()或者执行kill**，kafka自动关闭是由**消费者端参数connection.max.idle.ms**控制的，默认9分钟。

### 消费者组监控进度

1、Kafka自带命令**kafka-consumer-groups 脚本是 Kafka 为我们提供的最直接的监控消费者消费进度的工具**

使用方法：

**$ bin/kafka-consumer-groups.sh --bootstrap-server <Kafka broker 连接信息 > --describe --group <group 名称 >**

![kafkamonitor](../images/kafkamonitor.png)

当出现Consumer group '' has no active members时，是因为我们运行 kafka-consumer-groups 脚本时没有启动消费者程序。它显式地告诉我们，当前消费者组没有任何 active 成员，即没有启动任何消费者实例。虽然这些列没有值，但 LAG 列依然是有效的，它依然能够正确地计算出此消费者组的 Lag 值。

2、Kafka Java Consumer API 

简单来说，社区提供的 Java Consumer API 分别提供了查询当前分区最新消息位移和消费者组最新消费消息位移两组方法，我们使用它们就能计算出对应的 Lag。

~~~java
public static Map<TopicPartition, Long> lagOf(String groupID, String bootstrapServers) throws TimeoutException {
        Properties props = new Properties();
        props.put(CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        try (AdminClient client = AdminClient.create(props)) {
            ListConsumerGroupOffsetsResult result = client.listConsumerGroupOffsets(groupID);
            try {
                Map<TopicPartition, OffsetAndMetadata> consumedOffsets = result.partitionsToOffsetAndMetadata().get(10, TimeUnit.SECONDS);
                props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false); // 禁止自动提交位移
                props.put(ConsumerConfig.GROUP_ID_CONFIG, groupID);
                props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
                props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
                try (final KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props)) {
                    Map<TopicPartition, Long> endOffsets = consumer.endOffsets(consumedOffsets.keySet());
                    return endOffsets.entrySet().stream().collect(Collectors.toMap(entry -> entry.getKey(),
                            entry -> entry.getValue() - consumedOffsets.get(entry.getKey()).offset()));
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                // 处理中断异常
                // ...
                return Collections.emptyMap();
            } catch (ExecutionException e) {
                // 处理 ExecutionException
                // ...
                return Collections.emptyMap();
            } catch (TimeoutException e) {
                throw new TimeoutException("Timed out when getting lag for consumer group " + groupID);
            }
        }
    }

~~~

3、Kafka JMX监控指标

当前，Kafka 消费者提供了一个名为 kafka.consumer:type=consumer-fetch-manager-metrics,client-id=“{client-id}”的 JMX 指标，里面有很多属性。和我们今天所讲内容相关的有两组属性：records-lag-max 和 records-lead-min，它们分别表示此消费者在测试窗口时间内曾经达到的最大的 Lag 值和最小的 Lead 值。这里的 Lead 值是指消费者最新消费消息的位移与分区当前第一条消息位移的差值，Lag 越大的话，Lead 就越小，反之也是同理。**一旦你监测到 Lead 越来越小，甚至是快接近于 0 了，你就一定要小心了，这可能预示着消费者端要丢消息了**

#### kafka副本机制

副本的好处：1、提供数据冗余 2、提供高伸缩性 3、改善数据局部性。但是Apache Kafka智能享受到副本带来的第一个好处，也就是提供数据冗余实现高可用性和高持久性。

kafka有主题的概念，每个主题进一步划分成若干个分区，副本的概念实际是在分区层级下定义的，每个分区配置有若干个副本。**所谓副本（Replica），本质就是一个只能追加写消息的提交日志**。。根据 Kafka 副本机制的定义，同一个分区下的所有副本保存有相同的消息序列，这些副本分散保存在不同的 Broker 上，从而能够对抗部分 Broker 宕机带来的数据不可用。在实际生产环境中，每台 Broker 都可能保存有各个主题下不同分区的不同副本，因此，单个 Broker 上存有成百上千个副本的现象是非常正常的。如图：

![kafkareplica](../images/kafkareplica.png)

如何保证数据的一致性呢？答案是**基于领导者(Leader-based)的副本机制**，如图：

![leaderbased](../images/leaderbased.png)

第一，在 Kafka 中，副本分成两类：领导者副本（Leader Replica）和追随者副本（Follower Replica）。每个分区在创建时都要选举一个副本，称为领导者副本，其余的副本自动称为追随者副本。第二，Kafka 的副本机制比其他分布式系统要更严格一些。在 Kafka 中，追随者副本是不对外提供服务的。这就是说，任何一个追随者副本都不能响应消费者和生产者的读写请求。所有的请求都必须由领导者副本来处理，或者说，所有的读写请求都必须发往领导者副本所在的 Broker，由该 Broker 负责处理。追随者副本不处理客户端请求，它唯一的任务就是从领导者副本<strong>异步拉取</strong>消息，并写入到自己的提交日志中，从而实现与领导者副本的同步。第三，当领导者副本挂掉了，或者说领导者副本所在的 Broker 宕机时，Kafka 依托于 ZooKeeper 提供的监控功能能够实时感知到，并立即开启新一轮的领导者选举，从追随者副本中选一个作为新的领导者。老 Leader 副本重启回来后，只能作为追随者副本加入到集群中。**追随者副本是不对外提供服务的**。对于客户端用户而言，Kafka 的追随者副本没有任何作用，它既不能像 MySQL 那样帮助领导者副本“抗读”，也不能实现将某些副本放到离客户端近的地方来改善数据局部性。

为什么这么设计呢？

1、<strong>方便实现“Read-your-writes”</strong>，所谓 Read-your-writes，顾名思义就是，当你使用生产者 API 向 Kafka 成功写入消息后，马上使用消费者 API 去读取刚才生产的消息。

2、**方便实现单调读（Monotonic Reads）**，什么是单调读呢？就是对于一个消费者用户而言，在多次消费消息时，它不会看到某条消息一会儿存在一会儿不存在。

**In-sync Replicas（ISR）**

我们刚刚反复说过，追随者副本不提供服务，只是定期地异步拉取领导者副本中的数据而已。既然是异步的，就存在着不可能与 Leader 实时同步的风险。在探讨如何正确应对这种风险之前，我们必须要精确地知道同步的含义是什么。或者说，Kafka 要明确地告诉我们，追随者副本到底在什么条件下才算与 Leader 同步。基于这个想法，Kafka 引入了 In-sync Replicas，也就是所谓的 ISR 副本集合。ISR 中的副本都是与 Leader 同步的副本，相反，不在 ISR 中的追随者副本就被认为是与 Leader 不同步的。那么，到底什么副本能够进入到 ISR 中呢？我们首先要明确的是，Leader 副本天然就在 ISR 中。也就是说，<strong>ISR 不只是追随者副本集合，它必然包括 Leader 副本。甚至在某些情况下，ISR 只有 Leader 这一个副本</strong>。我们看下图：

![kafkaleader](../images/kafkaleader.png)

图中有 3 个副本：1 个领导者副本和 2 个追随者副本。Leader 副本当前写入了 10 条消息，Follower1 副本同步了其中的 6 条消息，而 Follower2 副本只同步了其中的 3 条消息。现在，请你思考一下，对于这 2 个追随者副本，你觉得哪个追随者副本与 Leader 不同步？事实上，这张图中的 2 个 Follower 副本都有可能与 Leader 不同步，但也都有可能与 Leader 同步。也就是说，Kafka 判断 Follower 是否与 Leader 同步的标准，不是看相差的消息数，而是另有“玄机”。<strong>这个标准就是 Broker 端参数 replica.lag.time.max.ms 参数值</strong>。。这个参数的含义是 Follower 副本能够落后 Leader 副本的最长时间间隔，当前默认值是 10 秒。这就是说，只要一个 Follower 副本落后 Leader 副本的时间不连续超过 10 秒，那么 Kafka 就认为该 Follower 副本与 Leader 是同步的，即使此时 Follower 副本中保存的消息明显少于 Leader 副本中的消息。我们在前面说过，Follower 副本唯一的工作就是不断地从 Leader 副本拉取消息，然后写入到自己的提交日志中。如果这个同步过程的速度持续慢于 Leader 副本的消息写入速度，那么在 replica.lag.time.max.ms 时间后，此 Follower 副本就会被认为是与 Leader 副本不同步的，因此不能再放入 ISR 中。此时，Kafka 会自动收缩 ISR 集合，将该副本“踢出”ISR。值得注意的是，倘若该副本后面慢慢地追上了 Leader 的进度，那么它是能够重新被加回 ISR 的。这也表明，ISR 是一个动态调整的集合，而非静态不变的。

**Unclean 领导者选举（Unclean Leader Election）**

既然 ISR 是可以动态调整的，那么自然就可以出现这样的情形：ISR 为空。因为 Leader 副本天然就在 ISR 中，如果 ISR 为空了，就说明 Leader 副本也“挂掉”了，Kafka 需要重新选举一个新的 Leader。可是 ISR 是空，此时该怎么选举新 Leader 呢？<strong>Kafka 把所有不在 ISR 中的存活副本都称为非同步副本</strong>。通常来说，非同步副本落后 Leader 太多，因此，如果选择这些副本作为新 Leader，就可能出现数据的丢失。毕竟，这些副本中保存的消息远远落后于老 Leader 中的消息。在 Kafka 中，选举这种副本的过程称为 Unclean 领导者选举。<strong>Broker 端参数 unclean.leader.election.enable 控制是否允许 Unclean 领导者选举</strong>。开启 Unclean 领导者选举可能会造成数据丢失，但好处是，它使得分区 Leader 副本一直存在，不至于停止对外提供服务，因此提升了高可用性。反之，禁止 Unclean 领导者选举的好处在于维护了数据的一致性，避免了消息丢失，但牺牲了高可用性。

#### Kafka是如何处理请求的？

**Reactor模式**，Reactor 模式是事件驱动架构的一种实现方式，特别适合应用于处理多个客户端并发向服务器端发送请求的场景。reactor模式架构如图：

![reactor](../images/reactor.png)

从这张图中，我们可以发现，多个客户端会发送请求给到 Reactor。Reactor 有个请求分发线程 Dispatcher，也就是图中的 Acceptor，它会将不同的请求下发到多个工作线程中处理。在这个架构中，Acceptor 线程只是用于请求分发，不涉及具体的逻辑处理，非常得轻量级，因此有很高的吞吐量表现。而这些工作线程可以根据实际业务处理需要任意增减，从而动态调节系统负载能力。kafka类似的图：

![kafkaart](../images/kafkaart.png)

显然，这两张图长得差不多。Kafka 的 Broker 端有个 SocketServer 组件，类似于 Reactor 模式中的 Dispatcher，它也有对应的 Acceptor 线程和一个工作线程池，只不过在 Kafka 中，这个工作线程池有个专属的名字，叫网络线程池。Kafka 提供了 Broker 端参数 num.network.threads，用于调整该网络线程池的线程数。其<strong>默认值是 3，表示每台 Broker 启动时会创建 3 个网络线程，专门处理客户端发送的请求</strong>。Acceptor线程采用轮训的方式将入站请求公平地发到所有网络线程中。网络线程接收到请求后，处理流程如图：

![brokerhandle](../images/brokerhandle.png)

网络线程拿到请求后，将请求放入共享请求队列。Broker端还有个IO线程池，负责从该队列取出请求，执行真正的处理逻辑。如果是PRODUCE生产请求，则将消息写入底层的磁盘日志，如果是FETCH请求，从磁盘或者页缓存中读取消息。IO 线程池处中的线程才是执行请求逻辑的线程。Broker 端参数<strong>num.io.threads</strong>控制了这个线程池中的线程数。<strong>目前该参数默认值是 8，表示每台 Broker 启动后自动创建 8 个 IO 线程处理请求</strong>。你可以根据实际硬件条件设置此线程池的个数。IO线程处理完请求后，会将生成的响应发送到网络线程池的响应队列，然后由对应的万那个罗线程将Response返还给客户端。<strong>请求队列是所有网络线程共享的，而响应队列则是每个网络线程专属的</strong>。这么设计的原因就在于，Dispatcher 只是用于请求分发而不负责响应回传，因此只能让每个网络线程自己发送 Response 给客户端，所以这些 Response 也就没必要放在一个公共的地方。

上图中有一个叫 Purgatory 的组件，这是 Kafka 中著名的“炼狱”组件。它是用来<strong>缓存延时请求</strong>（Delayed Request）的。<strong>所谓延时请求，就是那些一时未满足条件不能立刻处理的请求</strong>。比如设置了 acks=all 的 PRODUCE 请求，一旦设置了 acks=all，那么该请求就必须等待 ISR 中所有副本都接收了消息后才能返回，此时处理该请求的 IO 线程就必须等待其他 Broker 的写入结果。当请求不能立刻处理时，它就会暂存在 Purgatory 中。稍后一旦满足了完成条件，IO 线程会继续处理该请求，并将 Response 放入对应网络线程的响应队列中。**Kafka的数据类请求和控制类请求是分离的。**

#### 消费者组重平衡流程

触发的条件：1、组成员数量发生变化 2、订阅主题数发生变化 3、订阅主题的分区数发生变化。**重平衡过程是如何通知到其他消费者的呢？**—消费者端的心跳线程

Kafka Java消费者定期的发送心跳请求到Broker端的协调者，0.10.1.0版本之前，发送心跳是在消费者主线程完成的，就是写代码调用KafkaConsumer.poll方法的那个线程，消息处理的逻辑也在这个线程中完成，这样会存在很大的问题，0.10.1.0之后，引入了单独的心跳线程专门执行心跳请求，来规避消息处理时间过长导致心跳无法及时发送的问题。当协调者决定开启新一轮重平衡后，会将**REBANLANCE_IN_PROGRESS**封装进心跳请求的响应中，发还给消费者实例。**重平衡的通知机制正是通过心跳线程来完成的。**消费者端参数 heartbeat.interval.ms 的真实用途，从字面上看，它就是设置了心跳的间隔时间，但这个参数的真正作用是控制重平衡通知的频率。如果你想要消费者实例更迅速地得到通知，那么就可以给这个参数设置一个非常小的值，这样消费者就能更快地感知到重平衡已经开启了。

#### 消费者组状态机

为了完成真个重平衡流程，Kafka设计了一组包含了五个状态的状态机。分别是：Empty，Dead，PreparingRebalance，CompletingRebalance和Stable。各自状态的含义如图：

![kafkastate](../images/kafkastate.jpeg)

状态之间的流转：

![statetransfer](../images/statetransfer.png)

一个消费者组最初是Empty状态，当重平衡过程开启后，会被置于PreparingRebalance状态等待成员加入，之后变成CompletingRebalance状态等待分配方案，最后扭转成Stable状态完成重平衡。当有新成员加入或退出时，状态从Stable变成PreparingRebalance，此时，所有成员必须重新申请加入组。当所有成员退出后，状态变为Empty。Kafka定期自动删除过期位移的条件就是，组处于Empty状态。如果消费者组停掉了很长时间(超过7天)，Kafka很可能把该组的位移数据删除了。日志中会输出：Removed ✘✘✘ expired offsets in ✘✘✘ milliseconds.这是Kafka在尝试定期删除过期位移。

在消费者端，重平衡分两个步骤：加入组和等待领导者消费者(Leader Consumer)分配方案，对应的请求是**JoinGroup请求和SyncGroup请求**

组内成员加入组时，会向协调者发送JoinGroup请求。请求中，每个成员都将自己订阅的主题上报，这样协调者就能获取到所有成员的订阅消息。待收集了全部成员的JoinGroup请求后，协调者会选一个担任这个消费者组的领导者，通常是第一个发送JoinGroup请求的成员自动成为领导者。选出领导者后，协调者会把消费者订阅消息封装进JoinGroup请求的响应体中，然后发给领导者，由领导者统一做出分配方案后，进入下一步：发送SyncGroup请求。

在这一步中，领导者向协调者发送SyncGroup请求，将刚做出的分配方案发给协调者。同时，其他成员也会向协调者发送SyncGroup请求，只是请求体中并没有实际内容。这一步的主要目的是让协调者接收分配方案，然后统一以SyncGroup响应的方式分发给所有成员，这样组内成员就知道自己该消费哪些分区了。

JoinGroup请求过程：JoinGroup 请求的主要作用是将组成员订阅信息发送给领导者消费者，待领导者制定好分配方案后，重平衡流程进入到 SyncGroup 请求阶段。

![joingroup](../images/joingroup.png)

SyncGroup请求处理流程：SyncGroup 请求的主要目的，就是让协调者把领导者制定的分配方案下发给各个组内成员。当所有成员都成功接收到分配方案后，消费者组进入到 Stable 状态，即开始正常的消费工作。

![syncgroup](../images/syncgroup.png)

#### Broker端重平衡

1、新成员入组：当组处于Stable状态后，有新成员加入。当协调者收到新的JoinGroup请求后，会通过心跳请求响应的方式通知组内现有的所有成员，强制开启新一轮的重平衡。时序图如下：

![newjoin](../images/newjoin.png)

2、组成员主动离组：消费者实例所在线程或进程调用close()方法主动通知协调者它要退出。这个场景会有第三类请求：**LeaveGroup请求**。协调者收到LeaveGroup后，依然以心跳响应的方式通知其他成员。时序图如下：

![memberleave](../images/memberleave.png)

3、组成员崩溃离组：消费者实例出现严重故障，突然宕机导致的离组。因为崩溃离组是被动的，所以协调者通常需要等待一段时间才能感知到，这段时间由消费者端参数session.timeout.ms控制。时序图如下：

![downleave](../images/downleave.png)

4、重平衡时协调者对组内成员提交位移的处理

正常情况，每个组内成员都会定期汇报位移给协调者。当重平衡开启时，协调者会给予成员一段缓冲时间，要求每个成员必须在这段时间内快速地上报自己的位移信息，然后再开启正常的JoinGroup与SyncGroup请求，时序如图。

![commitoffset](../images/commitoffset.png)

#### Zookeeper

**Apache ZooKeeper 是一个提供高可靠性的分布式协调服务框架**。。它使用的数据模型类似于文件系统的树形结构，根目录也是以“/”开始。该结构上的每个节点被称为 znode，用来保存一些元数据协调信息。如果以 znode 持久性来划分，**znode 可分为持久性 znode 和临时 znode**，。持久性 znode 不会因为 ZooKeeper 集群重启而消失，而临时 znode 则与创建该 znode 的 ZooKeeper 会话绑定，一旦会话结束，该节点会被自动删除。ZooKeeper 赋予客户端监控 znode 变更的能力，即所谓的 Watch 通知功能。一旦 znode 节点被创建、删除，子节点数量发生变化，抑或是 znode 所存的数据本身变更，ZooKeeper 会通过节点变更监听器 (ChangeHandler) 的方式显式通知客户端。

#### Controller组件

控制器组件（Controller），是 Apache Kafka 的核心组件。它的主要作用是在 Apache ZooKeeper 的帮助下管理和协调整个 Kafka 集群。集群中任意一台 Broker 都能充当控制器的角色，但是，在运行过程中，只能有一个 Broker 成为控制器，行使其管理和协调的职责。每个正常运转的 Kafka 集群，在任意时刻都有且只有一个控制器。官网上有个名为 activeController 的 JMX 指标，可以帮助我们实时监控控制器的存活状态。这个 JMX 指标非常关键，你在实际运维操作过程中，一定要实时查看这个指标的值。下面，我们就来详细说说控制器的原理和内部运行机制。

实际上，Broker 在启动时，会尝试去 ZooKeeper 中创建 /controller 节点。Kafka 当前选举控制器的规则是：第一个成功创建 /controller 节点的 Broker 会被指定为控制器。

控制器的职责大致分为5种：

1、主题管理（创建、删除、增加分区）

这里的主题管理，就是指控制器帮助我们完成对 Kafka 主题的创建、删除以及分区增加的操作。换句话说，当我们执行kafka-topics 脚本时，大部分的后台工作都是控制器来完成的。

2、分区重分配

分区重分配是指kafka-reassign-partitions脚本提供的对已有主题分区进行细粒度的分配功能，这部分功能也是控制器实现的。

3、Preferred领导者选举

Preferred领导者选举主要是Kafka为了避免部分Broker负载过重而提供的一种换Leader的方案。

4、集群成员管理（新增Broker、Broker主动关闭、Broker宕机）

包括自动检测新增Broker、Broker主动关闭及被动宕机，这种自动检测依赖于Zookeeper的Watch功能和Zookeeper临时节点组合实现的。控制器组件会利用Watch 机制检查 ZooKeeper 的 /brokers/ids 节点下的子节点数量变更。目前，当有新 Broker 启动后，它会在 /brokers 下创建专属的 znode 节点。一旦创建完毕，ZooKeeper 会通过 Watch 机制将消息通知推送给控制器，这样，控制器就能自动地感知到这个变化，进而开启后续的新增 Broker 作业。

侦测 Broker 存活性则是依赖于刚刚提到的另一个机制：临时节点。每个 Broker 启动后，会在 /brokers/ids 下创建一个临时 znode。当 Broker 宕机或主动关闭后，该 Broker 与 ZooKeeper 的会话结束，这个 znode 会被自动删除。同理，ZooKeeper 的 Watch 机制将这一变更推送给控制器，这样控制器就能知道有 Broker 关闭或宕机了，从而进行“善后”。

5、数据服务

控制器的最后一类工作，就是向其他Broker提供数据服务。控制器上保存了最全的集群元数据信息，其他所有Broker会定期接收控制器发来的元数据更新请求，从而更新其内存中的缓存数据。

控制器中保存的数据如图，其中比较重要的数据有：（1）所有主题信息。包括具体的分区信息，比如领导者副本是谁，ISR 集合中有哪些副本等。（2）所有 Broker 信息。包括当前都有哪些运行中的 Broker，哪些正在关闭中的 Broker 等。（3）所有涉及运维任务的分区。包括当前正在进行 Preferred 领导者选举以及分区重分配的分区列表。

![kafkacontrollerdata](../images/kafkacontrollerdata.png)

当然，这些数据在Zookeeper中也保存了一份。当控制器初始化时，会从Zookeeper上读取对应的元数据并填充到自己的缓存中。有了这些数据，控制器就可以通过向其他Broker发送请求，将数据同步过去。

#### 控制器故障转移

Kafka提供服务过程中，只有一台Broker充当控制器角色。当单点失效之后，就会发生Failover。故障转移指的是，当运行中的控制器突然宕机或意外终止时，Kafka 能够快速地感知到，并立即启用备用控制器来代替之前失败的控制器。这个过程就被称为 Failover，而且该过程是自动完成的，无需手动干预。故障转移过程如图：最开始时，Broker 0 是控制器。当 Broker 0 宕机后，ZooKeeper 通过 Watch 机制感知到并删除了 /controller 临时节点。之后，所有存活的 Broker 开始竞选新的控制器身份。Broker 3 最终赢得了选举，成功地在 ZooKeeper 上重建了 /controller 节点。之后，Broker 3 会从 ZooKeeper 中读取集群元数据信息，并初始化到自己的缓存中。至此，控制器的 Failover 完成，可以行使正常的工作职责了。

![kafkafailover](../images/kafkafailover.png)

#### 控制器内部设计原理

控制器之前是多线程设计，自0.11版本开始改为单线程加事件队列的方案，规避了多线程之间的线程同步开销。同时，将之前同步操作Zookeeper全部改为异步操作。最新的改进是Broker对接收的请求会分优先级，比如删除了一个主题，那么控制器就会给该主题所有副本所在的 Broker 发送一个名为**StopReplica**的请求。如果此时 Broker 上存有大量积压的 Produce 请求，那么这个 StopReplica 请求只能排队等。如果这些 Produce 请求就是要向该主题发送消息的话，这就显得很讽刺了：主题都要被删除了，处理这些 Produce 请求还有意义吗？此时最合理的处理顺序应该是，**赋予 StopReplica 请求更高的优先级，使它能够得到抢占式的处理**。自2.2开始，支持这种不同优先级请求的处理。

#### 关于高水位和Leader Epoch

水位的经典定义：在时刻 T，任意创建时间（Event Time）为 T’，且 T’≤T 的所有事件都已经到达或被观测到，那么 T 就被定义为水位。也有这样描述：水位是一个单调增加且表征最早未完成工作（oldest work not yet completed）的时间戳。如图：图中标注“Completed”的蓝色部分代表已完成的工作，标注“In-Flight”的红色部分代表正在进行中的工作，两者的边界就是水位线。

![watermark](../images/watermark.png)

Kafka中，水位是和位置信息绑定的，具体来说，它是用消息位移来表征的。

#### 高水位的作用：

1、定义消息可见性，及用来标识分区下的哪些消息是可以被消费者消费的

2、帮助Kafka完成副本同步

例如：

![kafkahw](../images/kafkahw.png)

假如这是某个分区Leader副本的高水位图。在分区高水位以下的消息被认为是已提交消息，反之就是未提交消息。消费者只能消费已提交消息，即图中位移小于 8 的所有消息。注意，这里我们不讨论 Kafka 事务，因为事务机制会影响消费者所能看到的消息的范围，它不只是简单依赖高水位来判断。它依靠一个名为 LSO（Log Stable Offset）的位移值来判断事务型消费者的可见性。另外，**位移值等于高水位的消息也属于未提交消息。也就是说，高水位上的消息是不能被消费者消费的**。图中还有一个日志末端位移的概念，即 Log End Offset，简写是 LEO。它表示副本写入下一条消息的位移值。注意，数字 15 所在的方框是虚线，这就说明，这个副本当前只有 15 条消息，位移值是从 0 到 14，下一条新消息的位移是 15。显然，介于高水位和 LEO 之间的消息就属于未提交消息。这也从侧面告诉了我们一个重要的事实，那就是：**同一个副本对象，其高水位值不会大于 LEO 值**。

**高水位和 LEO 是副本对象的两个重要属性**，。Kafka 所有副本都有对应的高水位和 LEO 值，而不仅仅是 Leader 副本。只不过 Leader 副本比较特殊，Kafka 使用 Leader 副本的高水位来定义所在分区的高水位。换句话说，**分区的高水位就是其 Leader 副本的高水位**。







#### Kafka命令行集合：

**1、查看消费者组消费情况**：     bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group test_order_to_es_groupId0

**2、创建topic**：bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test，推荐使用--bootstrap-server而非--Zookeeper的原因有两个：

- 使用 --zookeeper 会绕过 Kafka 的安全体系。这就是说，即使你为 Kafka 集群设置了安全认证，限制了主题的创建，如果你使用 --zookeeper 的命令，依然能成功创建任意主题，不受认证体系的约束。这显然是 Kafka 集群的运维人员不希望看到的。

- 使用 --bootstrap-server 与集群进行交互，越来越成为使用 Kafka 的标准姿势。换句话说，以后会有越来越少的命令和 API 需要与 ZooKeeper 进行连接。这样，我们只需要一套连接信息，就能与 Kafka 进行全方位的交互，不用像以前一样，必须同时维护 ZooKeeper 和 Broker 的连接信息。

删除topic：删除操作是异步的，仅仅是被标记成“已删除”的状态，真正删除，需要配置文件参数enable。

bin/kafka-topics.sh --bootstrap-server broker_host:port --delete  --topic <topic_name>

**3、查看topiclist**：bin``/kafka-topics``.sh --list --bootstrap-server localhost:9092

**4、查询单个主题的详细数据**：

bin/kafka-topics.sh --bootstrap-server broker_host:port --describe --topic <topic_name>

**5、给某个topic发消息**：bin``/kafka-console-producer``.sh --broker-list localhost:9092 --topic ``test

**6、开启消费者**：bin``/kafka-console-consumer``.sh --bootstrap-server localhost:9092 --topic ``test` `--from-beginning    --group test-group 命令中指定了group。如果未指定，每次会自动生成一个新的消费者组来消费。久而久之会有大量的console-consumer开头的消费者组。from-beginning等同于将Consumer端参数auto.offset.reset设置成earliest，表明从头开始消费主体。如果不指定，会默认从最新位移读取消息。如果此时没有任何新消息，该命令输出为空。  

**7、修改主题分区**，目前不允许减少某个主题的分区，可以用kafka-topics脚本结合-alter参数来增肌主题的分区，命令如下：bin/kafka-topics.sh --bootstrap-server broker_host:port --alter --topic <topic_name> --partitions < 新分区数 > **新分区数一定要比原分区数大，不然会抛异常**

8、修改主题级别参数，主题创建之后，可以使用kafka-configs脚本来修改对应的参数：bin/kafka-configs.sh --zookeeper zookeeper_host:port --entity-type topics --entity-name <topic_name> --alter --add-config max.message.bytes=10485760  **设置常规的主题级别参数，还是用\--Zookeeper**

9、变更副本数，使用自带的kafka-reassign-partitions脚本，帮助我们增加主题的副本数。

10、生产消息使用kafka-console-produ脚本

bin/kafka-console-producer.sh --broker-list kafka-host:port --topic test-topic --request-required-acks -1 --producer-property compression.type=lz4

11、测试生产者及消费者性能 脚本 ：**kafka-producer-perf-test     kafka-consumer-perf-test** 

典型的调用方式：

bin/kafka-producer-perf-test.sh --topic test-topic --num-records 10000000 --throughput -1 --record-size 1024 --producer-props bootstrap.servers=kafka-host:port acks=-1 linger.ms=2000 compression.type=lz4

2175479 records sent, 435095.8 records/sec (424.90 MB/sec), 131.1 ms avg latency, 681.0 ms max latency.
4190124 records sent, 838024.8 records/sec (818.38 MB/sec), 4.4 ms avg latency, 73.0 ms max latency.
10000000 records sent, 737463.126844 records/sec (720.18 MB/sec), 31.81 ms avg latency, 681.00 ms max latency, 4 ms 50th, 126 ms 95th, 604 ms 99th, 672 ms 99.9th.

上述命令向指定主题发送1千万条消息，每条消息大小1KB。命令的输出：

它会打印出测试生产者的吞吐量 (MB/s)、消息发送延时以及各种分位数下的延时。一般情况下，消息延时不是一个简单的数字，而是一组分布。或者说，我们应该关心延时的概率分布情况，仅仅知道一个平均值是没有意义的。这就是这里计算分位数的原因。通常我们关注到99th 分位就可以了。比如在上面的输出中，99th 值是 604ms，这表明测试生产者生产的消息中，有 99% 消息的延时都在 604ms 以内。你完全可以把这个数据当作这个生产者对外承诺的 SLA。

12、**查看主题消息总数**

bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list kafka-host:port --time -2 --topic test-topic

test-topic:0:0
test-topic:1:0

$ bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list kafka-host:port --time -1 --topic test-topic

test-topic:0:5500000
test-topic:1:5500000 使用Kafka提供的工具类 GetOffsetShell来计算给定主题特定分区当前的最早位移和最新位移，两者累加即可得到总的消息数。

>

#### 常见错误

1、主题删除失败。实际上，造成主题删除失败的原因有很多，最常见的原因有两个：副本所在的 Broker 宕机了；待删除主题的部分分区依然在执行迁移过程。如果是因为前者，通常你重启对应的 Broker 之后，删除操作就能自动恢复；如果是因为后者，那就麻烦了，很可能两个操作会相互干扰。不管什么原因，一旦你碰到主题无法删除的问题，可以采用这样的方法：第 1 步，手动删除 ZooKeeper 节点 /admin/delete_topics 下以待删除主题为名的 znode。第 2 步，手动删除该主题在磁盘上的分区目录。第 3 步，在 ZooKeeper 中执行 rmr  /controller，触发 Controller 重选举，刷新 Controller 缓存。在执行最后一步时，你一定要谨慎，因为它可能造成大面积的分区 Leader 重选举。事实上，仅仅执行前两步也是可以的，只是 Controller 缓存中没有清空待删除主题罢了，也不影响使用。

 2：__consumer_offsets 占用太多的磁盘。一旦你发现这个主题消耗了过多的磁盘空间，那么，你一定要显式地用 **jstack 命令**查看一下 kafka-log-cleaner-thread 前缀的线程状态。通常情况下，这都是因为该线程挂掉了，无法及时清理此内部主题。倘若真是这个原因导致的，那我们就只能重启相应的 Broker 了。

#### 消费者组位移

重设位移策略

![offsetpolicy](../images/offsetpolicy.jpeg)

可通过API以及命令行的方式：

Earliest 策略直接指定–to-earliest：

bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group --reset-offsets --all-topics --to-earliest –execute

Latest 策略直接指定–to-latest：

bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group --reset-offsets --all-topics --to-latest --execute

Current 策略直接指定–to-current

bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group --reset-offsets --all-topics --to-current --execute

bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group --reset-offsets --all-topics --to-offset <offset> --execute

bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group --reset-offsets --shift-by <offset_N> --execute

bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group --reset-offsets --to-datetime 2019-06-20T20:00:00.000 --execute

bin/kafka-consumer-groups.sh --bootstrap-server kafka-host:port --group test-group --reset-offsets --by-duration PT0H30M0S --execute

#### 动态调整参数

有可能需要调整的参数：

1. log.retention.ms。修改日志留存时间应该算是一个比较高频的操作，毕竟，我们不可能完美地预估所有业务的消息留存时长。虽然该参数有对应的主题级别参数可以设置，但拥有在全局层面上动态变更的能力，依然是一个很好的功能亮点。

2. num.io.threads 和 num.network.threads。这是我们在前面提到的两组线程池。就我个人而言，我觉得这是动态 Broker 参数最实用的场景了。毕竟，在实际生产环境中，Broker 端请求处理能力经常要按需扩容。如果没有动态 Broker 参数，我们是无法做到这一点的。

3. 与 SSL 相关的参数。主要是 4 个参数（ssl.keystore.type、ssl.keystore.location、ssl.keystore.password 和 ssl.key.password）。允许动态实时调整它们之后，我们就能创建那些过期时间很短的 SSL 证书。每当我们调整时，Kafka 底层会重新配置 Socket 连接通道并更新 Keystore。新的连接会使用新的 Keystore，阶段性地调整这组参数，有利于增加安全性。
4. num.replica.fetchers。这也是我认为的最实用的动态 Broker 参数之一。Follower 副本拉取速度慢，在线上 Kafka 环境中一直是一个老大难的问题。针对这个问题，常见的做法是增加该参数值，确保有充足的线程可以执行 Follower 副本向 Leader 副本的拉取。现在有了动态参数，你不需要再重启 Broker，就能立即在 Follower 端生效，因此我说这是很实用的应用场景。

#### 重设消费者位移

Kafka可以从时间和位移维度来重设位移，具体策略有：

![kafkaresetoffset](../images/kafkaresetoffset.jpeg)





bin/kafka-consumer-groups.sh --bootstrap-server 172.16.6.52:9092, 172.16.6.53:9092,172.16.6.54:9092 --describe --group pro_rev_consumer_award_group_001

bin/kafka-topics.sh --bootstrap-server 172.16.6.52:9092, 172.16.6.53:9092,172.16.6.54:9092,--describe --topic pro_award_compute_rev_topic

