# ElasticSearch

## 1、命令

### 1.1	Cat系列

_cat系列提供了一系列查询elasticsearch集群状态的接口，可以通过执行
curl -XGET localhost:9200/_cat获取所有_cat系列的操作：
/_cat/allocation
/_cat/shards
/_cat/shards/{index}
/_cat/master
/_cat/nodes
/_cat/indices
/_cat/indices/{index}
/_cat/segments
/_cat/segments/{index}
/_cat/count
/_cat/count/{index}
/_cat/recovery
/_cat/recovery/{index}
/_cat/health
/_cat/pending_tasks
/_cat/aliases
/_cat/aliases/{alias}
/_cat/thread_pool
/_cat/plugins
/_cat/fielddata
/_cat/fielddata/{fields}
可以后面加一个v，让输出内容表格显示表头，命令示例：显示所有索引：
curl '10.116.182.65:9200/_cat/indices?v' 

__显示线程信息：
curl '10.110.79.24:9200/_cat/thread_pool?v'
显示结点：
curl '10.110.79.24:9200/_cat/nodes'

### 1.2 cluster系列

1、查询设置集群状态
curl -XGET localhost:9200/`_`cluster/health?pretty=true
pretty=true 表示格式化输出
level=indices 表示显示索引状态
level=shards 表示显示分片信息

2、curl -XGET 10.116.182.65:9200/_cluster/stats?pretty=true 显示集群系统信息，包括CPU JVM等等

3、curl -XGET 10.116.182.65:9200/_cluster/state?pretty=true集群的详细信息。包括节点、分片等

4、curl -XGET 10.116.182.65:9200/_cluster/pending_tasks?pretty=true获取集群堆积的任务

5、修改集群配置 举例：
curl -XPUT localhost:9200/_cluster/settings -d '{
    "persistent" : {
        "discovery.zen.minimum_master_nodes" : 2
    }
}'
transient 表示临时的，persistent表示永久的

6、关闭节点 关闭指定192.168.1.1节点
curl -XPOST 'http://192.168.1.1:9200/_cluster/nodes/_local/_shutdown'
curl -XPOST 'http://localhost:9200/_cluster/nodes/192.168.1.1/_shutdown'
关闭主节点
curl -XPOST 'http://localhost:9200/_cluster/nodes/_master/_shutdown'
关闭整个集群
curl -XPOST 'http://localhost:9200/_shutdown?delay=10s'
curl -XPOST 'http://localhost:9200/_cluster/nodes/_shutdown'
curl -XPOST 'http://localhost:9200/_cluster/nodes/_all/_shutdown'
delay=10s表示延迟10秒关闭

### 1.3 nodes系列

1、查询节点的状态
curl -XGET 'http://localhost:9200/_nodes/stats?pretty=true'
curl -XGET 'http://localhost:9200/_nodes/192.168.1.2/stats?pretty=true'
curl -XGET 'http://localhost:9200/_nodes/process'
curl -XGET 'http://localhost:9200/_nodes/_all/process'
curl -XGET 'http://localhost:9200/_nodes/192.168.......,192.168......./jvm,process'
curl -XGET 'http://localhost:9200/_nodes/192.168.1.2,192.168.1.3/info/jvm,process'
curl -XGET 'http://localhost:9200/_nodes/192.168.1.2,192.168.1.3/_all
curl -XGET 'http://localhost:9200/_nodes/hot_threads

查看所有索引 ：
curl 'localhost:9200/_cat/indices?v' 

获取mapping
curl -XGET http://localhost:9200/{index}/{type}/_mapping?pretty

刷新索引：
curl -XPOST 'http://localhost:9200/kimchy,elasticsearch/_refresh'
curl -XPOST 'http://localhost:9200/_refresh'

统计ES某个索引数据量：
curl -XGET '10.110.79.22:9200/_cat/count/new-sgs-rbil-core-system-dds-next-tcs-server-core-dcn-2017-06-27'

查看模板：
curl -XGET 10.116.182.65:9200/_template/fvp_waybillnewstatus_template
ES慢日志设置：
index.search.slowlog.level: TRACE
index.search.slowlog.threshold.query.warn: 10s
index.search.slowlog.threshold.query.info: 5s
index.search.slowlog.threshold.query.debug: 2s
index.search.slowlog.threshold.query.trace: 500ms
index.search.slowlog.threshold.fetch.warn: 1s
index.search.slowlog.threshold.fetch.info: 800ms
index.search.slowlog.threshold.fetch.debug:500ms
index.search.slowlog.threshold.fetch.trace: 200ms

## 2、调优

### 2.1 系统层面调优

系统层面的调优主要是`内存的设定`与`避免交换内存`。es默认是1g大小，设置堆内存大小两种方式，一种是export设置参数，另一种是启动时指定。即使是大内存机器，也不建议设置超过32g。

~~~powershell
export ES_HEAP_SIZE=10g
./bin/elasticsearch -Xmx10g -Xms10g 
~~~

关于最大分配32个g的原因：

1. 内存对于Elasticsearch来说绝对是重要的，用于更多的内存数据提供更快的操作。而且还有一个内存消耗大户-Lucene

   Lucene的设计目的是把底层OS里的数据缓存到内存中。Lucene的段是分别存储到单个文件中的，这些文件都是不会变化的，所以很利于缓存，同时操作系统也会把这些段文件缓存起来，以便更快的访问。Lucene的性能取决于和OS的交互，如果你把所有的内存都分配给Elasticsearch，不留一点给Lucene，那你的全文检索性能会很差的。最后标准的建议是把50%的内存给elasticsearch，剩下的50%也不会没有用处的，Lucene会很快吞噬剩下的这部分内存。

2. 不分配大内存给Elasticsearch，事实上jvm在内存小于32G的时候会采用一个内存对象指针压缩技术。

   在java中，所有的对象都分配在堆上，然后有一个指针引用它。指向这些对象的指针大小通常是CPU的字长的大小，不是32bit就是64bit，这取决于你的处理器，指针指向了你的值的精确位置。

   对于32位系统，内存最大可使用4G。64系统可以使用更大的内存。但是64位的指针意味着更大的浪费，因为你的指针本身大了。浪费内存不算，更糟糕的是，更大的指针在主内存和缓存器之间移动数据的时候，会占用更多的带宽。

   java 使用一个叫内存指针压缩的技术来解决这个问题。它的指针不再表示对象在内存中的精确位置，而是表示偏移量。这意味着32位的指针可以引用40亿个对象，而不是40亿个字节。最终，也就是说堆内存长到32G的物理内存，也可以用32bit的指针表示。

   一旦你越过那个神奇的30-32G的边界，指针就会切回普通对象的指针，每个对象的指针都变长了，就会使用更多的CPU内存带宽，也就是说你实际上失去了更多的内存。事实上当内存到达40-50GB的时候，有效内存才相当于使用内存对象指针压缩技术时候的32G内存。

设置 ES 集群内存的时候，还有一点就是确保堆内存最小值（Xms）与最大值（Xmx）的大小是相同的，防止程序在运行时改变堆内存大小，这是一个很耗系统资源的过程。

### 2.2 分片与副本

- `分片 (shard)`：ES 是一个分布式的搜索引擎, 索引通常都会分解成不同部分, 分布在不同节点的部分数据就是分片。ES 自动管理和组织分片, 并在必要的时候对分片数据进行再平衡分配, 所以用户基本上不用担心分片的处理细节。创建索引时默认的分片数为 5 个，并且一旦创建不能更改。
- `副本 (replica)`：ES 默认创建一份副本，就是说在 5 个主分片的基础上，每个主分片都相应的有一个副本分片。额外的副本有利有弊，有副本可以有更强的故障恢复能力，但也占了相应副本倍数的磁盘空间。
- 对于副本数，比较好确定，可以根据我们集群节点的多少与我们的存储空间决定，我们的集群服务器多，并且有足够大多存储空间，可以多设置副本数，一般是 1-3 个副本数，如果集群服务器相对较少并且存储空间没有那么宽松，则可以只设定一份副本以保证容灾（副本数可以动态调整）。
- 对于分片数，是比较难确定的。因为一个索引分片数一旦确定，就不能更改，所以我们在创建索引前，要充分的考虑到，以后我们创建的索引所存储的数据量，否则创建了不合适的分片数，会对我们的性能造成很大的影响。

### 2.3 参数调优

如果ES集群CPU使用率高，有可能是写导致的，也有可能是查询导致的先通过 curl -XGET 'http://localhost:9200/_nodes/hot_threads'查看线程，看看是那个线程占用CPU高。如果是 `elasticsearch[{node}][search][T#10]` 则是查询导致的，如果是 `elasticsearch[{node}][bulk][T#1]` 则是数据写入导致的。在实际调优中，cpu 使用率很高，如果不是 SSD，建议把 `index.merge.scheduler.max_thread_count: 1` 索引 merge 最大线程数设置为 1 个，该参数可以有效调节写入的性能。因为在存储介质上并发写，由于寻址的原因，写入性能不会提升，只会降低。

还有几个重要参数可以进行设置：

- `index.refresh_interval`：这个参数的意思是数据写入后几秒可以被搜索到，默认是 1s。每次索引的 refresh 会产生一个新的 lucene 段, 这会导致频繁的合并行为，如果业务需求对实时性要求没那么高，可以将此参数调大，实际调优告诉我，该参数确实很给力，cpu 使用率直线下降。
- `indices.memory.index_buffer_size`：如果我们要进行非常重的高并发写入操作，那么最好将 `indices.memory.index_buffer_size` 调大一些，index buffer 的大小是所有的 shard 公用的，对于每个 shard 来说，最多给 `512mb`，因为再大性能就没什么提升了。ES 会将这个设置作为每个 shard 共享的 index buffer，那些特别活跃的 shard 会更多的使用这个 buffer。默认这个参数的值是 10%，也就是 jvm heap 的 10%。
- `translog`：ES 为了保证数据不丢失，每次 index、bulk、delete、update 完成的时候，一定会触发刷新 translog 到磁盘上。在提高数据安全性的同时当然也降低了一点性能。如果你不在意这点可能性，还是希望性能优先，可以设置如下参数：

~~~
"index.translog": {
        "sync_interval": "120s",     --sync间隔调高
        "durability": "async",       -– 异步更新
        "flush_threshold_size":"1g"  --log文件大小
    }
~~~

这样设定的意思是开启异步写入磁盘，并设定写入的时间间隔与大小，有助于写入性能的提升。

**还有一些超时参数的设置：**

- `discovery.zen.ping_timeout` 判断 master 选举过程中，发现其他 node 存活的超时设置
- `discovery.zen.fd.ping_interval` 节点被 ping 的频率，检测节点是否存活
- `discovery.zen.fd.ping_timeout` 节点存活响应的时间，默认为 30s，如果网络可能存在隐患，可以适当调大
- `discovery.zen.fd.ping_retries ping` 失败/超时多少导致节点被视为失败，默认为 3

### 2.4 其他

- `插入索引自动生成 id`：当写入端使用特定的 id 将数据写入 ES 时，ES 会检查对应的索引下是否存在相同的 id，这个操作会随着文档数量的增加使消耗越来越大，所以如果业务上没有硬性需求建议使用 ES 自动生成的 id，加快写入速率。
- `避免稀疏索引`：索引稀疏之后，会导致索引文件增大。ES 的 keyword，数组类型采用 doc_values 结构，即使字段是空值，每个文档也会占用一定的空间，所以稀疏索引会造成磁盘增大，导致查询和写入效率降低。

## 3、重建索引

在重建索引之前，首先要考虑一下重建索引的必要性，因为重建索引是非常耗时的。ES 的 reindex api 不会去尝试设置目标索引，不会复制源索引的设置，所以我们应该在运行_reindex 操作之前设置目标索引，包括设置映射（mapping），分片，副本等。

第一步，和创建普通索引一样创建新索引。当数据量很大的时候，需要设置刷新时间间隔，把 `refresh_intervals` 设置为-1，即不刷新,number_of_replicas 副本数设置为 0（因为副本数可以动态调整，这样有助于提升速度）。

~~~sql
{
    "settings": {

        "number_of_shards": "50",
        "number_of_replicas": "0",
        "index": {
            "refresh_interval": "-1"
        }
    }
    "mappings": {
    }
}
~~~

第二步，调用 reindex 接口，建议加上 `wait_for_completion=false` 的参数条件，这样 reindex 将直接返回 taskId。

~~~sql
POST _reindex?wait_for_completion=false

{
  "source": {
    "index": "old_index",   //原有索引
    "size": 5000            //一个批次处理的数据量
  },
  "dest": {
    "index": "new_index",   //目标索引
  }
}
~~~

第三步，等待。可以通过 `GET _tasks?detailed=true&actions=*reindex` 来查询重建的进度。如果要取消 task 则调用`_tasks/node_id:task_id/_cancel`。

第四步，删除旧索引，释放磁盘空间。更多细节可以查看 ES 官网的 reindex api。

那么有的同学可能会问，如果我此刻 ES 是实时写入的，那咋办呀？
这个时候，我们就要重建索引的时候，在参数里加上上一次重建索引的时间戳，直白的说就是，比如我们的数据是 100G，这时候我们重建索引了，但是这个 100G 在增加，那么我们重建索引的时候，需要记录好重建索引的时间戳，记录时间戳的目的是下一次重建索引跑任务的时候不用全部重建，只需要在此时间戳之后的重建就可以，如此迭代，直到新老索引数据量基本一致，把数据流向切换到新索引的名字。

~~~sql
baPOST /_reindex
{
    "conflicts": "proceed",          //意思是冲突以旧索引为准，直接跳过冲突，否则会抛出异常，停止task
    "source": {
        "index": "old_index"         //旧索引
        "query": {
          "constant_score" : {
                "filter" : {
                    "range" : {
                        "data_update_time" : {
                            "gte" : 123456789   //reindex开始时刻前的毫秒时间戳
                            }
                        }
                    }
                }
            }
        },
    "dest": {
        "index": "new_index",       //新索引
        "version_type": "external"  //以旧索引的数据为准
        }
}
~~~

## 4、ElasticSearch Circuit Breaker

为了防止OOM，ES包含了多种熔断器。每一种熔断器都指定了es可以用多少内存。除此之外，也有一个父级别的熔断器，指定了通过各个熔断器所能使用的总的内存。除了我们特别说明的，这些设置都可以通过在现有的集群上动态更新。

### 4.1Parent circuit breaker 

可以通过如下配置：

**`indices.breaker.total.use_real_memory`**，当配置为true时，parent breaker会计算真正使用的内存；当设置为false时，只会计算子断路器保留的内存，默认是true。Static setting determining whether the parent breaker should take real memory usage into account (`true`) or only consider the amount that is reserved by child circuit breakers (`false`). Defaults to `true`.

**`indices.breaker.total.limit`**， 当上一个参数为true时，默认是堆内存的95%，为false时，默认是堆内存的70%。 Starting limit for overall parent breaker, defaults to 70% of JVM heap if `indices.breaker.total.use_real_memory` is `false`. If `indices.breaker.total.use_real_memory` is `true`, defaults to 95% of the JVM heap.

### 4.2 Field data circuit breaker

列数据断路器允许es来计算总的字段加载到内存所需要的内存大小。它可以防止加载列超过内存导致的异常。默认配置的是40%的堆内存。可以通过如下参数配置：

**`indices.breaker.fielddata.limit`**，列数据断路器的限制大小，默认40%堆内存；Limit for fielddata breaker, defaults to 40% of JVM heap

**`indices.breaker.fielddata.overhead`** 一个与所有列数据都相乘来确定最终计算的大小的常量，默认是1.03。A constant that all field data estimations are multiplied with to determine a final estimation. Defaults to 1.03

### 4.3 Request circuit breaker

请求断路器允许 Elasticsearch 防止每个请求的数据结构（例如，用于在请求期间计算聚合的内存）超过一定量的内存。通过如下参数设置：

**`indices.breaker.request.limit`**默认是堆内存的60%。Limit for request breaker, defaults to 60% of JVM heap

**`indices.breaker.request.overhead`**，含义同字段数据断路器里的overhead，默认是1。A constant that all request estimations are multiplied with to determine a final estimation. Defaults to 1

### 4.4 In flight requests circuit breaker

In flight断路器允许es限制内存使用，当所有目前活动的传入请求传输总量或 HTTP级别在单节点上超过一定的总量的时候。内存使用情况基于请求本身的内容长度。In flight断路器认为内存不仅需要用来表示原始请求，而且还用作反映结构化对象的默认开销。

**`network.breaker.inflight_requests.limit`**，默认是堆内存的大小。Limit for in flight requests breaker, defaults to 100% of JVM heap. This means that it is bound by the limit configured for the parent circuit breaker.

**`network.breaker.inflight_requests.overhead`**，默认是2。

A constant that all in flight requests estimations are multiplied with to determine a final estimation. Defaults to 2.

### 4.5 Accounting requests circuit breaker

Accounting requests断路器允许 Elasticsearch 限制在请求完成时未释放的内存中的内存使用量，这包括Lucene段内存之类的内容。

**`indices.breaker.accounting.limit`**，默认配置100%堆内存。Limit for accounting breaker, defaults to 100% of JVM heap. This means that it is bound by the limit configured for the parent circuit breaker.

**`indices.breaker.accounting.overhead`**，默认配置1。A constant that all accounting estimations are multiplied with to determine a final estimation. Defaults to 1

### 4.6 Script compilation circuit breaker

与之前的基于内存的断路器有轻微的区别，script compilation断路器在一段时间内限制内联脚本编译的数量。通过如下配置：

**`script.max_compilations_rate`**，默认是5分钟75次。Limit for the number of unique dynamic scripts within a certain interval that are allowed to be compiled. Defaults to 75/5m, meaning 75 every 5 minutes.