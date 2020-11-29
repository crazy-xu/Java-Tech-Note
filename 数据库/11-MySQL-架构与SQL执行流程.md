### 客户端与服务端链接
通讯类型：同步/异步<br />
连接方式：长链接/短连接<br />
> 短连接，用完即关闭<br />

协议：TCP/Unix Socket<br />
> 各种程序语言，都是用TCP连接。

> show global status like 'Thread%'; -- 查看线程的活跃状态，分别是取消、连接、创建、运行。

MySQL是一个单进程多线程的模型，客户端每产生一个会话（session），就会有一个线程会处理这个操作。断开一个连接，把这个线程给kill掉。

> SHOW GLOBAL VARIABLES LIKE 'wait_timeout'; -- 非交互式超时时间，如JDBC程序。

> SHOW GLOBAL VARIABLES LIKE 'interactive_timeout'; -- 交互式超时时间，如数据库工具

如果要尽快回收资源，只需要把这个调小一些。

MySQL默认最大连接会话数：151，最大值可以调整100000。官网【max_connections】<br /> 
修改变量时，如果修改全局变量时需要加global，不加只对当前会话生效。

### 缓存
``
SHOW GLOBAL VARIABLES LIKE 'query_cache%';
``

5.7之后默认关闭状态，官方不推荐使用自带的缓存。应该把缓存交给更专业的工具去做。
* 大小写，空格，都不会命中缓存。
* 表中的任何一条数据变化，都会把整张表的缓存清空，需要再次写入，浪费内存。

### SQL解析
* 词法解析：把一个完整的SQL语句打碎成一个个的单词。
> select * from user where name = '哈哈哈哈哈';
* 语法解析：语法检查，不符合SQL语法的直接报错。
* pre processor 预处理器：语义解析，判断表名、字段等是否正确。
* optimizer 优化器：SQL语法优化，生成、选择执行路径（基于cost）。 一条SQl有不同的执行路径，如选索引、基准表等。子查询优化、等价谓词重写、条件化简、外连接消除、嵌套连接消除、连接消除、语义优化，非SPJ优化
* 执行计划 execution plans：
* 执行器 executor：
* 存储引擎 storage engine：innodb,memory,myisam表类型，建表时不指定，取库默认，可修改。查看DDL。engine=Innodb
> 如大数据插入，先用myisam插入，在将表类型修改为innodb。

不管任何的存储引擎，都是按相同的规范开发，所以改了之后不影响操作。插件式的存储引擎，可自己定制开发。 

> show engine innodb status;

``
查询很快，不需要持久化
历史数据，不经常修改，压缩
读写并发，需要一致性
``

### Innodb 内存缓冲区 buffer pool

1、读数据时先去buffer pool看一下数据页 ，是否在里边，是，直接返回
2、修改数据时，先把修改的数据页放入buffer pool中，不是立即写入到磁盘文件中，操作快。
从内存区域没有同步到磁盘文件时，是脏页，后台有个线程刷脏。

### redo log

作用：保证了内存数据的安全性，延迟刷盘实际，进而提升系统吞吐。写入内存后，直接返回。<br />
1、为Innodb提供了崩溃恢复的特性，实现持久化。<br />
2、redo log 记录的是“在某个数据页上做了什么修改”。属于物理日志。<br />
3、redo log 的大小是固定的，前面的内容会被覆盖，一旦写满（默认48M），就会触发buffer pool到磁盘的同步，以便腾出空间记录后面的修改。<br />

重启时，先去redo log中看看有没有数据，如果有，进行刷脏。


为啥要写内存和log文件，而不是直接写到磁盘中：顺序I/O和随机I/O


> SHOW GLOBAL VARIABLES LIKE 'innodb_log%';

### undo log
记录事务发生之前的数据状态，发生异常时回滚，保证原子性。
> SHOW GLOBAL VARIABLES LIKE 'undo%';

redo log 和undo log 同属于事务日志。

### 更新SQL执行流程
1、事务开始，从内存（buffer pool）或磁盘（data file）取到包含这条数据的数据页，返回给Server的执行器。<br />
2、Server的执行器修改数据页的这一行数据的值为"test"。<br />
3、记录name = abc 到undo log。<br />
4、记录name = test 到redo log。<br />
5、调用存储引擎接口，记录数据页到Buffer Pool（修改name = test）。<br />
6、事务提交。<br />

### Innodb 架构

1、内存结构
* Buffer Pool
> SHOW GLOBAL VARIABLES LIKE 'innodb_buffer_pool%'; -- 默认大小是128M，生产环境需要调大，越大，读写性能越高。<br />
    LRU内存淘汰机制
    
* Change Buffer
* Adaptive Hash Index
* Log Buffer 
> 刷盘时机<br />
> SHOW GLOBAL VARIABLES LIKE 'innodb_flush_log_at_trx_commit%';  -- 0, 1, 2

2、磁盘结构

### binlog
所有的存储引擎都可以使用，以事件的形式记录了所有的DDL和DML语句（因为他记录的是操作而不是数据值，属于逻辑日志），可以用来做主从复制和数据恢复。

崩溃恢复：
> 在崩溃恢复时，判断事务是否需要提交：<br />
1、binlog无记录，redo log 无记录：<br />
    在redo log写之前crash,恢复操作：回滚事务。<br />
2、binlog无记录，redo log状态prepare:<br />
    在binlog 写完之前的crash， 恢复操作：回滚事务。<br />
3、binlog有记录，redo log 状态为prepare:<br />
    在binlog写完提交事务之前的crash，恢复操作：提交事务。<br />
4、binlog有记录，redo log 状态为commit:<br />
    正常完成的事务，不需要恢复。<br />
    
### Innodb 内存结构：
1、Buffer Pool
缓冲池是主内存中的一个区域，读数据时先去buffer pool看一下数据页 ，是否在里边，是，直接返回。
修改数据时，先把修改的数据页放入buffer pool中，不是立即写入到磁盘文件中，操作快。
从内存区域没有同步到磁盘文件时，是脏页，后台有个线程进行刷脏。
2、Redo log
保证了内存数据的安全性，延迟刷盘实际，进而提升系统吞吐。写入内存后，直接返回。重启时，先去redo log中看看有没有数据，如果有，进行刷脏。
3、undo log
记录事务发生之前的数据状态，发生异常时回滚，保证原子性。

redo log 和undo log 同属于事务日志。

4、Change Buffer
更改缓冲区是一种特殊的数据结构，当二级索引页不在缓冲池中时，它们 会缓存这些更改 。
5、Adaptive Hash Index






















