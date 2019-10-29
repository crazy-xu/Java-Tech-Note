对于一些常用的比较冗余的命令，经常输入会比较麻烦，在Linux下，提供了一种alias别名命令，可以使用简单的一个别名执行等价的冗余命令。但alias命令默认仅针对当前终端有效，一旦开启新的终端窗口之前的alias别名便会实效。（后来才发现）

在学习 Redis 时，经常通过命令打开客户端，每次执行都比较复杂。
>  /usr/local/soft/redis-5.0.5/src/redis-cli

通过 alias 别名简化命令
> alias rcli='/usr/local/soft/redis-5.0.5/src/redis-cli'

这样只需要在命令行里输入rcli就等同于运行了 /usr/local/soft/redis-5.0.5/src/redis-cli

但是后来发现这样都是“临时”的，只要你关闭了当前的SSH链接后，再次SSH登录到控制台终端的时候，这些别名设置就失效了，纳闷。

经查询，要想让其永久生效只需要将这些 alias 别名设置保存到文件：/root/.bashrc里面就可以了。

> cat /root/.bashrc

    # .bashrc
    
    # User specific aliases and functions
    
    alias rm='rm -i'
    alias cp='cp -i'
    alias mv='mv -i'
    # 新增
    alias rcli='/usr/local/soft/redis-5.0.5/src/redis-cli'
    
    # Source global definitions
    if [ -f /etc/bashrc ]; then
    	. /etc/bashrc
    fi
    . "/root/.acme.sh/acme.sh.env"

如上文件中已经有了一些 alias 的设置了，就是rm、cp、mv的，我们只需要编辑/root/.bashrc在里面添加上我们需要的别名设置保存退出即可。

最后，执行
> source /root/.bashrc

> 使用source命令{注1}让这个初始化文件生效，这样以后再次通过SSH进入控制台别名设置就不会丢失了，也就实现了永久生效了。

> source命令也称为“点命令”，也就是一个点符号（.）,是bash的内部命令。功能：使Shell读入指定的Shell程序文件并依次执行文件中的所有语句。source`命令通常用于重新执行刚修改的初始化文件，使之立即生效，而不必注销并重新登录。


