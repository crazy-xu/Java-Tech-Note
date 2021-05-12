## Redis 特点：
> 1、Redis 支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。<br >
> 2、Redis 不仅仅支持简单的 key-value 类型的数据， 同时还提供 list， set， zset， hash等数据结构的存储。<br >
> 3、Redis 支持数据的备份，即 master-slave 模式的数据备份。<br >

## Redis 优势：
1、性能极高 – Redis 能读的速度是 110000 次/s,写的速度是 81000 次/s 。<br >
2、丰富的数据类型 – Redis 支持二进制案例的 Strings, Lists, Hashes, Sets 及OrderedSets 数据类型操作。<br >
3、原子 – Redis 的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过 MULTI 和 EXEC 指令包起来。<br >
4、丰富的特性 – Redis 还支持 publish/subscribe, 通知, key 过期等等特性。

## 数据类型：
1、String（字符创），Hash（哈希），List（列表），Set（集合）及Z