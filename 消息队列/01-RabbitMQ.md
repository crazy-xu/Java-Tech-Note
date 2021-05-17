## MQ主要特点：
1、是一个独立运行的服务，生产者发送消息，消费者接收消费，需要先跟服务器建立连接。<br >
2、采用队列作为数据结构，有先进先出的特点。<br >
3、具有发布/订阅（publish/subscribe）的模型，消费者可以获取自己需要的消息。<br >
MQ 用来帮我们存储和转发消息。

## 为什么使用MQ：

#### 实现异步通信：
当一个异步过程调用发出后，调用者不会马上得到结果。而是在调用发出后，被调用着通过状态、通知来通知调用者，或者通过回调函数处理这个调用。
#### 实现系统解耦：
系统复杂，多个模块解耦，订单、库存、支付、通知。
#### 实现流量削峰：
MQ是Queue，有队列的特性，先进先出（FIFO）
#### 实现广播通信：
一对多通信，已订单系统退货为例，新增了结果通知系统，只需要增加队列监听即可，不需要生产者任何代码修改。

> 使用MQ会增加运维成本，系统的可用性降低，系统复杂性提高。根据实际情况来分析如何使用MQ。



## RabbitMQ架构

基于Erlang语言开发，Erlang是为电话交换机编写的语言，适合分布式和高并发。
RabbitMQ实现了AMQP协议（**重点了解AMQP协议**）

#### 1、Broker

默认端口5672，中介/经纪人

#### 2、Connection

生产着发送消息，还是消费者接收消息，都必须跟Broker之间建立一个连接，这个连接是TCP长连接。

#### 3、Channel

AMQP里引入了Channel的概念，是一个虚拟连接，可以理解成消息信道，这样就可以在保持的TCP长连接里面去创建和释放Channel，大大减少了资源消耗。不同的Channel是相互隔离的，每个Channel有自己的编号。

Channel是RabbitMQ原生API里面的最重要的编程接口，定义交换机、队列、绑定关系、发送消息、消费消息，调用的都是Channel接口的方法。

#### 4、Queue

队列是生产者和消费者的纽带，生产者发送消息到队列，在队列中存储，消费者从队列里消费消息。

RabbitMQ是用数据库来存储消息的，用Erlang开发，名字叫Mnesia。存储目录RabbitMQ/db/...

#### 5、Consumer

消费者消费消息有两种模式：Pull，Push

##### 5.1、Pull--主动拉取

对应方法是basicGet，消息存放在服务端，只有消费者主动获取才能拿到消息，如果每隔一段时间获取一次，消息的实时性就会降低。但是好处是可以根据自己的能力来决定消费消息的频率。

##### 5.2、Push--被动接收

对应方法是basicConsume，只要生产者发消息到服务器，就马上推送给消费者，消息保存在客户端，实时性较高，如果消费者消费不及时可能会造成消息积压。



> RabbitMQ中Pull和Push都有实现。Kafka和RocketMQ只有pull



一个消费者可以监听多个队列，一个队列也可以被多个消费者监听。

在生产环境中，建议一个消费者只处理一个队列消息。如果需要提升处理消息的能力，可以增加多个消费者，这时，消息就会在多个消费者之间轮询。

#### 6、Exchange

路由消息组件，将消息发送到Exchange，由它来分发消息，Exchange不会存储消息，只会根据规则分发消息。

Exchange需要和这些接收消息的队列建立绑定关系，并且为每个队列指定一个特殊标识。

Exchange和队列是多对多的绑定关系，一个交换机的消息有一个路由给多个队列，一个队列也可以接收来个多个交换机的消息。

绑定关系建立好之后，生产者发送消息到Exchange，也会携带一个特殊的标识。当这个标识跟绑定的标识匹配的时候，消息就会发给一个或多个符合规则的队列。

#### 7、Vhost

虚拟主机概念，提高硬件资源的利用率，实现资源的隔离和权限的控制。类似于namespace和package，不同的VHOST中可以有同名的Exchange和Queue。

为不同的业务创建不同的VHOST，然后在为他们创建专属用户，给用户分配对应的VHOST权限。

会有自带的默认VHOST，名字是“/”。



### 路由方式

http://tryrabbitmq.com/

#### 1、Direct直连

一个队列与直连类型的交换机绑定，需指定一个**明确的绑定键（binding key）**。

生产者发送消息时，会携带一个路由键（routing key）。当消息的路由键与某个队列的绑定键完全匹配时，这条消息才会从交换机路由到这个队列上。多个队列也可以也使用相同的绑定键。

```java
channel.basicPublish("DIRECT_EXCHANGE", "Spring", "Direct直连，明确绑定键。"); // binding key = Spring。会发送到Spring的Queue。
```

#### 2、Topic主题

一个队列与主题类型的交换机绑定时，可以在绑定键中使用通配符。支持两个通配符。

“#”：代表0个或者多个单词。

“*”：代表只能有一个单词。

单词指的是用英文的点“.”隔开的字符。例如a.bc.def是3个单词。

> 例如：
>
> 1、binding key = java.# 路由键以java开头的消息路由，后面可以有单词，也可以没有。
>
> 2、binding key = #.java 路由键以java结尾的消息路由，前面可以有单词，也可以没有单词。
>
> 3、binding key = xu.* 路由键以xu开头，并且后边只有一个单词的消息路由。
>
> 4、binding key = *.xu 路由键以xu结尾，并且前面只有一个单词的消息路由。

```java
channel.basicPublish("TOPIC_EXCHANGE", "java.xu.crazy", "Topic主题，1收到");
channel.basicPublish("TOPIC_EXCHANGE", "xu.java", "Topic主题，2、3收到。");
channel.basicPublish("TOPIC_EXCHANGE", "crazy.xu", "Topic主题，4收到。");
```

可以用于消息过滤场景，多个二级场景下，就可以使用不同的绑定键接收消息。

#### 3、Fanout广播

广播类型的交换机与队列绑定时，不需要指定绑定键。生产者发送消息到广播类型的交换机上，不需要携带路由键。消息到达交换机时，所有与之绑定了的队列，都会收到相同的消息副本。

```java
channel.basicPublish("FANOUT_EXCHANGE", "", "Fanout广播，都收到。");
```

可以用于通用的业务消息，广播一下，然后大家都收到了。如果有新增队列，自己建队列即可，不需要生产者进行操作。

#### 4、Headers

emmmmm....不常用。

### RabbitMQ安装

> * RabbitMQ依赖于Erlang，需要先安装Erlang。
>
> * Erlang和RabbitMQ版本有对应关系。https://www.rabbitmq.com/which-erlang.html



##### 1、安装Erlang 21.3

安装依赖

```
// 注意：因为每个人的操作系统环境是不一样的，缺少的依赖不同，根据提示安装。
yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget
```

下载安装包

> ```
> wget http://erlang.org/download/otp_src_22.3.tar.gz
> tar -xvf otp_src_22.3.tar.gz
> cd otp_src_22.3
> ./configure --prefix=/usr/local/erlang
> ```

> configure的过程如果有err，要解决依赖的问题。
> 如果有APPLICATIONS INFORMATION，DOCUMENTATION INFORMATION，没有影响。

> ```
> make && make install
> 
> // 如果提示缺少socat
> yum install -y socat
> ```

##### 2、配置Erlang环境变量

> ```
> vim /etc/profile
> ```

加入一行

> ```
> export PATH=$PATH:/usr/local/erlang/bin // 安装地址
> ```

编译生效

> ``` 
> source /etc/profile
> ```



##### 3、验证Erlang是否安装成功

输入`erl`，会出现版本信息，即安装成功

##### 4、安装RabbitMQ 3.8.15

##### 5、配置RabbitMQ环境变量

##### 6、启动RabbitMQ

##### 7、添加用户

##### 8、启用管理插件

##### 9、访问http://IP:15672











、STOMP、MQTT、HTTP、WebSockets协议。

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

