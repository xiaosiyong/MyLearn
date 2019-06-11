### NSQ 启动：

1、nsqlookupd

2、nsqd --lookupd-tcp-address=127.0.0.1:4160

3、nsqadmin --lookupd-http-address=127.0.0.1:4161

### Kafka：

1、启动kafka前先启动zookeeper   进入libexe

bin/zookeeper-server-start.sh config/zookeeper.properties

2、启动kafka bin/kafka-server-start.sh config/server.propertiesls



