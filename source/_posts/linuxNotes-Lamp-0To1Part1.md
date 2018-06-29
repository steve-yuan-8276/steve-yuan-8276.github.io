---
title: 'linux学习笔记：源码包安装lamp环境'
date: 2018-05-30 11:11:13
categories: linux
tags: [linux, LAMP, php, mysql, notes] 
---


## 写在前面

最近家里事情很多，学习过程也很不顺利，lamp搞得乱七八糟，干脆推倒重来。通过这篇笔记把这周的笔记全部梳理一遍。

这个笔记计划分成两部分：

1. mysql、apache、php安装

2. lamp启动服务

## lamp环境安装

**前提准备：**采用源码包安装的方式，需要用编译器进行。linux下的c语言编译器是gcc，centos下运行 `yum install -y gcc` 进行安装。

**lamp工作流：**

1. 系统防火墙配置

2. mysql、apache、php安装

3. lamp基础配置

废话不多说，直接开始吧。

<!--more-->

### 系统防火墙设置

#### 第一步：关闭firewalld

**注：**因为系统版本centos7，所以关闭默认的防火墙，该用iptables

```
//关闭firewalld
[root@stevey ~]# systemctl stop firewalld.service
//禁止开机启动
[root@stevey ~]# systemctl disable firewalld.service
```

#### 第二步：安装配置iptables


```
//安装iptables
[root@stevey ~]# yum install -y iptables-services
//编辑配置文件，重点是开放80、3306端口
[root@stevey ~]# vim /etc/sysconfig/iptables
			# sample configuration for iptables service
			# you can edit this manually or use system-config-firewall
			# please do not ask us to add additional ports/services to this default configuration
			*filter
			:INPUT ACCEPT [0:0]
			:FORWARD ACCEPT [0:0]
			:OUTPUT ACCEPT [0:0]
			-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
			-A INPUT -p icmp -j ACCEPT
			-A INPUT -i lo -j ACCEPT
			-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
			-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT       //开放80端口，针对httpd
			-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT     //开放3306端口，针对mysql
			-A INPUT -j REJECT --reject-with icmp-host-prohibited
			-A FORWARD -j REJECT --reject-with icmp-host-prohibited
			COMMIT
//重启防火墙使配置生效
[root@stevey ~]# systemctl restart iptables.service
//设置防火墙开机启动		
[root@stevey ~]# systemctl enable iptables.service		
```

### 第三步：关闭selinux

**注：**selinux是centos7一种安全策略，这里选择关闭。


```
[root@stevey ~]# vim /etc/selinux/config

//找到SELINUX=enforcing这一行，将enforcing 改为 disabled，保存退出并重启虚拟机
```

### mysql 安装

mysql是数据库管理软件，目前最新的版本是5.7；因为mysql到源码包很大，编译起来比较费劲，所以这里下载的是已经编译好的二进位制安装包，版本是5.6。

在日常使用中，更多的是使用mysql的开源版本mariaDB，最新的版本是10.2，对应mysql的5.7版本。

#### 第一步：下载源码包

```
//这里使用wget下载工具，如果没有，则执行 yum install -y wget 安装
wget http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz
```

**注：**这个包有300M，我花了一个小时才下载下来，so喝杯咖啡，别着急。


#### 第二步：源码包解压并移动到`/usr/local/mysql`

```
//解压
tar -zvxf mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz

//判断是否已有/usr/local/mysql，如果有则改名，避免影响后面的操作
[ -d /usr/local/mysql ] && mv /usr/local/mysql /usr/local/mysql-old

//移动文件到/usr/local/mysql
mv mysql-5.6.36-linux-glibc2.5-x86_64 /usr/local/mysql

//cd到/usr/local/mysql
cd /usr/local/mysql

```

#### 第三步：创建mysql用户及数据库文件夹

```
//创建mysql用户 -s指定shell
useradd -s /sbin/nologin mysql

//创建数据库文件
mkdir -p /data/mysql

//更改用户名和所属组权限
chown -R mysql:mysql /data/mysql
```

#### 第四步：安装依赖项

```
//安装依赖包perl-Module-Install
yum install -y perl-Module-Install

//安装依赖包libaio
yum install -y libaio
```

#### 第五步：安装数据库

```
//当前目录为/usr/local/mysql
./scripts/mysql_install_db --user=mysql --datadir=/data/mysql
```

#### 第六步：校验是否安装成功

```
[root@stevey mysql]# echo $?
0
```

### apache 安装

**工作流：**

1. 安装依赖包apr

2. 安装依赖包apr-util

3. 安装apache


#### 第一步：下载安装包


```
wget http://mirrors.cnnic.cn/apache/httpd/httpd-2.4.33.tar.gz 

wget http://mirrors.cnnic.cn/apache/apr/apr-1.6.3.tar.gz

wget http://mirrors.cnnic.cn/apache/apr/apr-util-1.6.1.tar.bz2 
```


#### 第二步：安装依赖包apr

```
//解压apr源码包
tar -zvxf apr-1.6.3.tar.gz

//进入tar安装包
cd apr-1.6.3

//配置编辑参数
./configure --prefix=/usr/local/apr

//编译安装
make && make install

//校验是否安装成功
echo $?      //如果输出0，说明安装成功；否则，执行不成功

```

#### 第三步：安装依赖包apr-util

```
//安装bzip2压缩工具
yum install -y bzip2

//安装expat-devel，这是apr-util的依赖包
yum install -y expat-devel

//解压源码包
tar -jvxf apr-util-1.6.1.tar.bz2

//进入安装包目录
cd apr-util-1.6.1

//配置编辑参数
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr

//编译安装
make && make install

//校验是否安装成功
echo $?      //如果输出0，说明安装成功；否则，执行不成功
```

#### 第四步：安装apache

```
//解压源码包
tar -zvxf httpd-2.4.33.tar.gz

//进入安装文件夹
cd httpd-2.4.33

//安装依赖项pcre-devel
yum install -y pcre-devel

//编译参数
[root@stevey httpd-2.4.33]# ./configure \
> --prefix=/usr/local/apache2.4 \
> --with-apr=/usr/local/apr \
> --with-apr-util=/usr/local/apr-util \
> --enable-so \
> --enable-mods-shared=most

//编译安装
make && make install
```

#### 第五步：校验是否安装成功

```
[root@stevey mysql]# echo $?
0
```

### php安装

**工作流：**

1. 下载源码包

2. 安装依赖包

3. 编译安装php源码包

#### 第一步：下载源码包

```
[root@steve src]# wget http://cn2.php.net/distributions/php-7.2.6.tar.gz

//解压源码包
[root@steve src]# tar -zxvf php-7.2.6.tar.gz

//进入安装目录
[root@steve src]# cd php-7.2.6

```

#### 第二步：安装依赖包

```
[root@steve src]# yum install -y libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel

```

#### 第三步：编译参数

```
./configure \
--prefix=/usr/local/php \
--with-apxs2=/usr/local/apache2.4/bin/apxs \
--with-config-file-path=/usr/local/php/etc \
--enable-fpm \
--with-fpm-user=nginx  \
--with-fpm-group=nginx \
--enable-inline-optimization \
--disable-debug \
--disable-rpath \
--enable-shared  \
--enable-soap \
--with-libxml-dir \
--with-xmlrpc \
--with-openssl \
--with-mhash \
--with-pcre-regex \
--with-sqlite3 \
--with-zlib \
--enable-bcmath \
--with-iconv \
--with-bz2 \
--enable-calendar \
--with-curl \
--with-cdb \
--enable-dom \
--enable-exif \
--enable-fileinfo \
--enable-filter \
--with-pcre-dir \
--enable-ftp \
--with-gd \
--with-openssl-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib-dir  \
--with-freetype-dir \
--enable-gd-jis-conv \
--with-gettext \
--with-gmp \
--with-mhash \
--enable-json \
--enable-mbstring \
--enable-mbregex \
--enable-mbregex-backtrack \
--with-libmbfl \
--with-onig \
--enable-pdo \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-zlib-dir \
--with-pdo-sqlite \
--with-readline \
--enable-session \
--enable-shmop \
--enable-simplexml \
--enable-sockets  \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-wddx \
--with-libxml-dir \
--with-xsl \
--enable-zip \
--enable-mysqlnd-compression-support \
--with-pear \
--enable-opcache
```

#### 第四步：编译安装

```
make && make install 
```

##### TroubleShooting: 虚拟内存不足

![](https://farm2.staticflickr.com/1723/42447649381_d171bd86b3_o.png)

**解决方法： 手动增加swap**

- 建立swapfile

```
[root@stevey php-7.2.6]# mkdir -p /usr/img/
[root@stevey php-7.2.6]# rm -rf /usr/img/swap
[root@stevey php-7.2.6]# dd if=/dev/zero of=/usr/img/swap bs=1024 count=2048000
2048000+0 records in
2048000+0 records out
2097152000 bytes (2.1 GB) copied, 7.43002 s, 282 MB/s

```

- 格式化swap

```
[root@stevey php-7.2.6]# mkswap /usr/img/swap
Setting up swapspace version 1, size = 2047996 KiB
no label, UUID=db4f95a9-d67c-4d8c-b32f-a5f07ab4f051
```

- 挂载swap

```
[root@stevey php-7.2.6]# swapon /usr/img/swap
swapon: /usr/img/swap: insecure permissions 0644, 0600 suggested.
```

- 修改文件夹权限

```
chmod 0600 /usr/img/swap
```

- 增加swap空间后，再次执行编译安装

```
make && make install 
```

#### 第五步：校验安装是否成功

```
[root@stevey php-7.2.6]# echo $?
0
```

### 环境变量问题

至此，mysql、apache、php安装部分结束。不过，这里还有一个重要的问题没有解决。因为我们在采用源码包安装时自定义了安装目录，而不是linux系统的默认目录，所以如果这时查找命令是找不到的，就像这样。

```
[root@stevey init.d]# mysql -v
-bash: mysql: command not found

```

解决方案：

#### 方法一：修改环境变量

系统环境变量的配置文件在`/etc/profile`（全局修改）或者`~/.bash_profile`（只影响当前用户），直接修改配置文件。

PATH声明用法：

```
PATH=$PAHT:<PATH 1>:<PATH 2>:<PATH 3>:< PATH n > //变量之间用逗号隔开
export PATH

```

示例：

```
vim ～/.bash_profile     //加入以下内容

PATH=$PATH:$HOME/bin:/usr/local/mysql/bin:/usr/local/apache2.4/bin:/usr/local/php/bin

//保存退出执行以下命令生效
source ～/.bash_profile
```

验证一下

```
[root@stevey php]# php -v
PHP 7.2.6 (cli) (built: May 30 2018 14:37:03) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
[root@stevey php]# httpd -v
Server version: Apache/2.4.33 (Unix)
Server built:   May 30 2018 13:54:01

```


#### 第二种方法：软链接

如果我们不想修改环境变量，也可以采取软链接的办法。

比如：

```
ln -s /usr/local/mysql/bin/ /usr/bin
```

验证一下，也是可以的。

```
[root@stevey ~]# mysql -V
mysql  Ver 14.14 Distrib 5.6.36, for linux-glibc2.5 (x86_64) using  EditLine wrapper

```

-------

## lamp环境启动

### mysql启动服务

#### 第一步：配置mysql

```
//进入mysql 目录
[root@stevey src]# cd /usr/local/mysql

//复制配置文件
cp support-files/my-default.cnf /etc/my.cnf

//编辑配置文件
vim /etc/my.cnf

//修改以下配置
log_bin = root    //15行
basedir = /usr/local/mysql
datadir = /data/mysql
port = 3306
server_id = 128
socket = /tmp/mysql.sock

//去掉以下配置前面的的型号
innodb_buffer_pool_size = 128M     
join_buffer_size = 128M
sort_buffer_size = 2M
read_rnd_buffer_size = 2M

```

#### 第二步：配置mysql启动脚本

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

#### 第三步：设置开机启动

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

#### 第四步：校验是否启动

```
//方法一：查看进程，结果应大于2行
[root@stevey init.d]# ps -aux |grep  mysqld  
root      84371  0.0  0.1  11816  1620 pts/1    S    15:06   0:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/data/mysql --pid-file=/data/mysql/stevey.pid
mysql     84607  1.2 44.9 1300832 449812 pts/1  Sl   15:06   0:01 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/data/mysql --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/data/mysql/stevey.err --pid-file=/data/mysql/stevey.pid --socket=/tmp/mysql.sock --port=3306
root      84633  0.0  0.0 112704   976 pts/1    S+   15:07   0:00 grep --color=auto mysqld

方法二：查看端口监听情况
[root@stevey init.d]# netstat -lnp |grep mysqld
tcp6       0      0 :::3306                 :::*                    LISTEN      84607/mysqld
unix  2      [ ACC ]     STREAM     LISTENING     169619   84607/mysqld         /tmp/mysql.sock
```

至此，这一部分就结束了，后面的内容在接着整理。

