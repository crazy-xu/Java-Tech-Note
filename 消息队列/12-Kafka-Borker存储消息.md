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
>
> **副本的编号是从1开始的**

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

#### 删除Topic

```shell
delete.topic.enable=true 开启永久删除开关，否则只是标记删除
```

```shell
./kafka-topics.sh --delete --zookeeper localhost:2181 --topic top1rep3part3

#### 副本在Broker的分布

分配策略是有AdminUtils.scale的assignReplicasToBrokers函数决定的

规则如下：

1. firt of all，副本因子不能大于Broker的个数。
2. 第一个分区（编号为0的分区）的第一个副本放置位置是随机从BrokerList选择的（Broker2的副本）。
3. 其他分区的第一个副本放置位置相对于第0个分区依次往后移。
4. 每个分区剩余的副本相对于第一个副本放置位置其实是有nextReplicaShift决定的，而这个数也是随机产生的。

### Segment

为了防止log不断追加导致文件过大，检索消息效率变低，一个partition又被划分成多个segment来组织数据（MySql也有segment的逻辑概念，叶子节点就是数据段，非叶子阶段就是索引段）。

在磁盘上，每个segment由一个log文件和两个index文件组成，成套出现的。

#### .log日志文件（消息数据）

log文件切割方式

1. 根据日志文件大小

   > 当一个segment写满之后，会创建一个新的segment，用最新的offset作为名称。默认大小是：log.segment.bytes=1G

2. 根据消息的最大时间戳，和当前系统时间戳的差值

   > log.roll.hours=168，默认一周
   >
   > 如果服务器上次写入消息是一周之前，旧的segment就不写了，会创建一个新的segment。
   >
   > 还可以从更加精细的时间单位进行控制，如果配置了毫秒级别的日志切分，会优先使用这个单位，否则就用小时的。log.roll.ms。

3. offset索引文件或者timeStamp索引文件到达一定大小

   > log.index.size.max.bytes，索引文件默认是10485760字节（10M）。如果要减少日志文件的切分，可以把这个值调大一些。索引文件写满后，数据文件也要跟着切分，是成套的。

   

#### .index 偏移量（offset）索引文件

#### .timeindex 时间戳（timestamp）索引文件

### 索引

偏移量索引文件记录的是offset和消息物理地址（在log文件中的位置）的映射关系。

时间戳索引文件记录的是时间戳和offset的关系。

#### 偏移量索引

查看索引日志文件内容：

```shell
./kafka-dump-log.sh --files /tmp/kafka-logs/mytopic-0/*.index | head -n 10
```

Kafka并不是每一条消息都会建立索引，使用稀疏索引（sparse index）

通过**log.index.interval.bytes=4096**来控制消息的大小，默认是4KB。只要写入的消息超过了4kb，偏移量索引文件.index和时间戳索引文件.timeindex就会增加一条索引记录。

这个值设置越小，索引就越密集。值设置越大，索引就越稀疏。相对来说，越稠密的索引检索数据更快，但是会消耗更多的存储空间。

Kafka索引的时间复杂度为O(log2n)+O(m)，n是索引文件里索引的个数，m为稀疏程度。



#### 时间戳索引





消费者与偏移量之间的关系 /tmp/kafka-logs/__consumer_offsets-23


读写在一个副本上，不用担心主从同步延迟的问题。


稀疏索引








消息清理，， 删除/压缩，默认是删除

定期清理，，5分钟，，log.retention.check.interval.ms=300000

日志删除的阈值
log.retention.*
log.retention.hours



谁来主持选举，，谁可以参加选举，，主从如何同步



