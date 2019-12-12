## 一、查看Linux系统版本的命令

### 1、cat /etc/redhat-release，这种方法只适合Redhat系的Linux：
```
[root@VM_0_16_centos ~]# cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core) 
```

### 2、lsb_release -a，即可列出所有版本信息：
```
[root@VM_0_16_centos ~]# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch:cxx-4.1-amd64:cxx-4.1-noarch:desktop-4.1-amd64:desktop-4.1-noarch:languages-4.1-amd64:languages-4.1-noarch:printing-4.1-amd64:printing-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.6.1810 (Core) 
Release:	7.6.1810
Codename:	Core
```
如果提示找不到lsb_release，使用如下命令安装：
```
yum install redhat-lsb -y
```

## 二、查看Linux内核版本命令

### 1、cat /proc/version
```
[root@VM_0_16_centos ~]# cat /proc/version
Linux version 3.10.0-957.21.3.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) 
(gcc version 4.8.5 20150623 (Red Hat 4.8.5-36) (GCC) ) #1 SMP Tue Jun 18 16:35:19 UTC 2019
```

### 2、uname -a
```
[root@VM_0_16_centos ~]# uname -a
Linux VM_0_16_centos 3.10.0-957.21.3.el7.x86_64 #1 SMP Tue Jun 18 16:35:19 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```