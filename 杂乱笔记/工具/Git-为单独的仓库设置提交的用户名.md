在家里或者公司中，一般会参与多个项目的开发。对应个人或者公司的项目，提交时需要使用不同的git用户名/Email。

大部分情况下，git配置时使用全局配置，即多个项目使用一个用户名。

### 全局配置
在公司的a.b.c等等项目中使用"真实姓名/公司邮箱"，这时候可以设置全局配置。
> 在gitbash上执行如下命令
> 
> git config --global user.name "xxx"
> 
> git config --global user.email "xxx@163.com"

> 如果是window电脑，可用在自己的用户目录下找到一个 .gitconfig 文件，里面有刚才的配置，进行直接修改。

### 项目单独配置
找到项目所在的目录，与.git平级，在命令行中执行如下命令：
> git config user.name "xxx"
> 
> git config user.email "xxx@163.com"

> 如下为本机操作时执行的命令
> 
> ~/Documents/code/test/Java-Tech-Note$git config user.name "crazy-xu"
> 
> ~/Documents/code/test/Java-Tech-Note$git config user.email "youxu66@163.com"


**使用 git config --list 命令进行查看，可以看到全局和本项目的配置**

