
## 可靠性投递
我们在使用RabbitMQ收发消息的流程：
1. 消息从生产者发送到Broker。
2. 消息从Exchange路由到Queue。Exchange绑定Queue。
3. 消息在Queue中存储。
4. 消费者订阅Queue并消费消息。

### 1、消息发送到RabbitMQ服务器

在RabbitMQ中提交了两种服务端确认机制，生产者发送消息给RabbitMQ的服务端的时候，服务端会通过某种方式返回一个应答，生产者收到这个应答就知道消息发送成功。

##### Transaction事务模式

在创建channel的时候，可以把信道设置成事务模式，然后发送消息给RabbitMQ，如果channle.txCommit();方法调用成功，就说明事务提交成功。

```java
public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri(ResourceUtil.getKey("rabbitmq.uri"));
        // 建立连接
        Connection conn = factory.newConnection();
        // 创建消息通道
        Channel channel = conn.createChannel();
        String msg = "Hello world, Rabbit MQ";
        // 声明队列（默认交换机AMQP default，Direct）
        // String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        try {
            channel.txSelect();
            // 发送消息，发布了4条，但只确认了3条
            // String exchange, String routingKey, BasicProperties props, byte[] body
            channel.basicPublish("", QUEUE_NAME, null, (msg).getBytes());
            channel.txCommit();
            channel.basicPublish("", QUEUE_NAME, null, (msg).getBytes());
            int i =1/0;
            channel.txCommit();
            System.out.println("消息发送成功");
        } catch (Exception e) {
            channel.txRollback();
            System.out.println("消息已经回滚");
        }

        channel.close();
        conn.close();
    }
```

在事务模式里，只有收到了Commit指令后，才能提交成功。但是事务模式有一个缺点，它是阻塞的，一条消息没有发送完毕，不能发送下一条消息，它会榨干RabbitMQ的性能，不建议使用。

* 指令流程

  * 发送txSelect()，返回ok
  * 发送basicPublish()
  * 发送txCommit()，返回ok

  交互次数频繁，影响性能。

  

##### Confirm确认模式

###### 普通确认模式

生产者通过调用channel.confirmSelect()方法，将信道设置为Confirm模式，然后发送消息。一旦消息被发送到交换机之后，RabbitMQ就会发送一个确认（Basic.Ack）给生产者，也就是调用channel.waitForConfirms()返回true，这样生产者就知道消息被服务端接收了。这种模式是发送一条确认一条消息。

```java
String msg = "Hello world, Rabbit MQ ,Normal Confirm";
// 开启发送方确认模式
channel.confirmSelect();
channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
// 普通Confirm，发送一条，确认一条
if (channel.waitForConfirms()) {
	System.out.println("消息发送成功" );
}else{
	System.out.println("消息发送失败");
}
```

###### 批量确认模式

只要channel.waitForConfirmsOrDie();没有抛出异常，说明消息都被服务端接收了，比单条确认方式效率要高。但是要全部接收成功才会响应成功，假如最后一条失败了，前面的所有消息都要重发。

```java
 	try {
            channel.confirmSelect();
            for (int i = 0; i < 5; i++) {
                // 发送消息
                // String exchange, String routingKey, BasicProperties props, byte[] body
                channel.basicPublish("", QUEUE_NAME, null, (msg +"-"+ i).getBytes());
            }
            // 批量确认结果，ACK如果是Multiple=True，代表ACK里面的Delivery-Tag之前的消息都被确认了
            // 比如5条消息可能只收到1个ACK，也可能收到2个（抓包才看得到）
            // 直到所有信息都发布，只要有一个未被Broker确认就会IOException
            channel.waitForConfirmsOrDie();
            System.out.println("消息发送完毕，批量确认成功");
        } catch (Exception e) {
            // 发生异常，可能需要对所有消息进行重发
            e.printStackTrace();
        }
```

###### 异步确认模式

需要添加一个addConfirmListener，并且用一个SortedSet来维护一个批次中没有被确认的消息。

### 2、消息从交换机Exchange路由到队列Queue

有可能是routingkey错误，或者是队列Queue不存在（生产环境基本上不会出现这两种问题）

##### 消息回发

需要Mandatory(true)

```java
rabbitTemplate.setMandatory(true);
rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback(){
    public void returnedMessage(Message message,
                                int replyCode,
                                String replyText,
                                String exchange,
                                String routingKey){
        System.out.println("回发的消息：");
        System.out.println("replyCode: "+replyCode);
        System.out.println("replyText: "+replyText);
        System.out.println("exchange: "+exchange);
        System.out.println("routingKey: "+routingKey);
    }
});
```

##### 消息路由到备份交换机

在创建交换机的时候，从属性中指定备份交换机。

队列可以指定死信交换机，交换机可以指定备份交换机。

```java
// 在声明交换机的时候指定备份交换机
Map<String,Object> arguments = new HashMap<String,Object>();
arguments.put("alternate-exchange","ALTERNATE_EXCHANGE");
channel.exchangeDeclare("TEST_EXCHANGE","topic", false, false, false, arguments);
```

### 3、消息在队列Queue存储

##### 队列持久化

```java
// 声明队列
// String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
channel.queueDeclare(QUEUE_NAME, false, false, false, null);
```

* durable：没有持久化的队列，保存在内存中，服务重启后队列和消息都会消失。
* exclusive：排他性队列
  * 只对首次声明它的连接可见
  * 会在其连接断开的时候自动删除
* autoDelete：没有消费者连接的时候，自动删除

##### 交换机持久化

##### 消息持久化

##### 集群

### 4、消息投递到消费者

RabbitMQ提供了消费者的消息确认机制，消费者可以自动或者手动的发送ack给服务端。

没有收到ack的消息，消费者断开连接后，RabbitMQ会把这条消息发送给其他消费者。如果没有其他消费者，消费者重启后会重新消费这条消息，重复执行业务逻辑。

* 消费者应答有两种方式

  * 1、自动ack

    > 默认，消费者会在收到消息的时候，就自动发送ack，而不是在方法执行完毕的时候发送ack（并不关心你有没有正常消费消息）

  * 2、手动ack

    > 需要把自动ack设置成手动ack，把autoAck设置成false。就需要RabbitMQ等待消费者显式的回复ack之后才会从队列中移去消息。



```java
spring.rabbitmq.listener.direct.acknowledge-mode=
spring.rabbitmq.listener.simple.acknowledge-mode=
```

* AcknowledgeMode.NONE：不确认
* AcknowledgeMode.MANUAL：手动确认
* AcknowledgeMode.AUTO：自动确认--如果未抛出异常，则发送ack。如果抛出异常，并且不是AmqpRejectAndDontRequeueException()则发送nack，并且重新入队列。如果抛出异常是AmqpRejectAndDontRequeueException()，则发送nack，不会重新入队列。

```java
// 拒绝消息 requeue：是否重新入队列，true：是；false：直接丢弃，相当于告诉队列可以直接删除掉
channel.basicReject(envelope.getDeliveryTag(), false);
// 批量拒绝
// requeue：是否重新入队列
channel.basicNack(envelope.getDeliveryTag(), true, false);
// 手工应答
// 如果不应答，队列中的消息会一直存在，重新连接的时候会重复消费
channel.basicAck(envelope.getDeliveryTag(), true);
```

> 如果requeue参数设置为true，可以把这条消息重新存入队列，以便发给下一个消费者。（如果只有一个消费者的时候，这种方式可以会出现无限循环重复消费的情况）

### 5、生产者如何知道是否消费成功

### 6、消费者回调

### 7、补偿机制

### 8、消息幂等性

### 9、最终一致

### 10、消息顺序性



## RabbitMQ 集群

集群有两种节点类型，一种是磁盘节点（disc Node）,一种是内存节点（RAM Node）.

* 磁盘节点：将元数据（包括队列名字属性、交换机类型名字属性、绑定、vhost）放在磁盘中。未指定类型的情况下，默认为磁盘节点。

  > 集群中至少需要一个磁盘节点用来持久化元数据，否则全部内存节点崩溃时，就无法同步元数据。

* 内存节点：将元数据放在内存中。

  > 内存节点会将磁盘节点的地址存放在磁盘，如果是持久化的消息，会同时存放在内存和磁盘。













































