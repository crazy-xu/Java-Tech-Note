## Broker存储消息

* 部署集群，奇数个，不会举手数一样，没有分歧
* leader 蛇形走位，按顺序选举



消息文件存储默认路径/tmp/kafka-logs

### partition 分区

实现横向扩展，可以把一个topic中的数据分隔成多个partition，一个partition的消息是有序的，顺序写入，但是全局不一定有序。

在服务器上，每个partition都有一个物理目录，topic名字后边的数字标号即代表分区。

### Replica 副本

提高分区的可靠性，kafka设计了副本机制。

在创建topic时，通过指定replication-factor确定topic的副本数量。

> 注：副本数必须小于等于节点数，而不能大于Broker的数量，否则会报错。就可以保证不会有一个分区的副本分布在同一个节点上。

```shell
./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 3 --topic top1rep3part3

# replication-factor 副本数量
# partitions 分区数量
```

副本分为两种角色，leader对外提供读写服务，follower只从leader异步拉取数据。

读写都在leader节点，就不存在读写分离带来的数据不一致问题。

#### 查看leader节点

```shell
./kafka-topics.sh --topic top1rep3part3 --describe --zookeeper localhost:2181
```

这个top1rep3part3有3个分区3个副本。











消费者与偏移量之间的关系 /tmp/kafka-logs/__consumer_offsets-23


读写在一个副本上，不用担心主从同步延迟的问题。


稀疏索引








消息清理，， 删除/压缩，默认是删除

定期清理，，5分钟，，log.retention.check.interval.ms=300000

日志删除的阈值
log.retention.*
log.retention.hours



谁来主持选举，，谁可以参加选举，，主从如何同步



