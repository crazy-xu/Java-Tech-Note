### 检查有没有已经安装 MySQL
rpm -qa | grep mysql

### 删除已经安装的 MySQL
rpm -e --nodeps xxx
> 如果不存在（上面检查结果返回空）则跳过步骤

### 添加 MySQL YUM 源

> wget 'https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm'
> 
> 如果需要最新的，可前去官网查询https://dev.mysql.com/downloads/repo/yum/

### 安装 RPM
> sudo rpm -Uvh mysql80-community-release-el7-3.noarch.rpm

### 查询是否安装成功，Mysql版本
> yum repolist all | grep mysql

### 切换版本
> * sudo yum-config-manager --disable mysql80-community
> * sudo yum-config-manager --enable mysql57-community

### 查询当前启动仓库
> yum repolist enabled | grep mysql

### 安装 MySQL
> sudo yum install mysql-community-server

该命令会安装MySQL服务器 (mysql-community-server) 及其所需的依赖、相关组件，包括mysql-community-client、mysql-community-common、mysql-community-libs等
如果带宽不够，这个步骤时间会比较长，请耐心等待~

遇到选择，各种YYY

### 启动 MySQL 服务
> systemctl start mysqld

### 查询 MySQL 状态
```
[root@VM_0_16_centos ~]# systemctl status mysqld
  ● mysqld.service - MySQL Server
     Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
     Active: active (running) since Mon 2019-12-09 18:02:45 CST; 1min 27s ago
       Docs: man:mysqld(8)
             http://dev.mysql.com/doc/refman/en/using-systemd.html
    Process: 15971 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
   Main PID: 16061 (mysqld)
     Status: "Server is operational"
     CGroup: /system.slice/mysqld.service
             └─16061 /usr/sbin/mysqld
  
  Dec 09 18:02:36 VM_0_16_centos systemd[1]: Starting MySQL Server...
  Dec 09 18:02:45 VM_0_16_centos systemd[1]: Started MySQL Server.
```
### 初始密码查询
> sudo grep 'temporary password' /var/log/mysqld.log

> cat /var/log/mysqld.log | grep -i 'temporary password'

### 登录数据库，修改密码
> mysql -u root -p
> 
> 然后输入密码
> 
> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
> 
> ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```
出现上面的提示是因为密码太简单了，解决方法如下：

使用复杂密码，MySQL默认的密码策略是要包含数字、字母及特殊字符；
如果只是测试用，不想用那么复杂的密码，可以修改默认策略，即validate_password_policy（以及validate_password_length等相关参数），
使其支持简单密码的设定，具体方法可以自行百度；
修改配置文件/etc/my.cnf，添加validate_password=OFF，保存并重启MySQL
```

### 设置远程登录
进入到mysql
> use mysql;

查询当前登录地址,正常情况，host=localhost
> select host,user from user where user='root';

修改地址
> update user set host='%' where user='root';#更改host为所有ip

> select host,user from user where user='root'; #查看更改，此时 host=%

> 修改host字段的值，将localhost修改成需要远程连接数据库的ip地址。或者直接修改成%。修改成%表示，所有主机都可以通过root用户访问数据库。

* 最后，切记要进行刷新，否则更新无效。
> FLUSH PRIVILEGES;

### Navicat 登录报错
* 1251-client does not support authentication protocol requested by server;
> select host,user,plugin,authentication_string from mysql.user;
> 
> plugin非mysql_native_password 则需要修改密码
```
mysql> select host,user,plugin,authentication_string from mysql.user;
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
| host      | user             | plugin                | authentication_string                                                  |
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
| %         | root             | caching_sha2_password | $A$005$z?z7`ladu(}dJaKVmodzlJOxMKknMEAHdlswDrRDz/4lYqgA0ujOylOqUA |
| localhost | mysql.infoschema | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.session    | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
| localhost | mysql.sys        | caching_sha2_password | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED |
+-----------+------------------+-----------------------+------------------------------------------------------------------------+
4 rows in set (0.00 sec)

mysql>ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'newpassword';#更新一下用户的密码 root用户密码为newpassword
```

## CentOS7 yum方式安装MySQL 5.7
> 在CentOS中默认安装有MariaDB（MySQL的一个分支），安装完成之后可以直接覆盖MariaDB。

### 下载yum repository
``
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
``
### 安装
``
yum -y install mysql57-community-release-el7-10.noarch.rpm
``
### 安装MySQL服务器
``
yum -y install mysql-community-server
``

### 启动MySQL
``
systemctl start  mysqld.service
``
### 查看运行状态
``
systemctl status mysqld.service
``

### 找到MySQL root用户的初始密码：
``
grep "password" /var/log/mysqld.log
``

### 登录，如上8.0安装，授权远程访问，修改密码，不可设置简单密码。




### 索引
索引是为了加速对表中数据行的检索而创建的一种分散存储的数据结构。

* 索引能极大的减少存储引擎需要扫描的数据量
* 索引可以把随机IO变成顺序IO
* 索引可以帮助我们在进行分组、排序等操作时，避免使用临时表。

### B+Tree
