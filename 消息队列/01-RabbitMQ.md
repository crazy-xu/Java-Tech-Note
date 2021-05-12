世界上第一个MQ，发布订阅--Message Queue

MQ主要特点：
1、是一个独立运行的服务，生产者发送消息，消费者接收消费，需要先跟服务器建立连接。
2、采用队列作为数据结构，有先进先出的特点。
3、具有发布/订阅（publish/subscribe）的模型，消费者可以获取自己需要的消息。

异步：
系统数据量大，异步查询，
解耦：
系统复杂，多个模块解耦，订单、库存、支付、通知。
削峰：

广播通信：
一对多。

开发语言Erlang
支持AMQP、STOMP、MQTT、HTTP、WebSockets协议。

默认端口：5672
Broker 主机
跟Broker建立连接，TCP长链接。

Channel 虚拟连接，
使用数据库存储消息，Mnesia
Queue 是生产者和消费者的纽带。


Exchange交换机，与Queue队列建立绑定关系。


Vhost 主机


直连Direct类型交换机

主体Topic类型交换机
#有单词和没单词都可以


延迟消费，死信队列

消费模型
pull-消费者主动去broker拉取。basicGet
push-只要有消息到达队列就发送给消费者，消费者易堆积。basicConsume.


kafka 和RocketMQ 只有pull


