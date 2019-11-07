## 1、Redis 事务
Redis的单个命令都是原子性的（get、set、hget、hset等等），如果使用多个命令时，需要把多个命令作为一个不可分割的处理序列，就需要用到事务。

类似Set命令，集合多个小命令。

Redis的事务两个特点：
> * 按进入队列的顺序执行。
> * 不会受到其他客户端请求的影响。

一共涉及到5个命令：multi、exec、discard、watch、unwatch

### 1.1 multi

> 标记一个事务块的开始。
> 
> 事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 exec 命令原子性(atomic)地执行。

> 通过 multi 的命令开启事务。事务不能嵌套，多个 multi 命令效果一样。

    127.0.0.1:6379> multi               --开启事务
    OK
    127.0.0.1:6379> incr crazy          --crazy+1
    QUEUED                              --标记添加到队列
    127.0.0.1:6379> incr crazy          --crazy+1
    QUEUED
    127.0.0.1:6379> incr crazy          --crazy+1
    QUEUED
    127.0.0.1:6379> exec                --执行，并返回每个命令的执行结果
    1) (integer) 1
    2) (integer) 2
    3) (integer) 3

### 1.2 exec
> 执行所有事务块内的命令。
> 
> 假如某个(或某些) key 正处于 WATCH 命令的监视之下，且事务块中有和这个(或这些) key 相关的命令，那么 EXEC 命令只在这个(或这些) key 没有被其他命令所改动的情况下执行并生效，否则该事务被打断(abort)。

### 1.3 discard
> 取消事务，放弃执行事务块内的所有命令。
> 
> 如果正在使用 WATCH 命令监视某个(或某些) key，那么取消所有监视，等同于执行命令 UNWATCH。

    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> set discard XX
    QUEUED
    127.0.0.1:6379> discard             --取消执行
    OK
    127.0.0.1:6379> get discard         --获取失败
    (nil)

### 1.4 watch key [key …]
> 监视一个(或多个) key，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

### 1.5 unwatch
> 取消watch命令对**所有key**的监视。
> 
> 如果在执行watch命令之后，exec命令或discard命令先被执行了的话，那么就不需要再执行UNWATCH了。
> 
> 因为 EXEC 命令会执行事务，因此 WATCH 命令的效果已经产生了；而 DISCARD 命令在取消事务的同时也会取消所有对 key 的监视，因此这两个命令执行之后，就没有必要执行 UNWATCH 了。

### 1.6 事务执行可能遇到问题
* 在执行exec之前发生错误
> 入队的命令存在语法错误，包括参数变量，参数名称等等（编译错误）。这种情况下，事务会被拒绝执行，队列中的所有命令都不会执行。

    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> set test 1
    QUEUED
    127.0.0.1:6379> hset test a
    (error) ERR wrong number of arguments for 'hset' command
    127.0.0.1:6379> exec
    (error) EXECABORT Transaction discarded because of previous errors.

* 在执行exec之后发生错误
> 比如，类型错误，比如对String使用了Hash的命令，这是一种运行时错误。

> 在如下这种发生了运行时异常的情况下，只有错误的命令没有被执行，但是其他命令没有受到影响。
    
    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> set test 1
    QUEUED
    127.0.0.1:6379> hset test a b
    QUEUED
    127.0.0.1:6379> set test2 2
    QUEUED
    127.0.0.1:6379> exec
    1) OK
    2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
    3) OK

### 1.6 总结
> multi执行后，客户端可以向服务器发送多条命令，这些命令不会被立即执行，而是放到一个队列中，当exec命令被调用时，队列里的所有命令会按顺序执行。如果没有执行exec，队列里的所有命令都不会被执行。中途想取消执行事务，通过discard清空事务队列，放弃执行。

> * 当发生语法错误异常时，队列中的所有命令都不会得到执行。
> * 当发生运行时异常时，只有错误的命令没有被执行，但是其他命令没有受到影响。

> 一个事务中存在错误，Redis 不回滚？
> 
> Redis 说，这个锅，我不背，，失败的命令是由程序猿编程错误造成的，而这些错误应该在开发测试的过程中被发现，而不应该出现在生产环境中。所以Redis选择了更简单、更快速的无回滚方式来处理事务。
                                                                      
## 2、Lua 脚本
Lua脚本是一种轻量级脚本语言，它是用C语言编写的，跟SQL的存储过程有些类似。

使用Lua脚本执行Redis命令有如下几个好处：
* 一次性发送多个命令，减少网络开销。
* Redis会将整个脚本作为一个整体执行，不会被其他请求打断，保持其原子性。
* 对于复杂的组合命令，可以把它放在文件中，可以实现程序间的命令集复用。

### 2.1 在Redis中调用Lua脚本





















