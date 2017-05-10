非docker下 kafka集群搭建，部署三台服务器，分别是k1,k2,k3
1、启动zookeeper集群
```
root@k1:/usr/local/kafka# cat config/zookeeper.properties 
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
initLimit=5
syncLimit=2
server.1=k1:2888:3888
server.2=k2:2888:3888
server.3=k3:2888:3888
```
各节点配置文件统一即可，注意，要保证hosts文件能够解析到k1,k2,k3；各节点上都必须有/tmp/zookeeper/myid文件，myid属性不能重复
```
root@k1:mkdir /tmp/zookeeper && echo 1 > /tmp/zookeeper/myid 
root@k2:mkdir /tmp/zookeeper && echo 2 > /tmp/zookeeper/myid 
root@k3:mkdir /tmp/zookeeper && echo 3 > /tmp/zookeeper/myid 
```
各节点可以同时启动zookeeper
```
root@k1:/usr/local/kafka# bin/zookeeper-server-start.sh  config/zookeeper.properties 
root@k2:/usr/local/kafka# bin/zookeeper-server-start.sh  config/zookeeper.properties 
root@k3:/usr/local/kafka# bin/zookeeper-server-start.sh  config/zookeeper.properties 
```

查看zookeeper节点信息
```
root@k1:/usr/local/kafka# bin/zookeeper-shell.sh k1
Connecting to k1
Welcome to ZooKeeper!
JLine support is disabled

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
ls /
[node, cluster, controller_epoch, controller, brokers, zookeeper, test, admin, isr_change_notification, consumers, config]
```

2、启动kafka集群
```
root@k1:/usr/local/kafka# vim config/server.properties
broker.id=1
zookeeper.connect=k1:2181,k2:2181,k3:2181
```
注意，各节点的broker.id不能相同，zookeeper.connect可以根据实际情况自行添加
```
三个节点同时启动服务
root@k1:/usr/local/kafka# bin/kafka-server-start.sh config/server.properties
```

3、创建及查看topic
```
root@k1:/usr/local/kafka# bin/kafktopics.sh --create --zookeeper k1:2181 --replication-factor 3 --partitions 1 --topic mytest
root@k1:/usr/local/kafka# bin/kafktopics.sh --list --zookeeper k1:2181 --topic mytest

```

4、发布和消费topic
在任意节点上启动生产者，并输入任意字符
```
root@k3:/usr/local/kafka# bin/kafka-console-proder.sh --broker-list localhost:9092 --topic test
1
2
3
```
在任意节点上启动消费者，可以看到收到生产者发生的字符
```
root@k3:/usr/local/kafka# bin/kafka-console-consumer.sh --bootstrap-server k3:9092 --topic test --from-beginning
1
2
3
```
