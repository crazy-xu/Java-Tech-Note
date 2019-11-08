## 1、Redis为什么这么快

### 1.1、有多快？
Redis官网：[redis.io](https://redis.io/topics/benchmark)

根据官方给出的数据，Redis的QPS可以达到10万左右（每秒请求数）

* 根据官方提供的工具测试各种命令请求处理速度


    [root@VM_0_16_centos redis-5.0.5]# cd src/
    [root@VM_0_16_centos src]# redis-benchmark -t set,get -n 100000 -q
    SET: 69832.40 requests per second
    GET: 72046.11 requests per second
    
* 测试lua脚本处理速度
    
    
    [root@VM_0_16_centos src]# redis-benchmark -n 100000 -q script load "redis.call('set', 'crazy','xu')"
    script load redis.call('set', 'crazy','xu'): 73746.31 requests per second
    
### 1.2 为什么这么快呀~

* 是一个纯内存的KV结构的数据库，时间复杂度是O(1)。
* 单线程
    不需要频繁的创建和销毁线程。
    避免上下文切换所导致的CPU的消耗。
    不需要频繁的加锁释放锁等等，没有资源竞争。
* 采用异步非阻塞I/O，多路复用处理并发连接。

在Redis这里单线程已经够用了，CPU不是Redis的瓶颈，它的瓶颈最有可能的是机器内存或者网络宽带。

## 2、内存回收
Redis所有的数据都是存储在内存中的，在某些情况下需要对占用的内存空间进行回收。

内存回收主要分为两类：
* key过期
* 内存使用达到上限(max_memory)，触发内存淘汰。

### 2.1 过期策略
> Redis中同时使用了惰性过期和定期过期两种过期策略。

#### 2.1.1 定期过期删除
* redis 默认是每隔 100ms 就随机抽取一些设置了过期时间的key，检查其是否过期，如果过期就删除。

> 注：这里是随机抽取一定数量的Key。
> 如果采用定时过期，会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。


#### 2.1.2 惰性过期删除
* 只有当访问一个Key时，才会判断该Key是否已经过期，过期则删除，不会返回任何东西。该策略可以最大化的节省CPU资源，却对内存非常不友好。
极端情况下可能出现大量的过期Key没有再次被访问，从而不会被清除，占用大量内存。

> 在getCommand里面会调用expireIfNeeded方法。在server.c expireIfNeeded

* 每次写入Key时，发现内存不够，调用activeExpireCycle释放一部分内存。
> expire.c activeExpireCycle(int type)
    
### 2.2 淘汰策略
Redis的内存淘汰策略，是指当内存使用达到最大内存极限时，需要使用淘汰算法来决定清理掉哪些数据，来保证新数据的存入。

#### 2.2.1 最大内存设置
> 可以通过不设置maxmemory或者设置为0，64位系统不限制内存，32位系统最多使用3GB内存。

> * 1.在redis.conf 参数配置：# maxmemory<bytes>
> * 2.通过客户端命令：config set maxmemory 2GB

#### 2.2.2 淘汰策略
在其[官网](https://redis.io/topics/lru-cache) 中可以看到如下淘汰策略：
* noeviction: return errors when the memory limit was reached and the client is trying to execute commands that could result in more memory to be used (most write commands, but DEL and a few more exceptions).
> 默认策略，不会删除任何数据，拒绝所有写入操作并返回客户端错误信息（(error)OOM）,此时只响应读操作。
* allkeys-lru: evict keys by trying to remove the less recently used (LRU) keys first, in order to make space for the new data added.
> 根据LRU算法删除键，不管数据有没有设置超时属性，知道腾出足够内存为止。
* volatile-lru: evict keys by trying to remove the less recently used (LRU) keys first, but only among keys that have an expire set, in order to make space for the new data added.
> 根据LRU算法删除设置了超时属性（expire）的键，直到腾出足够内存为止。如果没有可删除的键，执行noeviction策略。
* allkeys-random: evict keys randomly in order to make space for the new data added.
> 随机收回key，以便为添加的新数据腾出空间。
* volatile-random: evict keys randomly in order to make space for the new data added, but only evict keys with an expire set.
> 随机收回key，以便为添加的新数据腾出空间，但仅收回具有expire属性的key。
* volatile-ttl: evict keys with an expire set, and try to evict keys with a shorter time to live (TTL) first, in order to make space for the new data added.
> 使用expire属性回收最近将要过期的key。
* volatile-lfu Evict using approximated LFU among the keys with an expire set.
> 在带有expire属性的key中随机选择。
* allkeys-lfu Evict any key using approximated LFU.
> 在所有的键中选择最不常用的，不管key有没有设置expire属性。

> 注：如果没有符合前提条件的key被淘汰，那么volatile-lru、volatile-random、volatile-ttl相当于执行noeviction策略。

修改淘汰策略方法：
> * redis.conf中，# maxmemory-policy noeviction
> * 通过客户端命令：config set maxmemory-policy volatile-lru


* LRU是什么？
> 按照英文的直接原义就是Least Recently Used,最近最久未使用法，它是按照一个非常著名的计算机操作系统基础理论得来的：最近使用的页面数据会在未来一段时期内仍然被使用,已经很久没有使用的页面很有可能在未来较长的一段时间内仍然不会被使用。基于这个思想,会存在一种缓存淘汰机制，每次从内存中找到最久未使用的数据然后置换出来，从而存入新的数据！它的主要衡量指标是使用的时间，附加指标是使用的次数。在计算机中大量使用了这个机制，它的合理性在于优先筛选热点数据，所谓热点数据，就是最近最多使用的数据！因为，利用LRU我们可以解决很多实际开发中的问题，并且很符合业务场景。

* LRU淘汰原理
> Redis LRU对传统的LRU算法进行了改良，通过随机采样来调整算法的精度。
>
> 如果淘汰策略是LRU，则根据配置的采样值maxmemory_samples(默认是5个)，随机从数据库中选择key，淘汰其热度最低的key对应的缓存数据。

* 如果找到热度最低的数据？
> Redis中所有对象结构都有一个lru字段，且使用了unsigned的低24位，这个字段用来记录对象的热度。
对象被创建时会记录lru值。在被访问的时候也会更新lru。但是不是获取的当前时间戳，而是设置为全局变量：server.lruclock的值。

> 在server.h中，redisObject方法的lru属性

> 在Redis中有个定时处理的函数serverCron，默认每100毫秒调用函数updateCachedTime，更新一次全局变量的server.lruclock的值，记录的是当前unix的时间戳。

> 对象中的lru值和server.lruclock值得差值越大（越久没有得到更新）,该对象热度越低。
> * server.lruclock只有24位，按秒为单位来表示才能存储194天，当超过24bit能表示最大时间的时候，会从头开始计算。


* LFU??
稍等更新，先理解理解


## 3、持久化机制

* RDB快照（Redis DataBase）
* AOF（Append Only File）

### 3.1 RDB
RDB是Redis默认的持久化方案。当满足一定条件的时候，会把当前内存中的数据写入磁盘，生成一个快照文件dump.rdb。Redis重启后会通过加载dump.rdb文件恢复数据。

#### 3.1.1 RDB自动触发操作
* 配置规则触发
> redis.conf
    
    # 以下的配置满足任意一个都会触发。如果不需要RDB方案，注释save或者配置成空字符串""。
    save 900 1       # 900秒内至少有一个key被修改（包括添加）
    save 300 10      # 400秒内至少有十个key被修改
    save 60 10000    # 60秒内至少有1万个key被修改
    
    
    dir ./redis-5.0.5/XX  # 文件路径
    dbfilename dump.rdb   # 文件名称
    rdbcompression yes    # 是否是LZF压缩rdb文件
    rdbchecksum yes       # 是否开启数据完整性校验

* shutdown 触发，保证服务器正常关闭
* flushall RDB文件是空的

#### 3.1.2 RDB手动触发操作
* save
> 在生成快照的时候会阻塞当前Redis服务器，不能处理其他命令。如果内存中的数据比较多，会造成Redis长时间阻塞。生产环境不建议使用这个命令。

* bgsave
> Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。
>
> 具体操作是Redis进程执行fork操作，创建子进程（copy-on-write），RDB持久化过程由子进程负责，完成后自动结束。不会记录fork之后的命令，阻塞只发生在fork阶段，一般时间很短。

> 用lastsave命令可以查看最近一次成功生成快照的时间。

#### 3.1.3 RDB数据的恢复
通过shutdown操作会触发save操作，生成dump.rbd文件，再次启动Redis时，会读取。

#### 3.1.4 RDB文件的优势和劣势












