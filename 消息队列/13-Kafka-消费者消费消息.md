## 消费者消费消息

消息消费到哪里，保存在一个特殊的topic中，__consumer_offset_01*

 

消息分配逻辑：

* Range范围分配
* RoundRobin轮询分配
* Sticky粘滞分配，随机不固定，尽量跟上一次分配保持相同。



#### Kafka为什么这么快？

* 顺序IO/磁盘IO

​		再磁盘空间写入的数据是顺序的

* 索引
*   