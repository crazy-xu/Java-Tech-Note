## 生产者发送消息流程

生产端由两个线程协调运行，这两条线程分别为main线程和sender线程（发送线程）。

在创建KafkaProducer的时候，会创建一个Sender对象，并启动一个IO线程。

```java
Producer<String,String> producer = new KafkaProducer<String,String>(pros);
```

```java
this.sender = this.newSender(logContext, kafkaClient, this.metadata);
String ioThreadName = "kafka-producer-network-thread | " + this.clientId;
this.ioThread = new KafkaThread(ioThreadName, this.sender, true);
this.ioThread.start();
```

接下来执行send方法

```java
producer.send(new ProducerRecord<String,String>("mytopic",Integer.toString(i),Integer.toString(i)));
```

### 拦截器

```java
ProducerRecord<K, V> interceptedRecord = this.interceptors.onSend(record);
```

拦截器的作用是实现消息的定制化。interceptors是带s说明可以指定多个拦截器。

添加指定拦截器：

```java
// 添加拦截器
List<String> interceptors = new ArrayList<>();
interceptors.add("com.kafka.interceptor.testInterceptor");
props.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, interceptors);
```

自定义拦截器时，需要implements ProducerInterceptor<String, String>接口，分为以下四个方法，分别是发送消息、收到服务端的ACK、生产者关闭消息，用键值对配置时触发。

```java
// 发送消息
ProducerRecord<K, V> onSend(ProducerRecord<K, V> var1);
// 收到服务端ACK
void onAcknowledgement(RecordMetadata var1, Exception var2);
// 生产者关闭
void close();
// 用键值对配置
void configure(Map<String, ?> var1);
```

### 序列化

```java
byte[] serializedKey;
try {
    serializedKey = this.keySerializer.serialize(record.topic(), record.headers(), record.key());
} catch (ClassCastException var21) {
    throw new SerializationException("Can't convert key of class " + record.key().getClass().getName() + " to class " + this.producerConfig.getClass("key.serializer").getName() + " specified in key.serializer", var21);
}

byte[] serializedValue;
try {
    serializedValue = this.valueSerializer.serialize(record.topic(), record.headers(), record.value());
} catch (ClassCastException var20) {
    throw new SerializationException("Can't convert value of class " + record.value().getClass().getName() + " to class " + this.producerConfig.getClass("value.serializer").getName() + " specified in value.serializer", var20);
}
```

利用指定的序列化工具对key和value进行序列化操作。

在Serializer.java接口中，kafka针对不同的数据类型完善了相应的序列化工具。可以使用其它序列化工具，如：JSON、Thrift等等，或者自定义序列化器，实现Serializer接口即可。

### 路由指定（分区器）

```java
int partition = this.partition(record, serializedKey, serializedValue, cluster);
```

一条消息发送到那个partition，有4种情况：

1. 指定了partition；
2. 没有指定partition，自定义分区器；
3. 没有指定partition，没有自定义分区器，但是key不为空；
4. 没有指定partition，没有自定义分区器，但是key是空的。

* 指定partition

```java
// 创建producer
KafkaProducer<String, Integer> producer = new KafkaProducer<String, Integer>(props);
String topic = "qs4part2673";
// 根据topic获取分区数量partitionSize
int partitionSize = producer.partitionsFor(topic).size();
System.out.println("Partition size: " + partitionSize);
for (int i = 0; i < 10; i++) {
    // 根据Random随机选择分区
    int partition = new Random().nextInt(partitionSize);
    ProducerRecord<String, Integer> producerRecord = new ProducerRecord<String, Integer>(topic, partition, null, i);
    RecordMetadata metadata = producer.send(producerRecord).get();
    System.out.println("partition: " + metadata.partition() + ", offset: " + metadata.offset());
}
```

* 没有指定partition，自定义分区器

使用自定义的分区器算法选择分区。实现implements Partitioner接口，

```java
@Override
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
    // 根据业务规则自定义返回partition
    String k = (String) key;
    System.out.println(k);
    if (Integer.parseInt(k) % 2 == 0) {
        return 0;
    } else {
        return 1;
    }
}
```

```java
// 如上代码为该SimplePartitioner选择分区器规则
props.put("partitioner.class", "com.kafka.partition.SimplePartitioner");
```

* 没有指定partition，没有自定义分区器，但是key不为空

使用默认分区器DefaultPartitioner，将用 murmur2 的方法对 key 进行 HASH 与numPartitions（Topic 的分区数）进行取余得到partition值。

```java
Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions
```

* 没有指定partition，没有自定义分区器，但是key是空的。

第一次调用时随机生成一个整数，将这个值与topic可用的partition总数取余得到partition值。

```java
this.stickyPartitionCache.partition(topic, cluster)
```

```java
package org.apache.kafka.clients.producer.internals;

import java.util.List;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;
import java.util.concurrent.ThreadLocalRandom;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.PartitionInfo;
import org.apache.kafka.common.utils.Utils;

public class StickyPartitionCache {
    private final ConcurrentMap<String, Integer> indexCache = new ConcurrentHashMap();

    public StickyPartitionCache() {
    }

    public int partition(String topic, Cluster cluster) {
        // 通过ConcurrentMap对 Topic -> Partition 的映射，如果该值不存在则调用 nextPartition 方法选择一个分区并缓存。
        Integer part = (Integer)this.indexCache.get(topic);
        // 如果为null，则通过nextPartition计算
        return part == null ? this.nextPartition(topic, cluster, -1) : part;
    }

    public int nextPartition(String topic, Cluster cluster, int prevPartition) {
        // 获取集群的分区集合
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        Integer oldPart = (Integer)this.indexCache.get(topic);
        Integer newPart = oldPart;
        if (oldPart != null && oldPart != prevPartition) {
            return (Integer)this.indexCache.get(topic);
        } else {
            // 获取集群的可用分区
            List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
            Integer random;
            if (availablePartitions.size() < 1) {
                // 随机数与分区数取余
                random = Utils.toPositive(ThreadLocalRandom.current().nextInt());
                newPart = random % partitions.size();
            } else if (availablePartitions.size() == 1) {
                // 只有一个，则返回该分区
                newPart = ((PartitionInfo)availablePartitions.get(0)).partition();
            } else {
                while(newPart == null || newPart.equals(oldPart)) {
                    // 获取随机整数
                    random = Utils.toPositive(ThreadLocalRandom.current().nextInt());
                    // 随机数与可用分区取余，返回partition值
                    newPart = ((PartitionInfo)availablePartitions.get(random % availablePartitions.size())).partition();
                }
            }

            if (oldPart == null) {
                // 说明首次访问不存在，赋值初始化
                this.indexCache.putIfAbsent(topic, newPart);
            } else {
                // 赋值
                this.indexCache.replace(topic, prevPartition, newPart);
            }

            return (Integer)this.indexCache.get(topic);
        }
    }
}

```

### 消息累加器

选择分区后并没有直接发送消息，而是把消息放入了消息累加器。

```java
RecordAppendResult result = this.accumulator.append(tp, timestamp, serializedKey, serializedValue, headers, interceptCallback, remainingWaitMs, true, nowMs);
```

一个partition对应一个batch，当batch满了之后，会唤醒之前IO线程，并发送消息。

通过源代码可看到**ConcurrentMap<TopicPartition, Deque<ProducerBatch>>** batches是一个ConcurrentMap。

```java
if (result.batchIsFull || result.newBatchCreated) {
    this.log.trace("Waking up the sender since topic {} partition {} is either full or getting a new batch", record.topic(), partition);
    this.sender.wakeup();
}
```



**Kafka可以在拦截器里自定义消息处理逻辑，也可以选择指定的序列化工具，还以自由选择分区。**



## 数据可靠性保证ACK

### 服务端响应策略

生产者发送消息到服务端后，只有服务端确认接收成功之后，生产者才发送下一轮的消息，否则重新发送数据。

在Kafka中，需要所有的正常follower全部完成同步，才发送ack给生产者，延迟相对来说高一些的，但是可以保证所有节点数据都是完整的，可靠性高。



### ISR（in-sync replica set）

从概率上讲，不能保证所有的follower都是正常运行，如果某个follower异常，无法同步数据，leader节点就存在等待情况，无法给生产者发送ack。

Kafka把正常和leader保持同步的Replica维护到走一个动态set里，名字**in-sync replica set（ISR）**，只要ISR里的follower同步完成数据之后，就给生产者发送ack。

* replica.lag.time.max.ms 决定是否在ISR中

  >  如果一个follower在这个时间内没有发送fetch请求或消费leader日志到结束的offset，leader将从ISR中移除这个follower，并认为这个follower已经挂了。
  >
  > https://kafka.apachecn.org/documentation.html

### ACK应答机制

Kafka提供了三种可靠性级别应答，可根据对可靠性和延迟的要求进行选择。

#### acks=0

* 不等待ack

  生产者不等待Broker的ack，延迟最低，Broker一接收到还没有写入磁盘就已返回，当Broker故障时可能存在数据丢失。

#### acks=1（默认）

* leader 落盘返回ack

  生产者等待Broker的ack，partition的leader落盘成功后，返回ack。如果在follower同步成功之前leader鼓掌，将会丢失数据。

#### acks=-1 或者all

* leader和全部follower 落盘成功，返回ack。

  延迟较高，可能存在follower同步完成后，Broker发送ack之前，leader故障，没有给生产者发送ack，会造成数据重复。这种情况下需要将reties设置为0（不重发），才不会重复。

以上三种机制，性能依次递减，数据健壮性依次递增。可以根据不同的业务场景使用不同的参数。





