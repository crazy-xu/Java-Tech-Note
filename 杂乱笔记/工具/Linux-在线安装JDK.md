### 查询版本号
```
[root@VM_0_16_centos soft]# java -version
java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
```

## 在线安装JDK
### 下载JDK
貌似有有效期，可以去oracle官网找新的链接，[JDK1.8下载链接](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
```
wget https://download.oracle.com/otn/java/jdk/8u231-b11/5b13a193868b4bf28bcb45c792fce896/jdk-8u231-linux-x64.rpm?AuthParam=1576118819_a6ab195df9fe06c872ba61907bd57ef4
```

### RPM 安装
```
rpm -ivh jdk-8u231-linux-x64.rpm
```

## 在线安装OPEN-JDK
### 查看yum库中的Java安装包 yum -y list java*
```
[root@VM_0_16_centos soft]# yum -y list java*
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
Available Packages
java-1.6.0-openjdk.x86_64                                                                                                     1:1.6.0.41-1.13.13.1.el7_3                                                                                  os     
java-1.6.0-openjdk-demo.x86_64                                                                                                1:1.6.0.41-1.13.13.1.el7_3                                                                                  os     
java-1.6.0-openjdk-devel.x86_64                                                                                               1:1.6.0.41-1.13.13.1.el7_3                                                                                  os     
java-1.6.0-openjdk-javadoc.x86_64                                                                                             1:1.6.0.41-1.13.13.1.el7_3                                                                                  os     
java-1.6.0-openjdk-src.x86_64                                                                                                 1:1.6.0.41-1.13.13.1.el7_3                                                                                  os     
java-1.7.0-openjdk.x86_64                                                                                                     1:1.7.0.241-2.6.20.0.el7_7                                                                                  updates
java-1.7.0-openjdk-accessibility.x86_64                                                                                       1:1.7.0.241-2.6.20.0.el7_7                                                                                  updates
java-1.7.0-openjdk-demo.x86_64                                                                                                1:1.7.0.241-2.6.20.0.el7_7                                                                                  updates
java-1.7.0-openjdk-devel.x86_64                                                                                               1:1.7.0.241-2.6.20.0.el7_7                                                                                  updates
java-1.7.0-openjdk-headless.x86_64                                                                                            1:1.7.0.241-2.6.20.0.el7_7                                                                                  updates
java-1.7.0-openjdk-javadoc.noarch                                                                                             1:1.7.0.241-2.6.20.0.el7_7                                                                                  updates
java-1.7.0-openjdk-src.x86_64                                                                                                 1:1.7.0.241-2.6.20.0.el7_7                                                                                  updates
java-1.8.0-openjdk.i686                                                                                                       1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk.x86_64                                                                                                     1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-accessibility.i686                                                                                         1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-accessibility.x86_64                                                                                       1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-accessibility-debug.i686                                                                                   1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-accessibility-debug.x86_64                                                                                 1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-debug.i686                                                                                                 1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-debug.x86_64                                                                                               1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-demo.i686                                                                                                  1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-demo.x86_64                                                                                                1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-demo-debug.i686                                                                                            1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-demo-debug.x86_64                                                                                          1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-devel.i686                                                                                                 1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-devel.x86_64                                                                                               1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-devel-debug.i686                                                                                           1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-devel-debug.x86_64                                                                                         1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-headless.i686                                                                                              1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-headless.x86_64                                                                                            1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-headless-debug.i686                                                                                        1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-headless-debug.x86_64                                                                                      1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-javadoc.noarch                                                                                             1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-javadoc-debug.noarch                                                                                       1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-javadoc-zip.noarch                                                                                         1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-javadoc-zip-debug.noarch                                                                                   1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-src.i686                                                                                                   1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-src.x86_64                                                                                                 1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-src-debug.i686                                                                                             1:1.8.0.232.b09-0.el7_7                                                                                     updates
java-1.8.0-openjdk-src-debug.x86_64                                                                                           1:1.8.0.232.b09-0.el7_7                                                                                     updates
```

### RMP 安装
```
yum -y install java-1.8.0-openjdk*
```