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

#### Redis安装
1、下载Redis
> wget http://download.redis.io/releases/redis-5.0.5.tar.gz  // 自己指定目录/usr/local/soft/

2、解压压缩包
> tar -zxvf redis-5.0.5.tar.gz

3、安装gcc依赖，C语言编写的，编译需要
> yum install gcc   

4、编译安装
> cd redis-5.0.5

> make MALLOC=libc

将/usr/local/soft/redis-5.0.5/src目录下二进制文件安装到/usr/local/bin
> cd src
> 
> make install

#### Redis配置
> /usr/local/soft/redis-5.0.5/redis.conf
> 
> 配置后台启动：daemonize no --> daemonize yes
> 
> 配置外网访问：bind 127.0.0.1 --> bind 0.0.0.0 或注释
>
> 配置密码访问：requirepass yourpassword
#### Redis启动
> /usr/local/soft/redis-5.0.5/src/redis-server /usr/local/soft/redis-5.0.5/redis.conf
> 
> redis 的参数可以通过三种方式配置，一种是指定自己的redis.conf，一种是启动时--携带的参数，一种是动态config set。
> 
> 也可以配置alias redis='/usr/local/soft/redis-5.0.5/src/redis-server /usr/local/soft/redis-5.0.5/redis.conf'

#### Redis关闭
> ps -ef | grep redis
> 
> kill -9 ****

#### Redis打开客户端
* 如果没设置密码可直接登录
> /usr/local/soft/redis-5.0.5/src/redis-cli 或者在根目录执行redis-cli
* 设置密码后以如上方法登录，执行语句是会返回
> (error) NOAUTH Authentication required.
> 
> 此时可以执行auth myPassword获取权限
* 设置密码后通过密码直接登录
> redis-cli -h 127.0.0.1 -p 6379 -a myPassword
>
> **注：-h/-p 可以直接忽略，可直接执行redis-cli -a myPassword**

#### 基本操作

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


当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过1M，扩容时一次只会多扩1M的空间。(字符串最大长度为 512M)，通过checkStringLength()来检测容量大小。

不同类型的数据结构的差异就在于value的结构不一样。但是这个key使用的结构，都是字符串类型。

### 1.4.1 String 字符串
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
> 如果键 key 不存在， 那么键 key 的值会先被初始化为0，然后再执行 DECRBY 命令。
> 
> 如果键 key 储存的值不能被解释为数字， 那么 DECRBY 命令将返回一个错误。

* STRLEN key
> 返回键 key 储存的字符串值的长度。当键 key 不存在时，命令返回0。当 key 储存的不是字符串值时，返回一个错误。

* GETRANGE key start end
> 返回键 key 储存的字符串值的指定部分，字符串的截取范围由 start 和 end 两个偏移量决定 (包括 start 和 end 在内)。
> 
> 负数偏移量表示从字符串的末尾开始计数，-1表示最后一个字符，-2表示倒数第二个字符，以此类推。
> 
> GETRANGE 通过保证子字符串的值域(range)不超过实际字符串的值域来处理超出范围的值域请求。

#### 存储（实现）原理
    
Redis是KV的数据库，它是通过hashtable实现的，每个键值对都会有一个dictEntry（源码位置：dict.h），里面指向了key和value的指针。next指向下一个dictEntry。

key是字符串，但是Redis没有直接使用C的字符数组，而是存储在自定义的SDS中。
value既不是直接作为字符串存储，也不是直接存储在SDS中，而是存储在redisObject中。五种常用类型，都是通过redisObject存储的。

##### SDS:
> 全拼为：simple dynamic string，解释为：简单动态字符串。
> 数据结构与API相关文件是：sds.h, sds.c。
> SDS本质上就是char *，因为有了表头sdshdr结构的存在，所以SDS比传统C字符串在某些方面更加优秀，并且能够兼容传统C字符串。
> sds在Redis中是实现字符串对象的工具，并且完全取代char*..sds是二进制安全的，它可以存储任意二进制数据，不像C语言字符串那样以‘\0’来标识字符串结束，
> 因为传统C字符串符合ASCII编码，这种编码的操作的特点就是：遇零则止 。即，当读一个字符串时，只要遇到’\0’结尾，就认为到达末尾，就忽略’\0’结尾以后的所有字符。因此，如果传统字符串保存图片，视频等二进制文件，操作文件时就被截断了。
> SDS表头的buf被定义为字节数组，因为判断是否到达字符串结尾的依据则是表头的len成员，这意味着它可以存放任何二进制的数据和文本数据，包括’\0’
> SDS 和传统的 C 字符串获得的做法不同，传统的C字符串遍历字符串的长度，遇零则止，复杂度为O(n)。而SDS表头的len成员就保存着字符串长度，所以获得字符串长度的操作复杂度为O(1)。
> 总结下sds的特点是：可动态扩展内存、二进制安全、快速遍历字符串 和与传统的C语言字符串类型兼容。
> SDS又有多种结构（sds.h）：sdshdr5、sdshdr8、sdshdr16、sdshdr32、sdshdr64，用于存储不同的长度的字符串，分别代表 2^5=32byte，2^8=256byte，2^16=65536byte=64KB，2^32byte=4GB。


    src/dict.h
    typedef struct dictEntry {
        void *key;
        union {
            void *val;
            uint64_t u64;
            int64_t s64;
            double d;
        } v;
        struct dictEntry *next;
    } dictEntry;
    
##### RedisObject

    /** redisObject定义在src/server.h文件中。*/
    typedef struct redisObject {
        unsigned type:4;
        unsigned encoding:4; /* 字符串有3种编码 */
        unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                                * LFU data (least significant 8 bits frequency
                                * and most significant 16 bits access time). */
        int refcount; /* 当它为0时，说明没有引用，可以进行内存回收 */
        void *ptr; /* 指向真正的类型 */
    } robj;

##### 字符串类型的内部编码有三种：
> 1、int，存储8个字节的长整型（long，2^63-1）。
> 
> 2、embstr, 代表embstr格式的SDS（SimpleDynamicString简单动态字符串），存储小于44个字节的字符串。
> 
> 3、raw，存储大于44个字节的字符串。

**_embstr和raw的区别？_**
> embstr 的使用只分配一次内存空间（因为RedisObject 和SDS是连续的）， 而 raw需要分配两次内存空间（分别为RedisObject和SDS分配空间）。

> 因此与 raw 相比，embstr 的好处在于创建时少分配一次空间，删除时少释放一次空间，以及对象的所有数据连在一起，寻找方便。
> 
> 而 embstr 的坏处也很明显，如果字符串的长度增加需要重新分配内存时，整个RedisObject和SDS都需要重新分配空间，因此Redis中的embstr实现为只读。

**_int和embstr什么时候转化为raw？_**
> 当int数据不再是整数，或大小超过了long的范围（2^63-1=9223372036854775807）时，自动转化为embstr

**_明明没有超过阈值，为什么变成raw了？_**
> 对于embstr，由于其实现是只读的，因此在对embstr对象进行修改时，都会先转化为 raw 再进行修改。
  因此，只要是修改embstr对象，修改后的对象一定是raw的，无论是否达到了44个字节。

**_当长度小于阈值时，会还原吗？_**
> 编码转换在Redis写入数据完成，且转换过程不可逆，只能从小内存编码向大内存编码转换（但是不包括重新set）。

**_RedisObject为什么要对底层的数据进行一层包装呢？_**
> 通过封装，可以根据对象的类型动态的选择存储结构和可以使用的命令，实现节省空间和优化查询速度。比如，当字符串小时，使用enbstr，不满足时才使用raw，优化空间。

#### 应用场景
* 1、缓存
> 热点数据缓存，对象缓存等等，可以提升该数据的访问速度。
* 2、数据共享
> 分布式session，因为Redis是个独立服务，可以在多个应用之间共享数据。
* 3、分布式锁
> setnx方法，只有不存在时才能添加成功，返回true。
* 4、全局ID
> incrby方法，利用其原子性（incrby userid 1000）。
* 5、计数器
> int类型，incr方法，文章的阅读量、微博点赞数，访问量等等。
* 6、限流
> int类型，incr方法，以访问者的IP作为key，每访问一次，次数加一，限制次数。
* 7、位统计
> 具体参考String类型的BITCOUNT方法，说实话，我不太懂。。。！@#￥%……&*（）

### 1.4.2 Hash 哈希
包含键值对的无序散列表。value 只能是字符串，不能嵌套其他类型。

* HSET hash field value
> 时间复杂度：O(1)
> 
> 将哈希表hash中域field的值设置为value。
> 
> 如果给定的哈希表并不存在，那么一个新的哈希表将被创建并执行HSET操作。
> 
> 如果域field已经存在于哈希表中，那么它的旧值将被新值value覆盖。
>
> 当HSET命令在哈希表中新创建field域并成功为它设置值时，命令返回1；如果域field已经存在于哈希表，并且HSET命令成功使用新值覆盖了它的旧值，那么命令返回0。

* HSETNX hash field value
> 时间复杂度：O(1)
> 
> 当且仅当域 field 尚未存在于哈希表的情况下， 将它的值设置为 value 。
> 
> 如果给定域已经存在于哈希表当中， 那么命令将放弃执行设置操作。
> 
> 如果哈希表 hash 不存在， 那么一个新的哈希表将被创建并执行 HSETNX 命令。
>
> HSETNX 命令在设置成功时返回1， 在给定域已经存在而放弃执行设置操作时返回0。

* HGET hash field
> 时间复杂度：O(1)
> 
> HGET 命令在默认情况下返回给定域的值。
> 
> 如果给定域不存在于哈希表中，又或者给定的哈希表并不存在，那么命令返回nil。

* HEXISTS hash field
> 时间复杂度：O(1)
> 
> 检查给定域field是否存在于哈希表hash当中。
> 
> HEXISTS命令在给定域存在时返回1，在给定域不存在时返回0。

* HDEL key field [field …]
> 时间复杂度：O(N)， N 为要删除的域的数量。
> 
> 删除哈希表 key 中的一个或多个指定域，不存在的域将被忽略。

* HLEN key
> O(1)。哈希表中域的数量。当key不存在时，返回0。

* HSTRLEN key field
> O(1)。与给定域field相关联的值的字符串长度（string length）。如果给定的键或者域不存在，那么命令返回0。

* hincrby key field increment
> 为哈希表key中的域field的值加上增量increment。其也可以是负数，相当于给定域进行减法操作。
> 
> 如果key不存在，一个新的哈希表被创建并执行hincrby命令。
> 
> 如果field不存在，那么执行命令前，域的值被初始化为0。
> 
> 对一个存储字符串值得域field执行hincrby命令将造成一个错误。((error) ERR hash value is not an integer)
> 
> 本操作的值被限制在64位（bit）符号数字表示之内。

* hincrbyfloat key field increment
> 为哈希表key中的域field加上浮点数增量increment。

* hmset key field value [field value...]
> 同时将多个field-value(域-值)对设置到哈希表key中。
> 
> 此命令会覆盖哈希表中已存在的域。如果key不存在，一个空哈希表被创建并执行HMSET操作。

* hmget key field [field...]
> 返回哈希表key中，一个或多个给定域的值。如果给定的域不存在于哈希表，那么返回一个nil值。

* hkeys key
> 返回哈希表key中的所有域。时间复杂度：o(n)。
> 
> 当key不存在时，返回一个空表。(empty list or set)

* hvals key
> 返回哈希表key中所有域的值。时间复杂度：o(n)。
> 
> 当key不存在时，返回一个空表。(empty list or set)

* hgetall key
> 返回哈希表key中，所有的域和值。在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。
>
> 若key不存在，返回一个空表。

#### 存储（实现）原理
外层的Hash（Redis KV的实现）只用到了hashtable。当存储hash数据类型时，我们把它叫做内层的哈希，内层的哈希底层使用了两种数据结构实现：
* ziplist:OBJ_ENCODING_ZIPLIST(压缩列表)
* hashtable:OBJ_ENCODING_HT(哈希表)

当hash对象同时满足以下两个条件的时候，使用ziplist编码：

1、所有的键值对的键和值得字符串长度都小于等于64byte（一个英文字母一个字节）。

    # redis.conf ziplist 中最大能存放的值长度
    hash-max-ziplist-value 64
2、hash对象保存的键值对数量小于512个。

    # redis.conf ziplist 中最多能存放的 entry 节点数量
    hash-max-ziplist-entries 512
一个hash对象超过配置的阈值时，会转换成hash表（hashtable）。



### 1.4.3 List 列表

存储有序的字符串（从左到右），元素可以重复。可以充当队列和栈的角色。

使用PUSH命令，允许加入重复数据。

* LPUSH key value [value …]
> 将一个或多个值 value 插入到列表 key 的表头
> 
> 如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表头：比如说，对空列表 mylist 执行命令 LPUSH mylist a b c，列表的值将是 c b a，这等同于原子性地执行 LPUSH mylist a、LPUSH mylist b 和 LPUSH mylist c 三个命令。
> 
> 如果 key 不存在，一个空列表会被创建并执行 LPUSH 操作。
> 
> 当 key 存在但不是列表类型时，返回一个错误。正确返回值是列表的长度。

* LPUSHX key value
> 将值 value 插入到列表 key 的表头，当且仅当 key 存在并且是一个列表。和 LPUSH key value 命令相反，当 key 不存在时，LPUSHX命令什么也不做。

* RPUSH key value [value …]
> 将一个或多个值 value 插入到列表 key 的表尾(最右边)。
> 
> 如果有多个 value 值，那么各个 value 值按从左到右的顺序依次插入到表尾：比如对一个空列表 mylist 执行 RPUSH mylist a b c ，得出的结果列表为 a b c ，等同于执行命令 RPUSH mylist a 、 RPUSH mylist b 、 RPUSH mylist c 。
> 
> 如果 key 不存在，一个空列表会被创建并执行 RPUSH 操作。
> 
> 当 key 存在但不是列表类型时，返回一个错误。

* RPUSHX key value
> 将值 value 插入到列表 key 的表尾，当且仅当 key 存在并且是一个列表。和 RPUSH key value 命令相反，当 key 不存在时， RPUSHX 命令什么也不做。

* LPOP key
> 移除并返回列表 key 的头元素。

* RPOP key
> 移除并返回列表 key 的尾元素。



### 1.4.4 Set 集合

String 类型的无序集合，最大存储数量 2^32-1（40 亿左右）。

Redis 用 intset 或 hashtable 存储 set。如果元素都是整数类型，就用 inset 存储。如果不是整数类型，就用 hashtable（数组+链表的存来储结构）。如果元素个数超过 512 个，也会用 hashtable 存储。

* SADD key member [member …]
> 将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略。
> 
> 假如 key 不存在，则创建一个只包含 member 元素作成员的集合。
> 
> 当 key 不是集合类型时，返回一个错误。

* SISMEMBER key member
> 如果 member 元素是集合的成员，返回1。 如果 member 元素不是集合的成员，或 key 不存在，返回0。

* SPOP key
> 移除并返回集合中的一个随机元素。被移除的随机元素。 当 key 不存在或 key 是空集时，返回 nil。

* SRANDMEMBER key [count]
> 如果命令执行时，只提供了 key 参数，那么返回集合中的一个随机元素。
> 
> 从 Redis 2.6 版本开始， SRANDMEMBER 命令接受可选的 count 参数：
> 
> * 如果 count 为正数，且小于集合基数，那么命令返回一个包含 count 个元素的数组，数组中的元素各不相同。如果 count 大于等于集合基数，那么返回整个集合。
> 
> * 如果 count 为负数，那么命令返回一个数组，数组中的元素可能会重复出现多次，而数组的长度为 count 的绝对值。
> 
> 该操作和 SPOP key 相似，但 SPOP key 将随机元素从集合中移除并返回，而 SRANDMEMBER 则仅仅返回随机元素，而不对集合进行任何改动。

* SREM key member [member …]
> 移除集合key中的一个或多个member元素，不存在的member元素会被忽略。

* SMOVE source destination member
> 将 member 元素从 source 集合移动到 destination 集合。
> 
> SMOVE 是原子性操作。
> 
> 如果 source 集合不存在或不包含指定的 member 元素，则 SMOVE 命令不执行任何操作，仅返回 0 。否则， member 元素从 source 集合中被移除，并添加到 destination 集合中去。
> 
> 当 destination 集合已经包含 member 元素时， SMOVE 命令只是简单地将 source 集合中的 member 元素删除。
> 
> 当 source 或 destination 不是集合类型时，返回一个错误。

* SCARD key
> 返回集合 key 的基数(集合中元素的数量)。

* SMEMBERS key
> 返回集合 key 中的所有成员。（不存在即返回(empty list or set)）

* SINTER key [key …]
> 返回一个集合的全部成员，该集合是所有给定集合的交集。

* SDIFF key [key …]
> 返回一个集合的全部成员，该集合是所有给定集合之间的差集。

* SUNION key [key …]
> 返回一个集合的全部成员，该集合是所有给定集合的并集。

### 1.4.5 Sorted Set 有序集合
Sorted Set，有序的set，每个元素有个score。score 相同时，按照 key 的 ASCII 码排序。

* ZADD key score member [[score member] [score member] …]
> 将一个或多个 member 元素及其 score 值加入到有序集 key 当中。
> 
> 如果某个 member 已经是有序集的成员，那么更新这个 member 的 score 值，并通过重新插入这个 member 元素，来保证该 member 在正确的位置上。
> 
> score 值可以是整数值或双精度浮点数。
> 
> 如果 key 不存在，则创建一个空的有序集并执行 ZADD 操作。
> 
> 当 key 存在但不是有序集类型时，返回一个错误。

### 1.4.6 HyperLogLog


### 1.4.7 Streams


### 1.4.8 发布与订阅
* publish channel message
> 将信息 message 发送到指定的频道channel。
> 接收到信息 message 的订阅者数量。

* subscribe channel[channel...]
> 订阅给定的一个或者多个频道
> 返回值：
    
    127.0.0.1:6379> publish test crazy
    (integer) 1
    
    127.0.0.1:6379> subscribe test
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"      # 返回值的类型：显示订阅成功
    2) "test"           # 订阅的频道名字
    3) (integer) 1      # 目前已订阅的频道数量
    1) "message"        # 返回值的类型：信息
    2) "test"           # 来源(从那个频道发送过来)
    3) "crazy"          # 信息内容

* psubscribe pattern [pattern...]
> 订阅一个或者多个符合给定模式的频道。
> 
> 每个模式以*作为匹配符，比如it*匹配所有以it开头的频道（it.abc,it.blog等等）。

* unsubscribe [channel [channel...]]
> 指示客户端退订给定的频道。
> 
> 如果没有频道被指定，也即是，一个无参数的 UNSUBSCRIBE 调用被执行，那么客户端使用 SUBSCRIBE 命令订阅的所有频道都会被退订。在这种情况下，命令会返回一个信息，告知客户端所有被退订的频道。



此处省略好多，不太了解......