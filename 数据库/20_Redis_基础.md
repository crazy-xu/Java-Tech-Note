## 1.1 Redis介绍
全称是REmote DIctionary Service，直接翻译过来是远程字典服务。
是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。
支持多种类型的数据结构，如字符串（strings），散列（hashes），列表（lists），集合（sets），有序集合（sorted sets）与范围查询，bitmaps，hyperloglogs和地理空间（geospatial）索引半径查询。Redis内置了复制（replication），LUA脚本（Lua scripting），LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的磁盘持久化（persistence），并通过Redis哨兵（Sentinel）和自动分区（Cluster）提供高可用性（high availability）。

## 1.2 SQL与NoSQL
关系型数据库SQL，在大部分情况下，我们都会首先考虑用它来存储数据，比如使用MySQL、Oracle、SQLServer等等。
    
关系型数据库的特点:
    
* 1、以表格的形式，基于行存储数据，是个二维模式。
* 2、存储的是结构化数据，数据存储有固定的模式（schema），数据需要配合表结构类型
* 3、表与表之间存在关联（Relationship）。
* 4、大部分关系型数据库都支持SQL（结构化查询语言）操作，支持复杂的联表查询。
* 5、通过支持事务（ACID酸）来提供严格或者实时的数据一致性。

但是也有一些使用限制，比如：
* 1、要实现扩容，只能向上（垂直）扩展。比如磁盘限制了数据的存储，就要扩大磁盘容量，通过堆硬件的方式，不支持动态的扩缩容。水平扩容需要使用复杂的技术，必须分库分表。
* 2、表结构修改困难，因此存储的数据格式也受到限制。
* 3、在高并发和高数据量的情况下，关系型数据库需要把数据持久化到磁盘，磁盘的读写压力会比较大，网站处理速度将会受限。

非关系型数据库NoSQL，一般把它叫做"non-relational"或者"Not Only SQL"。
    
非关系型数据库的特点：

* 1、可以存储非结构化的数据，比如文本、图片、音频、视频等等。
* 2、表与表之间没有关联，可扩展性强，不受约束。
* 3、保证数据的最终一致性。遵循BASE（碱）理论，Basically Available（基本可用）、Soft-state（软状态）、Eventually Consistent（最终一致性）。
* 4、支持海量数据存储和高并发的高效读写。
* 5、支持分布式，能够对数据进行分片存储、扩缩容简单。

对于不同的存储类型，有不同的非关系型数据库，比如：

* 1、KV存储，用Key Value的形式来存储数据。常见的有Redis和MemcacheDB。
* 2、文档存储，MongoDB。
* 3、列存储，HBase。
* 4、图存储，这个图（Graph）是数据结构，不是文件格式，Neo4j。
* 5、对象存储。
* 6、XML存储等等。
具体的可参考 [nosql-database](http://nosql-database.org)，这个网站列举了各种各样的NoSQL数据库。

##1.3 Redis基本操作

Redis是字典结构的存储方式，采用key-value存储。key和value的最大长度限制是512M，来自[官网](https://redis.io/topics/data-types-intro)

#### Redis启动：

> 配置后台启动：daemonize yes
> 
> 启动Redis：/usr/local/soft/redis-5.0.5/src/redis-server /usr/local/soft/redis-5.0.5/redis.conf
> 
> redis 的参数可以通过三种方式配置，一种是指定自己的redis.conf，一种是启动时--携带的参数，一种是动态config set。
> 
> 也可以配置alias redis='/usr/local/soft/redis-5.0.5/src/redis-server /usr/local/soft/redis-5.0.5/redis.conf'

#### 基本操作：

默认有16个库（0-15），可以在配置文件中修改，默认使用第一个db0。

> 切换数据库：select index(0-15)

> 清空当前数据库：flushdb
 
> 清空所有数据库：flushall

常用命令参考该网站[http://redisdoc.com](http://redisdoc.com) 是 Redis Command Reference 和 Redis Documentation 的中文翻译版， 阅读这个文档可以帮助你了解 Redis 命令的具体使用方法，并学会如何使用 Redis 的事务、持久化、复制、Sentinel、集群等功能。

> 存值：set crazy xu

> 取值：get crazy

> 查看所有键：keys *

> 获取键总数：dbsize

> 查询键是否存在：exists crazy

> 删除键（可以批量）：del crazy crazy1 crazy2

> 重命名键：rename crazy crazy3

> 查看类型：type crazy3

##1.4 Redis基本数据类型

Redis 一共有几种数据类型？（注意是数据类型不是数据结构）
官网：[https://redis.io/topics/data-types-intro](https://redis.io/topics/data-types-intro)
Binary-safe strings、Lists、Sets、Sorted sets、Hashes、Bit arrays (or simply bitmaps)、HyperLogLogs、Streams
最基本也是最常用的数据类型就是String。set和get命令就是String的操作命令。

#### 1.4.1 String字符串
可以用来存储字符串、整数、浮点数。

* SET key value [EX seconds] [PX milliseconds] [NX|XX]
> * 如果key已经持有其他值，SET就覆写旧值， 无视类型。
> * EX seconds ： 将键的过期时间设置为 seconds 秒。 执行 SET key value EX seconds 的效果等同于执行 SETEX key seconds value 。
> * PX milliseconds ： 将键的过期时间设置为 milliseconds 毫秒。 执行 SET key value PX milliseconds 的效果等同于执行 PSETEX key milliseconds value 。
> * NX ：只在键不存在时，才对键进行设置操作。执行 SET key value NX 的效果等同于执行 SETNX key value。
> * XX ：只在键已经存在时，才对键进行设置操作。
>
> * 备注：因为 SET 命令可以通过参数来实现SETNX、SETEX以及PSETEX命令的效果， 所以Redis将来的版本可能会移除并废弃SETNX、SETEX和PSETEX这三个命令。

* SETNX key value
> 只在键key不存在的情况下，将键key的值设置为value。
> 命令在设置成功时返回1，设置失败时返回0。

* MSET key value [key value …]
> 如果某个给定键已经存在， 那么 MSET 将使用新值去覆盖旧值， 如果这不是你所希望的效果， 请考虑使用 MSETNX 命令， 这个命令只会在所有给定键都不存在的情况下进行设置。
> 
> MSET 是一个原子性(atomic)操作， 所有给定键都会在同一时间内被设置， 不会出现某些键被设置了但是另一些键没有被设置的情况。

* MGET key [key …]
> 返回给定的一个或多个字符串键的值。
>
> 如果给定的字符串键里面， 有某个键不存在， 那么这个键的值将以特殊值 nil 表示。

* APPEND key value
> 如果键 key 已经存在并且它的值是一个字符串， APPEND 命令将把 value 追加到键 key 现有值的末尾。
> 
> 如果 key 不存在， APPEND 就简单地将键 key 的值设为 value ， 就像执行 SET key value 一样。

* INCR key
> 为键 key 储存的数字值加上一。
> 
> 如果键 key 不存在， 那么它的值会先被初始化为 0 ， 然后再执行 INCR 命令。
> 
> 如果键 key 储存的值不能被解释为数字， 那么 INCR 命令将返回一个错误((error) ERR value is not an integer or out of range)。
> 
> INCR 命令会返回键 key 在执行加一操作之后的值。

* INCRBY key increment
> 为键 key 储存的数字值加上增量increment。
> 
> 如果键 key 不存在，那么键 key 的值会先被初始化为0，然后再执行INCRBY命令。
> 
> 如果键 key 储存的值不能被解释为数字，那么INCRBY命令将返回一个错误。

* INCRBYFLOAT key increment
> 为键 key 储存的值加上浮点数增量increment。

* DECR key
> 为键 key 储存的数字值减去一。
> 
> 如果键 key 不存在， 那么键 key 的值会先被初始化为 0 ， 然后再执行 DECR 操作。
> 
> 如果键 key 储存的值不能被解释为数字， 那么 DECR 命令将返回一个错误((error) ERR value is not an integer or out of range)。
> 
> DECR 命令会返回键 key 在执行减一操作之后的值。

* DECRBY key decrement
> 将键 key 储存的整数值减去减量 decrement 。
> 
> 如果键 key 不存在， 那么键 key 的值会先被初始化为 0 ， 然后再执行 DECRBY 命令。
> 
> 如果键 key 储存的值不能被解释为数字， 那么 DECRBY 命令将返回一个错误。

* STRLEN key
> 返回键 key 储存的字符串值的长度。当键 key 不存在时，命令返回0。当 key 储存的不是字符串值时，返回一个错误。

* GETRANGE key start end
> 返回键 key 储存的字符串值的指定部分， 字符串的截取范围由 start 和 end 两个偏移量决定 (包括 start 和 end 在内)。
> 
> 负数偏移量表示从字符串的末尾开始计数， -1 表示最后一个字符， -2 表示倒数第二个字符， 以此类推。
> 
> GETRANGE 通过保证子字符串的值域(range)不超过实际字符串的值域来处理超出范围的值域请求。








