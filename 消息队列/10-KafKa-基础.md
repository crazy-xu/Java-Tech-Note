

    Kafka是由LinkedIn公司采用Scala语言开发的一个多分区、多副本且基于ZooKeeper协调的分布式消息系统。
    定位是分布式流式处理平台，以高吞吐、可持久化、可水平扩展、支持流数据处理等多种特性而被广泛使用。

1. 消息系统：具备系统解耦、冗余存储、流量削峰、缓冲、异步通信、扩展性、可恢复性等功能。还有大多数消息系统难以实现的消息顺序性保障及回溯消费的功能。

2. 存储系统：可以把消息持久化到磁盘，降低数据丢失的风险。可以作为长期的数据存储系统来使用。只需要把对应的数据保留策略设置为“永久”或启动主题的日志压缩功能。

3. 流式处理平台：不仅为每个流式处理框架提供了可靠的数据来源，还提供了一个完整的流式处理类库，比如窗口、连接、变换和聚合等各类操作。

### 1.1 基本概念

    Kafka体系架构是由若干Producer、若干Borker、若干Consumer，以及一个Zookeeper集群。
    
    (1)Producer：生产者，发送消息的一方。生产者负责创建消息，然后将消息发送到Kafka中。
    (2)Consumer：消费者，接收消息的一方。连接到Kafka上并接收消息，从而进行相应的业务处理。
    (3)Broker：服务代理节点。Broker可以看作一个独立的Kafka服务节点或者Kafka服务实例。一个或者多个Broker组成一个Kafka集群。
    
    在Kafka中还有两个概念--主题（Topic）与分区（Partition）。Kafka中的消息以主题为单位进行归类，生产者负责将消息发送到特定的主题（Topic）。
    而消费者负责订阅主题并进行消费。

### 1.2 Kafka与ZK关系

利用ZK的有序节点、临时节点和监听控制，ZK帮Kafka提供了配置中心（管理Broker、Topic、Partition、Consumer的信息，包括元数据的变动）、负载均衡、命名服务、分布式通知、集群管理、选举、分布式锁。

### 1.3 Kafka架构

#### 1.3.1 Broker

默认端口9092，生产者和消费者都需要跟这个Broker建立一个连接，才可以实现消息的收发。存储和转发消息。

#### 1.3.2 消息

可以叫做记录（Record），Record在客户端代码中可以是一个KV键值对。

生产者对应的封装类是ProducerRecord，消费者对应的封装类是ConsumerRecord。消息在传输过程中需要序列化，需要在代码里指定序列化工具。

#### 1.3.3 生产者

为了提升消息发送速率，生产者不是逐条发送消息到Broker，而是批量发送。

batch.size：决定多少条发送一次，默认是16k；

linger.ms：批量发送的等待时间；

buffer.memory：客户端缓冲区大小，默认32M，满了也会触发消息发送。

#### 1.3.4 消费者

用Pull模式接收数据，消费者通过max.poll.records控制一次获取多少条消息，默认是500。

#### 1.3.5 Topic

在Kafka里，队列名叫Topic，是一个逻辑概念，理解为一组消息的集合。

生产者和Topic以及Topic和消费者的关系是多对多。

生产者发送消息时，如果Topic不存在，会自动创建，有一个参数控制**auto.create.topics.enable**，默认是true。



#### 1.3.6 Partition

分区概念，一个Topic可以划分成多个分区，在创建topic的时候指定，每个topic至少有一个分区。类似于分库分表，实现横向扩展和负载的目的。

每个partition都有一个物理目录，在/tmp/kafka-logs/中。partition里的消息被读取后不会被删除，同一批消息在一个partition里顺序、追加写入的。

分区数量怎么选择，可以通过性能测试的脚本验证。

#### 1.3.7 Partition副本Replica机制

每个partition可以有若干个副本（Replica）,副本必须在不同的Broker上面。副本数量小于等于Broker数量。

副本有leader和follower的概念，leader在哪台机器，选举出来。

生产者发消息和消费者读消息都是通过leader。follower的数据是从leader同步过来的。

#### 1.3.8 Segment

将partition做一个切分，切分出来的单位叫做段（segment），kafka的存储文件是划分成段来存储的。每个segment都有一个数据文件和两个索引文件，这个三个文件是成套出现的。一个segment默认大小是1073731824 bytes(1G)，由log.segment.bytes控制。

默认存储路径：/tmp/kafka-logs/

每个segment都至少有一个数据文件和两个索引文件，这三个文件是成套出现的。

#### 1.3.9 Consumer Group

如果生产者生产消息过快，会造成消息在Broker的堆积，影响性能。引入Consumer Group消费组概念，在代码中通过group id 来配置。消费通过一个topic的消费者不一定是同一个组，只有group id 相同的消费者才是同一个消费者组。

> 如果消费者比partition少，一个消费者可以消费多个partition。
>
> 如果消费者比partition多，会有消费者存在消费不上。空闲状态。
>
> 由程序分配，不可变更。

#### 1.3.10 Consumer Offset

消息是有序的，会对每个消息进行编号，用来标识一条唯一的消息。这个编号叫做offset-偏移量。

offset记录着下一条的发送给consumer的消息的序号。

#### 1.3.11 消息幂等性

```java
enable.idempotence=true
```

Producer 自动升级为幂等性Producer，Kafka会自动去重。

实现机制：

* PID(Producer ID)：幂等性的生产者每个客户端都有一个唯一的编号。
* sequence number：幂等性的生产者发送的每条消息都会带相应的sequence number，server端根据这个值来判断数据是否重复。
  * 只能保证单分区上的幂等性，即一个幂等性Producer能够保证某个肢体的一个分区上不出现重复消息。
  * 只能实现单会话上的幂等性，这里的会话指的是producer进程的一次运行，当重启producer进程之后，幂等性不保证。



### 1.4 生产者事务

Kafka的事务属于分布式事务。两阶段提交（2PC），如果大家都可以commit，那么就commit，否则abort。

既然是2PC，就要有一个协调者角色，叫做Transaction Coordinator。通过事务日志来记录事务的状态，kafka使用特殊的topic_transaction_state来记录事务状态。

如果生产者挂了，事务要在重启后可以继续处理，之前未处理完的事务，或者在其他机器上处理，必须要有一个唯一的ID，这个就是transaction.id。在配置enable.idempotence=true（事务实现的前提是幂等性）。事务ID相同的生产者，可以接着处理原来的事务。



> ```java
> producer.initTransactions(); // 初始化事务
> producer.beginTransaction(); // 开启事务
> producer.commitTransaction(); // 提交事务
> producer.abortTransaction(); // 中止事务
> producer.sendOffsetsToTransaction(); // 
> ```



```java
public static void main(String[] args) {
    Properties props=new Properties();
    props.put("bootstrap.servers","192.168.44.160:9092");
    props.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
    props.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
    // 0 发出去就确认 | 1 leader 落盘就确认| all或-1 所有Follower同步完才确认
    props.put("acks","all");
    // 异常自动重试次数
    props.put("retries",3);
    // 多少条数据发送一次，默认16K
    props.put("batch.size",16384);
    // 批量发送的等待时间
    props.put("linger.ms",5);
    // 客户端缓冲区大小，默认32M，满了也会触发消息发送
    props.put("buffer.memory",33554432);
    // 获取元数据时生产者的阻塞时间，超时后抛出异常
    props.put("max.block.ms",3000);

    props.put("enable.idempotence",true);
    // 事务ID，唯一
    props.put("transactional.id", UUID.randomUUID().toString());

    Producer<String,String> producer = new KafkaProducer<String,String>(props);

    // 初始化事务
    producer.initTransactions();

    try {
        producer.beginTransaction();
        producer.send(new ProducerRecord<String,String>("transaction-test","1","1"));
        producer.send(new ProducerRecord<String,String>("transaction-test","2","2"));
        // Integer i = 1/0;
        producer.send(new ProducerRecord<String,String>("transaction-test","3","3"));
        // 提交事务
        producer.commitTransaction();
    } catch (KafkaException e) {
        // 中止事务
        producer.abortTransaction();
    }
    producer.close();
}
```



















































