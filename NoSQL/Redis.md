#### Lua管理脚本的四个命令：

Script Flush  清除服务器中和Lua脚本有关的信息，释放并重建lua_scripts字典，关闭现有的Lua环境并重新创建一个新的Lua环境

Script Exists

Script Load

Script Kill



#### Redis 备份两种方式

AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集

RDB 在指定的时间间隔内生成数据集的时间点快照

#### Redis 启动

定位到src   make

然后  ...../redis-server   

redis-cli -h 127.0.0.1 -p 6379 shutdown 关闭Redis

redis-cli   slaveof 10.28.51.53 6379  设置slave

Redis-cli  info  显示Redis信息      



**源码安装redis步骤：**

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

