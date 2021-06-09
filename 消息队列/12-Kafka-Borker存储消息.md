



// 存在多个副本，都成功

// 部署集群，奇数个，不会举手数一样。

leader 蛇形走位，按顺序选举


全部副本落盘成功，则响应成功。
要求所有副本同步成功。

ISR in-sync-replica set  动态，正常节点集合

replica.lag.time.max.ms=30s



acks=0 不等待ack
acks=1 leader 落盘返回ack
acks=-1 或者all, leader和全部follower 落盘成功，返回ack。需要预防消息重复。



消费者与偏移量之间的关系 /tmp/kafka-logs/__consumer_offsets-23


读写在一个副本上，不用担心主从同步延迟的问题。


稀疏索引








消息清理，， 删除/压缩，默认是删除

定期清理，，5分钟，，log.retention.check.interval.ms=300000

日志删除的阈值
log.retention.*
log.retention.hours



谁来主持选举，，谁可以参加选举，，主从如何同步



