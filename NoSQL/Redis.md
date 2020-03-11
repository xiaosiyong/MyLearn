# Redis

## 命令行

### Lua管理脚本的四个命令：

Script Flush  清除服务器中和Lua脚本有关的信息，释放并重建lua_scripts字典，关闭现有的Lua环境并重新创建一个新的Lua环境

Script Exists

Script Load

Script Kill

### Redis 启动

定位到src   make

然后  ...../redis-server   

redis-cli -h 127.0.0.1 -p 6379 shutdown 关闭Redis

redis-cli   slaveof 10.28.51.53 6379  设置slave

Redis-cli  info  显示Redis信息      

## 源码安装redis步骤：

**1、install 源码  make  PREFIX=/usr/loal/redis-4.0.9 install**

**2、启动redis服务**

**3、启动哨兵**

**服务配置：**

daemonize yes

pidfile /mnt/redis/redis_6379/redis.pid

port 6379

bind 172.16.6.228

timeout 600

tcp-keepalive 0

loglevel notice

logfile /mnt/redis/redis_6379/logs/redis.log

databases 16

stop-writes-on-bgsave-error yes

rdbcompression yes

rdbchecksum yes

dbfilename redis_6379.rdb

dir /mnt/redis/redis_6379/data/

maxclients 4096

maxmemory 1024mb

maxmemory-policy volatile-lru

appendonly yes

appendfilename redis_6379.aof

appendfsync everysec

no-appendfsync-on-rewrite no

auto-aof-rewrite-percentage 100

auto-aof-rewrite-min-size 64mb

lua-time-limit 5000

slowlog-log-slower-than 10000

slowlog-max-len 128

hash-max-ziplist-entries 512

hash-max-ziplist-value 64

list-max-ziplist-entries 512

list-max-ziplist-value 64

set-max-intset-entries 512

zset-max-ziplist-entries 128

zset-max-ziplist-value 64

activerehashing yes

client-output-buffer-limit normal 0 0 0

client-output-buffer-limit slave 256mb 64mb 60

client-output-buffer-limit pubsub 32mb 8mb 60

hz 10

aof-rewrite-incremental-fsync yes

rename-command FLUSHDB ISS_FLUSHDB

rename-command FLUSHALL ISS_FLUSHALL

rename-command KEYS ISS_KEYS

**哨兵配置：**

Redis 启动

/usr/local/redis/src/redis-server /mnt/redis/redis_6379/conf/redis.conf

/usr/local/redis-3.2.11/src/redis-cli -h 10.28.51.53 -p 6379

/mnt/opt/mongodb-4.0.2/bin/mongo --host=10.28.55.56:27017

========================================================================================

## Redis 两种不同的持久化模式：RDB与AOF

### 1、RDB 快照模式，

该模式用于生成某个时间点的备份信息，并且会对当前的key value进行编码存储到rdb文件中。RDB持久化可以**手动**执行，也可以根据服务器配置**定期**执行。RDB持久化所生成的RDB文件是一个经过**压缩**的二进制文件，Redis可以通过这个文件**还原**数据库的数据。

有两个命令可以生成RDB文件：

- `SAVE`会**阻塞**Redis服务器进程，服务器不能接收任何请求，直到RDB文件创建完毕为止。
- `BGSAVE`创建出一个**子进程**，由子进程来负责创建RDB文件，服务器进程可以继续接收请求。

Redis服务器在启动的时候，如果发现有RDB文件，就会**自动**载入RDB文件(不需要人工干预)，但服务器在载入RDB文件期间，会处于阻塞状态，直到载入工作完成。除了手动调用`SAVE`或者`BGSAVE`命令生成RDB文件之外，我们可以使用配置的方式来**定期**执行：

在默认的配置下，如果以下的条件被触发，就会执行**BGSAVE**命令

~~~shell
    save 900 1              #在900秒(15分钟)之后，至少有1个key发生变化，
    save 300 10            #在300秒(5分钟)之后，至少有10个key发生变化
    save 60 10000        #在60秒(1分钟)之后，至少有10000个key发生变化
~~~

原理大概就是这样子的(结合上面的配置来看)：

```c
struct redisServer{
    // 修改计数器
    long long dirty;
  	// 上一次执行保存的时间
    time_t lastsave;
  	// 参数的配置
    struct saveparam *saveparams;
};
```

遍历参数数组，判断修改次数和时间是否符合，如果符合则调用`besave()`来生成RDB文件。

![redissave](../images/redissave.png)

### 2、AOF 持久化模式，

该模式类似binlog的形式，会记录服务器所有的写请求，在服务重启的时候通过回放执行命令请求来恢复原有的数据。如图：

![redisaof](../images/redisaof.png)

AOF文件记录的是原始的Redis写请求命令，所以在了解AOF文件之前我们需要了解下Redis协议。

Redis客户端和服务端之间可以通过RESP (REdis Serialization Protocol)来进行通信，作者在设计这个协议主要依据了以下三点：

- 易实现
- 解析速度快
- 容易被人类理解

RESP协议主要由以下几种数据类型组成：简单字符串、错误信息、整数、字符串、数组。客户端发送给服务端的是一个数组命令，服务端根据不同命令的实现进行回复。每个数据类型的定义如下：

- 简单字符串： 以+号开头结尾为rn，比如+OKrn
- 错误信息： 以-号开头结尾为rn的字符串，比如-ERR Readonlyrn
- 整数: 以:开头结尾为rn，开头和结尾之间为整数，比如:1rn
- 字符串: 以$开头随后为该字符串长度和rn，接下去为真正的字符串内容和rn
- 数组: 以*****开头的，随后指定了数组元素个数并通过rn划分，每个数组元素都可以为上面的四种，比如*1rn$4rnpingrn

AOF持久化功能的实现可以分为3个步骤：

命令追加：命令写入aof_buf缓冲区

文件写入：调用flushAppendOnlyFile函数，考虑是否要将aof_buf缓冲区写入AOF文件中。该函数的行为由服务器配置的**appendfsyn**选项来决定：

~~~shell
 appendfsync always     # 每次有数据修改发生时都会写入AOF文件。
    appendfsync everysec   # 每秒钟同步一次，该策略为AOF的默认策略。
    appendfsync no         # 从不同步。高效但是数据不会被持久化。
~~~

文件同步：考虑是否将内存缓冲区的数据真正写入到硬盘（操作系统在写文件的时候，很多都是先把数据保存在一个内存缓冲区里，等到内存缓冲区满了才将缓冲区的数据写入磁盘）。

### AOF重写

我们写了3条命令，AOF文件就保存了三条命令。如果命令长这样：

~~~shell
redis > RPUSH list "Java" "3y"
(integer)2

redis > RPUSH list "Java3y"
integer(3)

redis > RPUSH list "yyy"
integer(4)

~~~

那么，AOF也会保存3条命令。我们发现，上面的命令是可以合并起来成为1条命令的，并不需要3条。这样可以让AOF文件的体积变得更小。AOF重写由Redis自行触发(参数配置)，也可以用`BGREWRITEAOF`命令**手动触发**重写操作。要值得说明的是：**AOF重写不需要对现有的AOF文件进行任何的读取、分析。AOF重写是通过读取服务器当前数据库的数据来实现的**！

比如，现有一个Redis数据库的数据如下：

![redisdb](../images/redisdb.png)

新的AOF文件的命令如下：

![redisraof](../images/redisraof.png)

### AOF 后台重写

Redis将AOF重写程序放到**子进程**里执行(`BGREWRITEAOF`命令)，像`BGSAVE`命令一样fork出一个子进程来完成重写AOF的操作，从而不会影响到主进程。为了解决数据不一致的问题，Redis服务器设置了一个**AOF重写缓冲区**，这个缓存区会在服务器**创建出子进程之后使用**。如图：

![bgraof](../images/bgraof.png)

#### RDB和AOF对过期键的策略

RDB持久化对过期键的策略：

- 执行`SAVE`或者`BGSAVE`命令创建出的RDB文件，程序会对数据库中的过期键检查，**已过期的键不会保存在RDB文件中**。
- 载入RDB文件时，程序同样会对RDB文件中的键进行检查，**过期的键会被忽略**。

AOF持久化对过期键的策略：

- 如果数据库的键已过期，但还没被惰性/定期删除，AOF文件不会因为这个过期键产生任何影响(也就说会保留)，当过期的键被删除了以后，会追加一条DEL命令来显示记录该键被删除了
- 重写AOF文件时，程序会对RDB文件中的键进行检查，**过期的键会被忽略**。

#### RDB和AOF选择

RDB和AOF并不互斥、他俩可以同时使用。

- RDB的优点：载入时**恢复数据快**、文件体积小。
- RDB的缺点：会一定程度上**丢失数据**(因为系统一旦在定时持久化之前出现宕机现象，此前没有来得及写入磁盘的数据都将丢失。)
- AOF的优点：丢失数据少(默认配置只丢失一秒的数据)。
- AOF的缺点：恢复数据相对较慢，文件体积大

如果Redis服务器**同时开启**了RDB和AOF持久化，服务器会**优先使用AOF文件**来还原数据(因为AOF更新频率比RDB更新频率要高，还原的数据更完善)

#### 分布式锁：

1、SET resource_name my_random_value NX PX max-lock-time

原子操作保证锁的持有者解锁是同一个、设置过期时间防止死锁

该方案的问题：

1) 通过过期时间来避免死锁，过期时间设置多长对业务来说往往比较头疼，时间短了可能会造成：持有锁的线程A任务还未处理完成，锁过期了，线程B获得了锁，导致同一个资源被A、B两个线程并发访问；时间长了会造成：持有锁的进程宕机，造成其他等待获取锁的进程长时间的无效等待

2) redis的主从异步复制机制可能丢失数据，会出现如下场景：A线程获得了锁，但锁数据还未同步到slave上，master挂了，slave顶成主，线程B尝试加锁，仍然能够成功，造成A、B两个线程并发访问同一个资源

2、通过zookeeper

加锁流程

1) 在/resource_name节点下创建临时有序节点

2) 获取当前线程创建的节点及/resource_name目录下的所有子节点，确定当前节点序号是否最小，是则加锁成功。否则监听序号较小的前一个节点

注：zab一致性协议保证了锁数据的安全性，不会因为数据丢失造成多个锁持有者；心跳保活机制解决死锁问题，防止由于进程挂掉或者僵死导致的锁长时间被无效占用。具备阻塞锁特性，并通过watch机制能够及时从阻塞状态被唤醒

解锁流程

1) 删除当前线程创建的临时接点

问题：通过心跳保活机制解决死锁会造成锁的不安全性，可能会出现如下场景：持有锁的线程A僵死或网络故障，导致服务端长时间收不到来自客户端的保活心跳，服务端认为客户端进程不存活主动释放锁，线程B抢到锁，线程A恢复，同时有两个线程访问共享资源



#### 理想的锁设计目标主要应该解决如下问题：

1、锁数据本身的安全性

2、不发生死锁

3、不会有多个线程同时持有相同的锁

实现不发生死锁：

1、锁设置过期时间，过期之后Server端自动释放锁

2、对锁的持有进程进行探活，发现持锁进程不存活时Server端自动释放

#### Redis内存分析方法

一般会采用bgsave生成dump.rdb文件，再结合redis-rdb-tools和sqlite来进行静态分析。使用redis-rdb-tools生成内存快照，命令：

~~~shell
rdb -c memory dump.rdb > memory.csv
~~~

生成CSV格式的内存报告。包含的列有：数据库ID，数据类型，key，内存使用量(byte)，编码。内存使用量包含key、value和其他值。
注意：内存使用量是理论上的近似值，在一般情况下，略低于实际值。
memory.csv例子：

~~~shell
$head memory.csv
database,type,key,size_in_bytes,encoding,num_elements,len_largest_element
0,string,"orderAt:377671748",96,string,8,8
0,string,"orderAt:413052773",96,string,8,8
0,sortedset,"Artical:Comments:7386",81740,skiplist,479,41
0,sortedset,"pay:id:18029",2443,ziplist,84,16
0,string,"orderAt:452389458",96,string,8,8
~~~

#### 分析内存快照

SQLite，是一款轻型的数据库。我们可以将前面生成的csv导入到数据库中之后，就可以利用sql语句很方便的对Redis的内存数据进行各种分析了。
导入方法：

~~~sqlite
sqlite3 memory.db
sqlite> create table memory(database int,type varchar(128),key varchar(128),size_in_bytes int,encoding varchar(128),num_elements int,len_largest_element varchar(128));
sqlite>.mode csv memory
sqlite>.import memory.csv memory
~~~

数据导入之后，直接查询分析：

~~~sqlite
sqlite>select count(*) from memory;//查询key个数
sqlite>select sum(size_in_bytes) from memory;//查询总的内存占用
sqlite>select * from memory order by size_in_bytes desc limit 10;//查询内存占用最高的10个key
sqlite>select * from memory where type='list' and num_elements > 1000 ;//查询成员个数1000个以上的lis
~~~

#### Redis应用场景：

缓存、排行榜（京东月度销量榜单，商品按时间的上新排行）、计数器（浏览量、播放量 incr 命令实现计数器功能）、分布式会话、分布式锁（全局ID、减库存、秒杀等场景 setnx功能）、社交网络（点赞、踩、关注……）、最新列表（Redis列表结构，LPUSH可以在列表头部插入一个内容ID作为关键字，LTRIM可用来限制列表的数量，这样列表永远为N个ID，无需查询最新的列表，直接根据ID去到对应的内容页即可）。

---

## 数据类型

### 常用数据类型：

String、Hash、Set、List、SortedSet、pub/sub、Transaction

1、String：Strings就是一个最最简单的Key-Value形式存储的变量。其中Value既可以是数字也可以是字符串。其实现方式是在Redis内部默认存储一个字符串，被redisObject引用，当检测到数字操作如自增自减incr、decr等等命令时，自动转化为数字进行计算，计算完毕后再转化为String存储起来。

2、Hash:Hash存储是键值对的value。即Key-Hash，而Hash又是一个k-v的结构，如果使用的Memcached，则需要把整个Hash打包存储在内存中，如果需要查询其中某个值，还要全部取出整个Hash，再查找对应值。而Redis可以直接通过命令获取到Value，大大提高了性能。 其实现原理：当成员较少时，Redis为了节约内存会采用类似一维数组的紧凑存储，而当对象较多时，则直接转为HashMap存储。

3、Set：Set是一个无序的天然去重的集合，即Key-Set。此外还提供了交集、并集等一系列直接操作集合的方法，对于求共同好友、共同关注什么的功能实现特别方便。其底层是靠HashMap实现的，其中value为null；

4、List：List是一个有序可重复的集合，其遵循FIFO的原则，底层是依赖双向链表实现的，因此支持正向、反向双重查找。通过List，我们可以很方面的获得类似于最新回复这类的功能实现。

5、SortedSet：类似于java中的TreeSet，是Set的可排序版。此外还支持优先级排序，维护了一个score的参数来实现。其底层主要依赖HashMap来实现的，通过维持插入的数值和Score优先级的映射来进行排序。

6、pub/sub：发布订阅，类似于消息队列mq。可以选择对某个Key进行订阅，一旦这个key发布了一些消息，则所有订阅了这个Key的对象就可以收到这个消息。主要可以用在实时消息系统上，例如聊天之类的。

7、Transactions：NoSQL不支持事务，但是通过提供了打包执行的功能，即这个包里面的所有命令必须要一起执行，此外还可以锁定某个Key，在打包执行命令时如果检测到这个Key发生了变化，则直接回滚。

### 底层数据结构

1. SDS

~~~c
struct sdshdr{  
     //记录buf数组中已使用字节的数量  
     //等于 SDS 保存字符串的长度  
     int len;  
     //记录 buf 数组中未使用字节的数量  
     int free;  
     //字节数组，用于保存字符串  
     char buf[];  
} 
~~~

C语言处理字符串和数组的成本是很高的，还容易出现以下问题：

- 极其容易造成缓冲区溢出问题，比如用strcat()，在用这个函数之前必须要先给目标变量分配足够的空间，否则就会溢出。
- 如果要获取字符串的长度，没有数据结构的支撑，可能就需要遍历，它的复杂度是O(N)
- 内存重分配。C字符串的每次变更(曾长或缩短)都会对数组作内存重分配。同样，如果是缩短，没有处理好多余的空间，也会造成内存泄漏。

基于以上，Redis自己构建了一种名叫Simple dynamic string(SDS)的数据结构，源码如上图。优点：

- 开发者不用担心字符串变更造成的内存溢出问题。
-  常数时间复杂度获取字符串长度len字段。
-  空间预分配free字段，会默认留够一定的空间防止多次重分配内存。

2. **链表**

Redis的链表在双向链表上扩展了头、尾节点、元素数等属性。如下图：![redis-linklist](../images/redis-linklist.jpg)

源码：

~~~c
typedef  struct listNode{  
       //前置节点  
       struct listNode *prev;  
       //后置节点  
       struct listNode *next;  
       //节点的值  
       void *value;    
}listNode

typedef struct list{  
     //表头节点  
     listNode *head;  
     //表尾节点  
     listNode *tail;  
     //链表所包含的节点数量  
     unsigned long len;  
     //节点值复制函数  
     void (*free) (void *ptr);  
     //节点值释放函数  
     void (*free) (void *ptr);  
     //节点值对比函数  
     int (*match) (void *ptr,void *key);  
}list; 
~~~

从上面可以看到，Redis的链表有这几个特点：

1.  可以直接获得头、尾节点。
2.  常数时间复杂度得到链表长度。
3.  是双向链表。

3. **字典(Hash)**

Redis的Hash，就是在数组+链表的基础上，进行了一些rehash优化等。![redis-hash](../images/redis-hash.jpg)

源码结构：

~~~c
//哈希表
typedef struct dictht {  
    // 哈希表数组  
    dictEntry **table;  
    // 哈希表大小  
    unsigned long size;  
    // 哈希表大小掩码，用于计算索引值  
    // 总是等于 size - 1  
    unsigned long sizemask;  
    // 该哈希表已有节点的数量  
    unsigned long used;  
} dictht; 

//Hash表节点
typedef struct dictEntry {  
    // 键  
    void *key;  
    // 值  
    union {  
        void *val;  
        uint64_t u64;  
        int64_t s64;  
    } v;  
    // 指向下个哈希表节点，形成链表  
    struct dictEntry *next;  // 单链表结构  
} dictEntry; 

//字典
typedef struct dict {  
    // 类型特定函数  
    dictType *type;  
    // 私有数据  
    void *privdata;  
    // 哈希表  
    dictht ht[2];  
    // rehash 索引  
    // 当 rehash 不在进行时，值为 -1  
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */  
} dict; 
~~~

有源码可以看出：

- Reids的Hash采用链地址法来处理冲突，然后它没有使用红黑树优化。
- 哈希表节点采用单链表结构。
- rehash优化

具体优化规则：

- 我们仔细可以看到dict结构里有个字段dictht ht[2]代表有两个dictht数组。第一步就是为ht[1]哈希表分配空间，大小取决于ht[0]当前使用的情况。
- 将保存在ht[0]中的数据rehash(重新计算哈希值)到ht[1]上。
- 当ht[0]中所有键值对都迁移到ht[1]后，释放ht[0]，将ht[1]设置为ht[0]，并ht[1]初始化，为下一次rehash做准备。

redis考虑到大量数据迁移带来的cpu繁忙(可能导致一段时间内停止服务)，所以采用了**渐进式rehash**的方案。步骤如下

- 为ht[1]分配空间，同时持有两个哈希表(一个空表、一个有数据)。
- 维持一个技术器rehashidx，初始值0。
-  每次对字典增删改查，会顺带将ht[0]中的数据迁移到ht[1],rehashidx++(注意：ht[0]中的数据是只减不增的)。
- 直到rehash操作完成，rehashidx值设为-1。

它的好处：采用分而治之的思想，将庞大的迁移工作量划分到每一次CURD中，避免了服务繁忙。

4. 跳跃表

**skipList & AVL 之间的选择**

1. 从算法实现难度上来比较，skiplist比平衡树要简单得多。
2.  平衡树的插入和删除操作可能引发子树的调整，逻辑复杂，而skiplist的插入和删除只需要修改相邻节点的指针，操作简单又快速。
3.  查找单个key，skiplist和平衡树的时间复杂度都为O(log n)，大体相当。
4.  在做范围查找的时候，平衡树比skiplist操作要复杂。
5.  skiplist和各种平衡树（如AVL、红黑树等）的元素是有序排列的。

可以看到，skipList中的元素是有序的，所以跳跃表在redis中用在**有序集合键、集群节点内部数据结构。**

源码：

~~~c
//跳跃表节点
typedef struct zskiplistNode {  
    // 后退指针  
    struct zskiplistNode *backward;  
    // 分值  
    double score;  
    // 成员对象  
    robj *obj;  
    // 层  
    struct zskiplistLevel {  
        // 前进指针  
        struct zskiplistNode *forward;  
        // 跨度  
        unsigned int span;  
    } level[];  
} zskiplistNode;

//跳跃表
typedef struct zskiplist {  
    // 表头节点和表尾节点  
    struct zskiplistNode *header, *tail;  
    // 表中节点的数量  
    unsigned long length;  
    // 表中层数最大的节点的层数  
    int level;  
} zskiplist; 
~~~

层：也就是level[]字段，层的数量越多，访问节点速度越快。(因为它相当于是索引，层数越多，它索引就越细，就能很快找到索引值)

前进指针：层中有一个forward字段，用于从表头向表尾方向访问。

跨度（span）：用于记录两个节点之间的距离。

后退指针：用于从表尾向表头方向访问。

5. 整数集合

Reids对整数存储专门作了优化，intset就是redis用于保存整数值的集合数据结构，可以保存类型为int16_t、int32_t、int64_t的整数值，并且保证集合中不会出现重复元素。当一个结合中只包含整数元素，而且数量不多时，redis就会用这个来存储。

~~~c
typedef struct intset {
    // 编码方式
    uint32_t encoding;
    // 集合包含的元素数量
    uint32_t length;
    // 保存元素的数组
    int8_t contents[];
} intset;
~~~

- `contents`数组：整数集合的每个元素在数组中按值的大小从小到大排序，且不包含重复项
- length`记录整数集合的元素数量，即contents数组长度`
- encoding`决定contents数组的真正类型，如INTSET_ENC_INT16、INTSET_ENC_INT32、INTSET_ENC_INT64

~~~shell
127.0.0.1:6379[2]> sadd number -6370 -5 18 233 14672
(integer) 5  
127.0.0.1:6379[2]> object encoding number  
"intset" 
~~~

![redisintset](../images/redisintset.png)

6. 压缩列表**ziplist**

ziplist是redis为了节约内存而开发的顺序型数据结构。它被用在列表键和哈希键中。一般用于小数据存储。

7. 快速列表 **quicklist**

---

#### Redis对比Memcache

1、数据类型更丰富

2、Redis支持数据的备份，master-slave模式的数据备份

3、Redis支持数据持久化，可将内存中你那个的数据保存在磁盘中，重启时可以再次加载进行使用，而Memcache把数据全存在内存

4、Memcache是多线程，非阻塞IO复用的网络模型，，分为监听主线程和worker子线程，监听线程监听网络连接，接受请求后，将连接描述字pipe传递给worker线程，进行读写IO，网络层使用libevent封装的事件库，多线程模型可以发挥多核作用，但是引入了cache concurrency和锁的问题，比如：memcached最常用的stats命令，实际memcached所有操作都要对这个全局变量加锁，进行计数等工作，带来了性能损耗。Redis使用单线程的IO复用模型，自己封装了一个简单的AeEvent事件处理框架，主要实现了epoll, kqueue和select，对于单纯只有IO操作来说，单线程可以将速度优势发挥到最大，但是redis也提供了一些简单的计算功能，比如排序、聚合等，对于这些操作，单线程模型施加会严重影响整体吞吐量，CPU计算过程中，整个IO调度都是被阻塞的。

5、内存管理机制不同。Memcached的内存模式，官方定义为 ”Slab Allocation“，大致流程图如下：

![Memcacheproc](../images/Memcacheproc.jpeg)

1、chunk是Memcached用来存储数据的最小单位，就好像一个盒子，每个盒子里装的是我们的午饭一样。Memcache这样设计的初衷是为了尽量减少内存碎片的问题。

2、slab和page是更大一些的盒子，用于承装不同尺寸的Chunk，Chunk的大小是通过Factor【自增长因子决定】

3、不同尺寸的Chunk最终会交给一个”目录“进行管理，以便于访问，而这个目录叫slab_class。数据访问流程： 客户端会现在slab_class里找到尺寸合适的Slab，并且通过一定的方式找到Chunk，最终保证数据会进入一个更合适的”盒子“，从而减少内存的浪费。

Redis在这一方面的处理相对简单，大致的形式如下：

![redisproc](../images/redisproc.jpeg)

Redis每一个数据块都是根据数据类型和大小进行分配的，这一块数据的元数据（比如数据块大小）会存入内存块的头部，real_ptr是redis调用malloc后返回的指针。redis将内存块的大小size存入头部，size所占据的内存大小是已知的，为size_t类型的长度，然后返回ret_ptr。当需要释放内存的时候，ret_ptr被传给内存管理程序。通过ret_ptr，程序可以很容易的算出real_ptr的值，然后将real_ptr传给free释放内存。

总的来说，Memcached使用预分配的内存池的方式，使用slab和大小不同的chunk来管理内存，Item根据大小选择合适的chunk存储，内存池的方式可以省去申请/释放内存的开销，并且能减小内存碎片产生，但这种方式也会带来一定程度上的空间浪费；Redis使用现场申请内存的方式来存储数据，并且很少使用free-list等方式来优化内存分配，会在一定程度上存在内存碎片，Redis跟据存储命令参数，会把带过期时间的数据单独存放在一起，并把它们称为临时数据，非临时数据是永远不会被剔除的，即便物理内存不够，导致swap也不会剔除任何非临时数据（但会尝试剔除部分临时数据），这点上Redis更适合作为存储而不是cache。

1、Redis内存空间的利用比Memcahced更精细，因为Memcached是用一个“盒子”对数据进行承载，哪怕这个盒子的尺寸再合适，也不可避免的会有空置；2、Memcached完美的解决了内存碎片的问题；3、Memcached内部还存在一个slot的机制，对内存的使用优先使用废弃内存，在内存的重复利用上也具有一定的优势；4、Redis并不是将所有内存数据都存放在内存中，只会将所有的key存放在内存，在读取的时候会有一定几率存在一次IO操作，在这一点上，Redis是使用时间换取了空间的策略；

#### Redis 源码存储结构

~~~c
typedef struct redisDb { 
    int id;         // 数据库ID标识
    dict *dict;     // 键空间，存放着所有的键值对              
    dict *expires;  // 过期哈希表，保存着键的过期时间                          
    dict *watched_keys; // 被watch命令监控的key和相应client    
    long long avg_ttl;  // 数据库内所有键的平均TTL（生存时间）     
} redisDb;
~~~

从代码上我们可以发现最重要的应该是`dict *dict`，它用来存放着所有的键值对。一般我们将存储所有键值对的`dict`称为**键空间**。示意图：

![rediskey](../images/rediskey.png)

Redis的数据库就是使用字典(哈希表)来作为底层实现的，对**数据库的增删改查都是构建在字典(哈希表)的操作之上的**。

#### 键的过期时间

设置键的生存时间可以通过`EXPIRE`或者`PEXPIRE`命令，设置键的**过期**时间可以通过`EXPIREAT`或者`PEXPIREAT`命令。其实`EXPIRE`、`PEXPIRE`、`EXPIREAT`这三个命令都是通过`PEXPIREAT`命令来实现的。我们在redisDb结构体中还发现了`dict *expires;`属性，存放所有键过期的时间。如执行命令：

~~~shell
EXPIREAT message 1391234400000
~~~

执行完之后：

![redisexpire](../images/redisexpire.png)

移除过期时间以及查看剩余生存时间的命令：

PERSIST(移除过期时间)、TTL(Time To Live)返回剩余生存时间，以秒为单位、PTTL以毫秒为单位返回键的剩余生存时间

#### 过期策略

过期键是保存在哈希表中了。那这些过期键到了过期的时间，何时会被删掉呢？这与Redis的过期策略有关，删除策略可以分为三种：

1、定时删除（对内存友好，对CPU不友好），到时间点上就把所有过期的键删除了

2、惰性删除（对CPU极度友好，对内存极度不友好），每次从键空间取键的时候，判断一下该键是否过期了，如果过期了就删除

3、定期删除（折中），每隔一段时间去删除过期键，限制删除的执行时长和频率。

Redis采用的是**惰性删除+定期删除**两种策略，所以说，在Redis里边如果过期键到了过期的时间了，未必被立马删除的！

#### Redis数据淘汰策略

- **volatile-lru**，从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
- **volatile-ttl**，从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰，优先删除剩余时间(time to live,TTL) 短的key。
- **volatile-random**，从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
- **allkeys-lru**，从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
- **allkeys-random**，从数据集（server.db[i].dict）中任意选择数据淘汰
- **no-enviction（默认使用的策略）**，（驱逐）：禁止驱逐数据 不删除策略, 达到最大内存限制时, 如果需要更多内存, 直接返回错误信息。

Redis回收使用的算法：**LRU 算法**

注意LFU和LRU算法的不同之处，LRU的淘汰规则是基于访问时间，而LFU是基于访问次数的。举个简单的例子：假设缓存大小为3，数据访问序列为set(2,2),set(1,1),get(2),get(1),get(2),set(3,3),set(4,4)，则在set(4,4)时对于LFU算法应该淘汰(3,3)，而LRU应该淘汰(1,1)。

Redis大量数据插入：从Redis 2.6开始`redis-cli`支持一种新的被称之为**pipe mode**的新模式用于执行大量数据插入工作。

### Redis分区

#### 分区的优势

- 通过利用多台计算机内存的和值，允许我们构造更大的数据库。
- 通过多核和多台计算机，允许我们扩展计算能力；通过多台计算机和网络适配器，允许我们扩展网络带宽。

#### 分区的不足

redis的一些特性在分区方面表现的不是很好：

- 涉及多个key的操作通常是不被支持的。举例来说，当两个set映射到不同的redis实例上时，你就不能对这两个set执行交集操作。
- 涉及多个key的redis事务不能使用。
- 当使用分区时，数据处理较为复杂，比如你需要处理多个rdb/aof文件，并且从多个实例和主机备份持久化文件。
- 增加或删除容量也比较复杂。redis集群大多数支持在运行时增加、删除节点的透明数据平衡的能力，但是类似于客户端分区、代理等其他系统则不支持这项特性。

#### 分区类型：范围分区和哈希分区

最简单的分区方式是按范围分区，就是映射一定范围的对象到特定的Redis实例。比如，ID从0到10000的用户会保存到实例R0，ID从10001到 20000的用户会保存到R1，以此类推。这种方式是可行的，并且在实际中使用，不足就是要有一个区间范围到实例的映射表。这个表要被管理，同时还需要各 种对象的映射表，通常对Redis来说并非是好的方法。

#### Redis常见性能问题和解决方案:

1. Master最好不要做任何持久化工作，如RDB内存快照和AOF日志文件
2. 如果数据比较重要，某个Slave开启AOF备份数据，策略设置为每秒同步一次
3. 为了主从复制的速度和连接的稳定性，Master和Slave最好在同一个局域网内
4. 尽量避免在压力很大的主库上增加从库

#### Redis分布式锁

lua脚本可以同时把setnx和expire合成一条指令来用的，这样不会出现setnx expire之前宕机死锁的问题

redis keys指令遍历会阻塞，可以用scan代替，但是会有重复的，需要客户端去重。Redis队列，一般使用list结构作为队列，rpush生产消息，lpop消费消息。当lpop没有消息的时候，要适当sleep一会再重试。list还有个指令叫blpop，在没有消息的时候，它会阻塞住直到消息到来。redis 通过 sortedset 可实现延时队列。拿时间戳作为score，消息内容作为key调用zadd来生产消息，消费者用zrangebyscore指令获取N秒之前的数据轮询进行处理。

#### Redis单线程为什么快？

#### IO多路复用

- I/O多路复用的特点是通过一种机制**一个进程能同时等待多个文件描述符**，而这些文件描述符其中的任意一个进入**读就绪状态、等等**，`select()`函数就可以返回。
- select/epoll的优势并不是对于单个连接能处理得更快，而是**在于能处理更多的连接**。

#### BitMap以及HyperLogLog的应用

### Redis面试必备

#### 1、常用指令![redis-command-often](../images/redis-command-often.png)

#### 2、场景解析

##### 2.1 String 

如图：![redis-command-often](../images/redis-command-often.png)

String类型使用场景

**场景一：商品库存数**

从业务上，商品库存数据是热点数据，交易行为会直接影响库存。而 Redis 自身 String 类型提供了：

1. set goods_id 10; 设置 id 为 good_id 的商品的库存初始值为 10；
2. decr goods_id; 当商品被购买时候，库存数据减 1。

**依次类推的场景：**商品的浏览次数，问题或者回复的点赞次数等。这种计数的场景都可以考虑利用 Redis 来实现。

**场景二：时效信息存储**

Redis 的数据存储具有自动失效能力。也就是存储的 key-value 可以设置过期时间：set(key, value, expireTime)。

比如，用户登录某个 App 需要获取登录验证码， 验证码在 30 秒内有效。那么我们就可以使用 String 类型存储验证码，同时设置 30 秒的失效时间。

![redis-string-scence](../images/redis-string-scence)

##### 2.2 Hash存储

![redis-hash-save](../images/redis-hash-save))

Hash 类型使用场景

Redis 在存储对象（例如：用户信息）的时候需要对对象进行序列化转换然后存储。还有一种形式，就是将对象数据转换为 JSON 结构数据，然后存储 JSON 的字符串到 Redis。对于一些对象类型，还有一种比较方便的类型，那就是按照 Redis 的 Hash 类型进行存储。

例如，我们存储一些网站用户的基本信息， 我们可以使用：这样就存储了一个用户基本信息，存储信息有：{name : 小明， phone : “123456”，sex : “男”}当然这种类似场景还非常多， 比如存储订单的数据，产品的数据，商家基本信息等。

Hash实现信息存储的优缺点

1. 原生:

- set user: 1:name james; 
- set user:1:age 23;
- set user:1:sex boy;

**优点:**简单直观，每个键对应一个值 **缺点:**键数过多，占用内存多，用户信息过于分散，不用于生产环境 

2. 将对象序列化存入

redis set user:1 serial ize (userInfo);

**优点:**编程简单，若使用序列化合理内存使用率高 **缺点:**序列化与反序列化有一定开销，更新属性时需要把userInfo全取出来进行反序列化，更新后再序列化到redis

3. hash存储

hmset user:1 name james age 23 sex boy 

**优点:**简单直观，使用合理可减少内存空间消耗 **缺点:**要控制ziplist 与hashtable两种编码转换，Mhashtable会消耗更多内存。 

##### 2.3 List类型及使用场景

list 是按照插入顺序排序的字符串链表。可以在头部和尾部插入新的元素（双向链表实现，两端添加元素的时间复杂度为 O(1)） 。![redis-list](../images/redis-list)

**场景一：消息队列实现**

目前有很多专业的消息队列组件 Kafka、RabbitMQ 等。 我们在这里仅仅是使用 list 的特征来实现消息队列的要求。在实际技术选型的过程中，大家可以慎重思考。

**list 存储就是一个队列的存储形式：**

1. lpush key value; 在 key 对应 list 的头部添加字符串元素；
2. rpop key;移除列表的最后一个元素，返回值为移除的元素。

**场景二：最新上架商品**

在交易网站首页经常会有新上架产品推荐的模块， 这个模块是存储了最新上架前 100 名。这时候使用 Redis 的 list 数据结构，来进行 TOP 100 新上架产品的存储。Redis ltrim 指令对一个列表进行修剪（trim），这样 list 就会只包含指定范围的指定元素。start 和 stop 都是由 0 开始计数的，这里的 0 是列表里的第一个元素（表头），1 是第二个元素。

##### 2.4 **set 类型使用场景**

set 也是存储了一个集合列表功能。和 list 不同，set 具备去重功能。当需要存储一个列表信息，同时要求列表内的元素不能有重复，这时候使用 set 比较合适。与此同时，set 还提供的交集、并集、差集。

例如，在交易网站，我们会存储用户感兴趣的商品信息，在进行相似用户分析的时候， 可以通过计算两个不同用户之间感兴趣商品的数量来提供一些依据。获取到两个用户相似的产品， 然后确定相似产品的类目就可以进行用户分析。类似的应用场景还有， 社交场景下共同关注好友， 相似兴趣 tag 等场景的支持。

![redis-set](../images/redis-set.png)

