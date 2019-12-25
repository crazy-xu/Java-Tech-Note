CentOS 7默认安装了firewalld，如果没有安装的话，则需要YUM命令安装；

### 安装Firewall命令
> yum install firewalld firewalld-config

### 查看开放端口(默认不开放任何端口)
> firewall-cmd --list-ports

### 查看开启端口
> firewall-cmd --permanent --list-port

### 开启端口
> firewall-cmd --zone=public --add-port=80/tcp --permanent

> 命令含义：
> * --zone #作用域
> * --add-port=80/tcp #添加端口，格式为：端口/通讯协议
> * --permanent #永久生效，没有此参数重启后失效

### 添加添加区间端口
> firewall-cmd --zone=public --add-port=4400-4600/udp --permanent

### 关闭端口
> firewall-cmd --zone=public --remove-port=80/tcp --permanent

### 开启防火墙
> systemctl start firewalld
### 重启防火墙
> * systemctl restart firewalld
> * firewall-cmd --reload

### 停止防火墙
> systemctl stop firewalld

### 查看防火墙状态
> * systemctl status firewalld
> * firewall-cmd --state

### 设置开机启动
> systemctl enable firewalld

