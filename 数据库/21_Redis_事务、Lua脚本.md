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

* eval script numKeys key [key...] arg [arg...]

> script参数是一段Lua脚本程序，它会被运行在Redis服务器上下文中。

> numKeys参数用于指定键名参数的个数。

> 键名参数 key[key...]从EVAL的第三个参数开始算起，表示在脚本中所用到的那些 Redis 键(key)，这些键名参数可以在 Lua 中通过全局变量 KEYS 数组，用1为基址的形式访问( KEYS[1] ， KEYS[2] ，以此类推)。

> 在命令的最后，那些不是键名参数的附加参数arg[arg ...]，可以在Lua中通过全局变量 ARGV 数组访问，访问的形式和 KEYS 变量类似( ARGV[1] 、 ARGV[2] ，诸如此类)。

    127.0.0.1:6379> eval "return {KEYS[1], KEYS[2], ARGV[1]}" 2 test test2 arg
    1) "test"
    2) "test2"
    3) "arg"

### 2.2 在Lua脚本中调用Redis命令

* redis.call(command, key[param1, param2...], arg[arg1,arg2...])
> 使用eval调用该方法时，只是执行该方法，并没有返回值return，可使用如下方法，将返回值返回。

    127.0.0.1:6379> eval "return redis.call('set',KEYS[1],ARGV[1])" 1 crazy xu
    OK
    127.0.0.1:6379> get crazy
    "xu"

### 2.3 使用脚本文件来执行Lua脚本
在文件夹下新建一个lua文件

    [root@VM_0_16_centos src]# cat test_lua.lua
    redis.call('set','crazy', '100')
    redis.call('incr', 'crazy')
    return redis.call('get', 'crazy')
    
执行Lua文件

    // 如果给Redis已设置密码，执行启动客户端命令时，需要增加-a参数。
    [root@VM_0_16_centos src]# redis-cli -a ****** --eval test_lua.lua 0
    Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
    "101"

### 2.4 lua脚本案例，对IP进行限流
需求：在X秒内，XX-IP只能访问Y次。
设计思路：用key记录IP，用Value记录访问次数，每次请求对值进行+1操作，如果是第一次访问，对key设置过期时间。最后获取值，判断次数是否超过Y。

* 创建一个如下Lua脚本命令


    [root@VM_0_16_centos src]# cat ip_limit.lua
    local num=redis.call('incr',KEYS[1])
    if tonumber(num)==1 then
       redis.call('expire',KEYS[1],ARGV[1])
       return 1
    elseif tonumber(num)>tonumber(ARGV[2]) then
       return 0
    else
       return 1
    end
    
    
* 传入参数，并执行
> 对IP（172.0.0.1)进行限制，时间为20秒只能请求1次
>
> 注：**Key值后边是参数值，中间要加上一个空格和一个逗号，在加上一个空格**
>
> 即：redis-cli -a ****** --eval [lua脚本名称] [key...] 空格逗号空格[args...]


    [root@VM_0_16_centos src]# redis-cli -a ****** --eval ip_limit.lua 172.0.0.1 , 20 1
    Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
    (integer) 1
    [root@VM_0_16_centos src]# redis-cli -a ****** --eval ip_limit.lua 172.0.0.1 , 20 1
    Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
    (integer) 0
    
### 2.5 缓存Lua脚本
对一些文件比较大的脚本，每次调用脚本都需要把整个脚本传给Redis服务端，会产生比较大的网络开销。
为了解决这个问题，Redis提供了evalsha命令，允许开发者通过脚本内容的SHA1摘要来执行脚本。

* script load lua脚本
> 将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。如果是个多行脚本，语句之间需要使用分号隔开。
> 
> eval 命令也会将脚本添加到缓存中，但是会立即执行这个脚本。

    127.0.0.1:6379> script load "return 'hello world'"
    "5332031c6b470dc5a0dd9b4bf2030dea6d65de91"
    127.0.0.1:6379> evalsha 5332031c6b470dc5a0dd9b4bf2030dea6d65de91 0
    "hello world"

* evalsha sha1 numkeys keys[keys...] arg[arg...]
> 根据给定的 sha1 校验码，对缓存在服务器中的脚本进行求值。

* script exists sha1[sha1...]
> 给定一个或多个脚本的 sha1 校验和，返回一个包含0和1的列表，表示校验和所指定的脚本是否已经被保存在缓存当中。

* script flush
> 清除所有 Lua 脚本缓存。

* script kill
> 杀死当前正在运行的 Lua 脚本，当且仅当这个脚本没有执行过任何写操作时，这个命令才生效。
> 
> 这个命令主要用于终止运行时间过长的脚本，比如一个因为 BUG 而发生无限 loop 的脚本，诸如此类。
>
> script kill 执行之后，当前正在运行的脚本会被杀死，执行这个脚本的客户端会从 EVAL script numkeys key [key …] arg [arg …] 命令的阻塞当中退出，并收到一个错误作为返回值。
>
> 另一方面，假如当前正在运行的脚本已经执行过写操作，那么即使执行 SCRIPT KILL ，也无法将它杀死，因为这是违反 Lua 脚本的原子性执行原则的。在这种情况下，唯一可行的办法是使用 SHUTDOWN NOSAVE 命令，通过停止整个 Redis 进程来停止脚本的运行，并防止不完整(half-written)的信息被写入数据库中。

### 2.6 脚本执行超时
Redis的指令执行本身是单线程的，这个线程还要执行客户端的Lua脚本，如果Lua脚本执行超过或者陷入了死循环，，，，，

为了防止某个脚本执行时间过长，Redis提供了lua-time-limit参数限制脚本的最长运行时间，默认是5秒钟。
> lua-time-limit 5000 (在redis.conf配置文件中)

> lua脚本里边提供了各式各样的钩子函数，Redis会在钩子函数没事时去处理客户端的请求，并且只有在lua脚本执行超时之后
才会处理请求，这个超时时间默认是5秒。


**完结**
