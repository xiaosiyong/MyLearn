#### Kafka是什么？

![kafka](../images/kafka学习导图.jpg)

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



#### 压缩算法

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

