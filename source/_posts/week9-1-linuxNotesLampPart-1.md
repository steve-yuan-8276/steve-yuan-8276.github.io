---
title: 'Linux运维学习笔记9-1：mysql安装'
date: 2018-05-15 10:02:13
categories: linux
tags: [linux, apache, mysql, php,notes] 
---

### 写在前面

LAMP，是linux、apache、mysql、php的简写，也就是把apache、mysql、php安装在linux系统上，共同构成一个运行php语言的网络环境。

### mysql 

#### 什么是数据库？

数据库是一种用于存储数据集合的独立应用程序。每种数据库都会有一个或多个独特的 API，用来创建、访问、管理、搜索或复制数据库中保存的数据。

现在业界一般采用关系型数据库管理系统（RDBMS）来存储并管理海量数据。之所以称其为关系型数据库，是因为所有数据都存储在不同的表中，表之间的关系是建立在主键或其他键（被称为外键）的基础之上的。

关系型数据库管理系统（RDBMS）具有以下特点：

* 能够实现一种具有表、列与索引的数据库；

* 保证不同表的行之间的引用完整性；

* 能自动更新索引；

* 能解释 SQL 查询，组合多张表的信息；

<!--more-->

#### 什么是mysql？

MySQL是一个开放源代码的关系数据库管理系统（DBMS），由于性能高、成本低、可靠性好，已经成为最流行的开源数据库，因此被广泛地应用在Internet上的中小型网站中。

MySQL 的特点：

* 使用C和C++编写，并使用了多种编译器进行测试，保证源代码的可移植性；

* 支持linux、macos、windows等系统，跨平台；

* 为多种編程语言提供了API，包括php、python、perl等等；

* 多语言支持，常见的编码如中文的GB 2312、BIG5等等；

#### MySQL安装

##### 下载rpm包

**rpm包地址：**

* [5.1 32位二进制包：](http://mirrors.sohu.com/mysql/MySQL-5.1/mysql-5.1.73-linux-i686-glibc23.tar.gz) 

* [5.1_64位二进制包：](http://mirrors.sohu.com/mysql/MySQL-5.1/mysql-5.1.73-linux-x86_64-glibc23.tar.gz)

* [5.5_64位二进制包：](http://mirrors.sohu.com/mysql/MySQL-5.5/mysql-5.5.55-linux2.6-x86_64.tar.gz ) 

* [5.5_32位二进制包：](http://mirrors.sohu.com/mysql/MySQL-5.5/mysql-5.5.55-linux2.6-i686.tar.gz)

* [5.6_32位二进制包：](http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.36-linux-glibc2.5-i686.tar.gz)

* [5.6_64位二进制包：](http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz)

* [5.5源码包：](http://mirrors.sohu.com/mysql/MySQL-5.5/mysql-5.5.55.tar.gz)

* [5.6源码包：](http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.36.tar.gz )


- 查询系统版本

```
[root@stevey ~]# uname -i
x86_64
```

- 下载对应的rpm包（这里写在的已经编译好的二进位制包）

```
wget http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz

```

- 解压压缩包

```
tar -zvxf mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz
```

- 将文件包移动到`/usr/local/mysql`

```
//判断是否已有/usr/local/mysql，如果有则改名，避免影响后面的操作
[ -d /usr/local/mysql ] && mv /usr/local/mysql /usr/local/mysql-old

//移动文件到/usr/local/mysql
mv mysql-5.6.36-linux-glibc2.5-x86_64 /usr/local/mysql

//cd到/usr/local/mysql
cd /usr/local/mysql

```

- 创建mysql用户及数据库文件夹

```
创建mysql用户
useradd -s /sbin/nologin mysql

创建数据库文件
mkdir -p /data/mysql

更改用户名和所属组权限
chown -R mysql:mysql /data/mysql
```

- 安装mysql

```
//安装依赖包，注意perl-Module-Install写法
yum install -y perl-Module-Install


//安装数据库
./scripts/mysql_install_db --user=mysql --datadir=/data/mysql

```

这里遇到了一个错误提示：

![](https://farm1.staticflickr.com/950/41394400745_36bb3ecb1c_o.png)

google了一下[，stackoverflow 给出了答案](https://stackoverflow.com/questions/10619298/libaio-so-1-cannot-open-shared-object-file)：

![](https://farm1.staticflickr.com/945/28423177778_a5d635e53f_o.png)

执行命令，成功安装：

```
yum install -y libaio
```

#### 配置mysql

- 复制配置文件

```
[root@stevey mysql]# cp support-files/my-default.cnf /etc/my.cnf
cp: overwrite ‘/etc/my.cnf’? y     //选择y
```

- 编辑配置文件

```
vim /etc/my.cnf
```

- 按照下面的设置进行配置

![](https://farm1.staticflickr.com/944/42073060032_08e0e24289_o.png)

- 配置启动脚本

```
//复制启动脚本到/etc/init.d/mysqld
cp support-files/mysql.server /etc/init.d/mysqld

//修改文件权限
chmod 755 /etc/init.d/mysqld

//编辑启动脚本
vim /etc/init.d/mysqld

//更改data保存目录
basedir=/usr/local/mysql
datadir=/data/mysql

```

- 设置开机启动


```
//进入系统服务目录 /etc/init.d/
cd /etc/init.d/

//将mysqld 服务加入系统服务列表
chkconfig --add mysqld      

//设置开机启动
chkconfig mysqld on

//启动mysqld
service mysqld start

```

判断服务是否启动

```
//查看进程，结果应大于两行
[root@steve ~]# ps -aux |grep  mysqld    
root     14858  0.0  0.0  11772  1592 ?        S    May23   0:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/data/mysql --pid-file=/data/mysql/steve.pid
mysql    15094  0.0 23.9 1301736 450716 ?      Sl   May23   0:29 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/data/mysql --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/steve.err --pid-file=/data/mysql/steve.pid --socket=/tmp/mysql.sock --port=3306
root     18381  0.0  0.0 112660   968 pts/0    R+   21:26   0:00 grep --color=auto mysqld

//查看端口监听情况
[root@steve ~]# netstat -lnp |grep mysqld
tcp6       0      0 :::3306                 :::*                    LISTEN      15094/mysqld
unix  2      [ ACC ]     STREAM     LISTENING     195472   15094/mysqld         /tmp/mysql.sock
```

##### TroubleShooting

###### 问题1:

But，这里有一个很奇怪的现象,貌似这个mysql是inactive的状态，这是什么意思呢？

```
[root@steve ~]# systemctl is-active mysqld
inactive
```

**[更新]**

找到问题答案了，重启服务即可。


```
systemctl restart mysql.service

```

再次测试，齐活儿！

![](https://farm1.staticflickr.com/967/28450554038_cf96ecbcc8_o.png)

###### 问题2:

试试查看系统版本，这又是什么鬼？

```
[root@steve ~]# mysql -V
-bash: mysql: command not found

```

结合前面的安装过程，我们知道在源码包安装是自定义的安装目录，所以怀疑是环境变量的问题，在网上google一下，果然也有人这么说？那怎么办呢？

设置一个软链接吧

```
ln -s /usr/local/mysql/bin/mysql  /usr/bin
```

再试一下 `mysql -v` 查看系统版本，齐活儿。

![](https://farm1.staticflickr.com/890/42276343402_12accde21c_o.png)

### 拓展：MariaDB

MariaDB的是甲骨文收购MySQL的项目的一个替代方案，免费。其安装步骤如下：

#### 第一步：备份数据库

```

//备份数据库
cp -a /var/lib/mysql/ /var/lib/mysql.bak

```

#### 第2步：添加MariaDB存储库

- 升级yum 源

![](https://farm1.staticflickr.com/982/27427337257_412e3cc51e_o.png)

- 编辑配置文件

```
vim /etc/yum.repos.d/MariaDB10.repo

//加入以下内容：
# MariaDB 10.2 CentOS repository list - created 2018-05-15 06:14 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```


注：该配置文件[来自这里](![](https://farm1.staticflickr.com/975/41220755955_9d3f8a4cd6_o.png))，选择需要的版本即可，如下图所示：

![](https://farm1.staticflickr.com/975/41220755955_9d3f8a4cd6_o.png)

#### 第三步：删除MariaDB 5.5

```
yum remove mariadb-server mariadb mariadb-libs
```

#### 第四步：安装MariaDB 10

```
yum -y install MariaDB-server MariaDB-client
```

#### 第五步：mariadb开启关闭

```
//启动服务
systemctl start mariadb

//设置开机启动
systemctl enable mariadb

//停止MariaDB
systemctl stop mariadb 

//重启MariaDB
systemctl restart mariadb  

//升级MariaDB
mysql_upgrade

//查看mariadb版本号
mysql -V

```

#### 第六步：mariadb基本操作

```
//修改root的密码
update mysql.user set password=PASSWORD('abc') where user='root';
// 更新权限
flush privileges; 

//新建用户
// 格式为： create user  '用户名'@'主机' identified by '密码'   
create user 'read_visa'@'%' identified by '123456'；  

//用户分配权限
// grant 操作类型 on 数据库.表 to 用户@'主机'   
grant all on visa.* to 'read_visa'@'%'；    // all 表示所有权限
grant select on visa.* to 'read_visa'@'%';

```


### 参考资料：

* [如何在CentOS / RHEL 7和Debian系统上将MariaDB 5.5升级到MariaDB 10.1](https://www.howtoing.com/upgrade-mariadb-5-5-to-10-centos-rhel-debian-ubuntu/)

* [aminglinux培训资源](https://coding.net/u/aminglinux/p/resource/git/blob/master/README.md?public=true)

* [MySQL 安装](http://wiki.jikexueyuan.com/project/mysql/installation.html)

* [什么是MySQL](https://github.com/jaywcjlove/mysql-tutorial/blob/master/chapter1/1.3.md)


