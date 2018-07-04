---
title: 'linux学习笔记12-1：lnmp环境的搭建和配置（一）'
date: 2018-06-04 17:02:42
categories: linux
tags: [linux, LNMP, notes]
---

## 写在前面

从这则笔记开始整理lnmp环境的搭建和配置，首先是三大件（mariadb、nginx、php）的安装。

## LNMP简介

### LNMP架构是什么？

所谓LNMP，就是Linux系统下Nginx+MySQL+PHP这种网站服务器架构。

与LAMP不同，LNMP的工作模式：

* 在LAMP中，提供web服务的是apache；而在LNMP中，提供web服务的是Nginx；

* 在Apache中，PHP是作为一个模块存在的；在Nginx中，PHP是作为一个独立服务存在的，这个服务叫做php-fpm；

* Nginx直接处理静态请求（支持的并发更高，速度比Apache快），动态请求转发给php-fpm处理。

![](https://farm2.staticflickr.com/1756/28740520778_5576c34786_o.jpg)

<!--more-->

#### nginx 的优缺点：



* 优点：占用VPS资源较少，Nginx配置起来也比较简单，利用fast-cgi的方式动态解析PHP脚本。

* 缺点：php-fpm组件的负载能力有限，在访问量巨大的时候，php-fpm进程容易僵死，容易发生502 bad gateway错误

#### php-fpm又是什么？

这里涉及到到一些新的概念，整理如下：

* `CGI`，即公共网关接口`Common Gateway Interface`，简单地说，就是网页的表单和你写的程序之间通信的一种协议。CGI针对每个http请求都是fork一个新进程来进行处理，处理过程包括解析php.ini文件，初始化执行环境等，然后这个进程会把处理完的数据返回给web服务器，最后web服务器把内容发送给用户，刚才fork的进程也随之退出。

* `FastCGI`，是CGI的更高级的一种方式，是用来提高CGI程序性能的。Fastcgi则会先fork一个master，解析配置文件，初始化执行环境，然后再fork多个worker。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复劳动，提高了工作效率。

* `PHP-CGI`，是PHP自带的FastCGI管理器。只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理。所以，就出现了很多进程调度的工具，比如spawn-fcgi。

* `PHP-FPM`，是PHP FastCGI管理器，之前是PHP源代码的一个补丁，旨在将FastCGI进程管理整合进PHP包中。从PHP5.3.3版本开始，PHP内核集成了PHP-FPM之后就方便多了，使用--enalbe-fpm这个编译参数安装即可。


## LNMP环境前期准备

### Linux系统环境

* LNMP需要预留5G以上的剩余空间；

* 安装MySQL 5.6或5.7及MariaDB 10必须1G以上内存；

* 安装vim `yum install -y vim-enhanced`

* 提前安装 好GCC编译器 yum install -y gcc

* 系统防火墙设置，关闭firewalld；安装并配置好itables，开放80、3306端口；同时，selinux设置为disabled

### 软件环境：

* MariaDB 10.2.6 64  [下载地址](https://downloads.mariadb.com/MariaDB/mariadb-10.2.6/bintar-linux-glibc_214-x86_64/mariadb-10.2.6-linux-glibc_214-x86_64.tar.gz)
 
* Nginx 1.14 [下载地址](https://nginx.org/download/nginx-1.14.0.tar.gz)

* PHP 7.2.6 [下载地址](http://cn2.php.net/distributions/php-7.2.6.tar.gz)

#### nginx 依赖的环境包：

* gcc、gcc+ 安装方法：`yum install -y gcc gcc-c++`

* PCRE库 安装方法：`yum install -y pcre pcre-devel`

* zlib库 安装方法：`yum install -y zlib zlib-devel`

* OpenSSL开发库（支持https） 安装方法：`yum install -y openssl openssl-devel`

#### php 依赖的环境包：

* `libssl-dev` 安装方法：`yum install libssl-dev`

* `libxml2-dev` 安装方法：`yum install libxml2-dev`

* `libcurl4-gnutls-dev` 安装方法：`yum install libcurl4-gnutls-dev`

## LNMP环境安装

### MariaDB安装

因为之前安装lamp环境，已经折腾过好几遍mysql了，所以这次安装开源的mariaDB试试。

centos7本身自带了mariaDB5.5版，我们需要先把这个卸载掉，之后再安装最新的MariaDB10.2版。

实现步骤：

#### 第一步：备份数据库

```
cp -a /var/lib/mysql/ /var/lib/mysql.bak
```

#### 第二步：添加MariaDB存储库

按照mariaDB官方给的方法，修改yum配置文件

![](https://farm1.staticflickr.com/975/41220755955_9d3f8a4cd6_o.png)

```
//升级yum
yum update

//编辑yum源
vim /etc/yum.repos.d/MariaDB10.repo

//加入以下内容：
# MariaDB 10.2 CentOS repository list - created 2018-05-15 06:14 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.2/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1

//先清除之前到缓存
yum clean all  

//生成缓存   
yum makecache     
         
```

![](https://farm2.staticflickr.com/1738/42614821661_f21309d9bb_o.png)

#### 第三步：删除系统自带的MariaDB 5.5

```
yum remove mariadb-server mariadb mariadb-libs
```

#### 第四步：安装MariaDB 10

```
yum install -y MariaDB-server MariaDB-client
```

**注：**mariaDB有170多M，但是是从国外的服务器下载，所以超级慢，此处可以休息休息。

#### 第五步：启动服务

```
//启动服务
systemctl start mariadb

//设置开机启动
systemctl enable mariadb

//检查服务状态
systemctl status mariadb

```

#### 第六步：初始化密码设定

```
mysql_secure_installation
```

根据提示，设置以下内容：

* 设置root密码

* 是否禁止远程 root访问

* 是否禁止 test数据库的访问

* 是否禁用匿名用户

* 是否重新加载privilleges-table信息

#### 第七步：创建用户以及设置权限


```
//数据库登陆
mysql -u[用户名，默认是root] -p[密码，输入刚刚初始化设定的密码]
```

![](https://farm2.staticflickr.com/1757/41733408005_59844f596b_o.png)


#### 第八步：测试是否启动

```
//检测进程是否启动
[root@steve ~]# ps -aux | grep mysqld
root      5460  0.0  0.0 112704   976 pts/1    R+   13:28   0:00 grep --color=auto mysqld
mysql    30064  0.0  4.1 1307692 78408 ?       Ssl  11:27   0:02 /usr/sbin/mysqld

//查看端口情况
[root@steve ~]# netstat -lnp | grep 3306
tcp6       0      0 :::3306                 :::*                    LISTEN      30064/mysqld

```

### Nginx安装

#### 第一步：下载源码包

```
//默认源码包都放到/usr/local/src目录
cd /usr/local/src

//下载
wget https://nginx.org/download/nginx-1.14.0.tar.gz

//解压
tar -zxvf nginx-1.14.0.tar.gz
```

#### 第二步：安装依赖包

* gcc、gcc+ 安装方法：`yum install -y gcc gcc-c++`

* PCRE库 安装方法：`yum install -y pcre pcre-devel`

* zlib库 安装方法：`yum install -y zlib zlib-devel`

* OpenSSL开发库（支持https） 安装方法：`yum install -y openssl openssl-devel`

#### 第三步：创建用户及安装目录

```
//创建安装目录
mkdir -p /usr/local/nginx

//创建用户
groupadd  nginx
useradd -M -s /sbin/nologin -g nginx nginx
```

#### 第四步：编译安装

```
//进入源码包
cd nginx-1.14.0

//编译参数
./configure \
  --prefix=/usr/local/nginx \
  --pid-path=/usr/local/nginx/logs/nginx.pid \
  --lock-path=/var/lock/nginx.lock \
  --user=nginx \
  --group=nginx \
  --with-http_ssl_module \
  --with-http_flv_module \
  --with-http_stub_status_module \
  --with-http_gzip_static_module \
  --with-pcre
```

![](https://farm2.staticflickr.com/1760/41915883234_4d38e74207_o.png)

```
//编译安装
make && make install

```

#### 第五步：启动nginx

```
//设置软链接
[root@steve ~]# ln -s /usr/local/nginx/sbin/nginx /usr/sbin

//语法检查
[root@steve ~]# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

//启动nginx
nginx
```

##### TroubleShooting

无法启动nigix服务

![](https://farm2.staticflickr.com/1736/41916951404_6e0902f0e5_o.png)

解决方法：

此错误表示80端口已经被别的进程占用，把这个进程干掉，重新启动服务即可。

```
//结束进程
[root@steve ~]# sudo fuser -k 80/tcp
80/tcp:               5379  5380

//如果出现提示错误： fuser command not found，则安装psmisc
[root@stevey ~]# yum install -y psmisc

//启动进程
[root@steve ~]# nginx

//查看进程情况
[root@steve ~]# ps -aux | grep nginx
root      5499  0.0  0.0  45928  1120 ?        Ss   13:39   0:00 nginx: master process nginx
nginx     5500  0.0  0.1  46376  1896 ?        S    13:39   0:00 nginx: worker process
root      5508  0.0  0.0 112704   976 pts/1    R+   13:39   0:00 grep --color=auto nginx

//查看端口情况
[root@steve ~]# netstat -lnp | grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5499/nginx: master
```

#### 第六步：编写启动脚本并加入系统服务

- 编写启动脚本

```
vi /etc/init.d/nginx

//加入以下内容
#!/bin/bash
# chkconfig: - 30 21
# description: http service.
# Source Function Library
. /etc/init.d/functions
# Nginx Settings

NGINX_SBIN="/usr/local/nginx/sbin/nginx"
NGINX_CONF="/usr/local/nginx/conf/nginx.conf"
NGINX_PID="/usr/local/nginx/logs/nginx.pid"
RETVAL=0
prog="Nginx"

start() {
        echo -n $"Starting $prog: "
        mkdir -p /dev/shm/nginx_temp
        daemon $NGINX_SBIN -c $NGINX_CONF
        RETVAL=$?
        echo
        return $RETVAL
}

stop() {
        echo -n $"Stopping $prog: "
        killproc -p $NGINX_PID $NGINX_SBIN -TERM
        rm -rf /dev/shm/nginx_temp
        RETVAL=$?
        echo
        return $RETVAL
}

reload(){
        echo -n $"Reloading $prog: "
        killproc -p $NGINX_PID $NGINX_SBIN -HUP
        RETVAL=$?
        echo
        return $RETVAL
}

restart(){
        stop
        start
}

configtest(){
    $NGINX_SBIN -c $NGINX_CONF -t
    return 0
}

case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  reload)
        reload
        ;;
  restart)
        restart
        ;;
  configtest)
        configtest
        ;;
  *)
        echo $"Usage: $0 {start|stop|reload|restart|configtest}"
        RETVAL=1
esac
exit $RETVAL

```

这里有一个小窍门，直接复制粘贴代码是乱的，这是由于vim自动缩紧造成的。

**解决办法：**打开文件，在普通模式下输入 `:set paste` ,之后输入`i `进入编辑模式，再复制格式就不会自动缩紧了；复制完成后，按esc退出编辑模式，再输入 `:set nopaste` 之后保存退出即可。

- 更改脚本权限

```
chmod a+x /etc/init.d/nginx
```

- 启动脚本

```
chkconfig --add nginx
chkconfig nginx on
```


### PHP 安装

这里还是安装php7.2.6版本，源码包地址：http://cn2.php.net/distributions/php-7.2.6.tar.gz

安装步骤如下：

#### 第一步：下载并解压源码包

```
//wget命令下载
[root@steve src]# wget http://cn2.php.net/distributions/php-7.2.6.tar.gz

//解压源码包
[root@steve src]# tar -zxvf php-7.2.6.tar.gz
```

#### 第二步：安装依赖包

```
//进入安装目录
[root@steve src]# cd php-7.2.6

//安装依赖包
[root@steve src]# yum install -y libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel
```

#### 第三步：创建账号

**注：**该账号用来运行php-fpm服务，在LNMP环境中，PHP是以一种服务的形式独立存在的。

```
[root@steve php-7.2.6]# useradd -s /sbin/nologin php-fpm
```

#### 第四步：编译安装

编译参数（此处markdown解析又出现莫名其妙的错误，只能截图）

![](https://farm2.staticflickr.com/1732/40829420100_fcdd90c673_o.png)

参数如下：

```
./configure --prefix=/usr/local/php-fpm --with-config-file-path=/usr/local/php-fpm/etc --with-fpm-user=php-fpm --with-fpm-group=php-fpm --enable-mysqlnd --with-mysqli --with-pdo-mysql --enable-fpm --with-gd --with-iconv --with-zlib --enable-xml --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --enable-mbstring --enable-ftp --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-curl --with-jpeg-dir --with-freetype-dir --enable-opcache
```

    
编译安装

```
make && make install
```
  

![](https://farm2.staticflickr.com/1756/40828251110_51decc6ce0_o.png)


#### 第五步：修改 php配置文件

查看php信息，可以发现此时php是没有配置文件的

```
/usr/local/php-fpm/bin/php -i | head
```

![](https://farm2.staticflickr.com/1742/28763296468_7b5dcebd78_o.png)


```
//移动php.ini文件
[root@steve php-7.2.6]# mv php.ini-production /usr/local/php-fpm/etc/php.ini

//进入配置文件
cd /usr/local/php-fpm/etc

```

此时查看以下配置目录

![](https://farm2.staticflickr.com/1743/42585641712_7d53e548b3_o.png)


```
//重命名配置文件
mv php-fpm.conf.default ./php-fpm.conf

//修改配置文件
vi php-fpm.conf

//加入以下内容
[global]
pid = /usr/local/php-fpm/var/run/php-fpm.pid
error_log = /usr/local/php-fpm/var/log/php-fpm.log
[www]
listen = /tmp/php-fcgi.sock
listen.mode = 666
user = php-fpm
group = php-fpm
pm = dynamic
pm.max_children = 50
pm.start_servers = 20
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500
rlimit_files = 1024

```

保存退出后检查语法错误

![](https://farm2.staticflickr.com/1745/42636936091_ee79c55fa1_o.png)

解决方法：直接把140行用； 注释掉就好了。

重新检查语法错误

```
[root@steve etc]# /usr/local/php-fpm/sbin/php-fpm -t
[07-Jun-2018 15:47:59] NOTICE: configuration file /usr/local/php-fpm/etc/php-fpm.conf test is successful
```

#### 第六步：设置开机启动

源码包提供了一个现成的启动脚本，位置在`/usr/local/src/php-7.2.6/sapi/fpm/init.d.php-fpm`

实现步骤：

```
//复制启动脚本
cp /usr/local/src/php-7.2.6/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm

//更改权限
chmod 755 /etc/init.d/php-fpm

//启动服务
service php-fpm start

//开机启动
chkconfig php-fpm on
```

检测服务是否已经启动

![](https://farm2.staticflickr.com/1744/41918910804_84b793c187_o.png)

注：最后，修改环境变量或者软链接，即可直接调用命令；否则，需要使用绝对路径执行。

```
[root@steve ~]# vi .bash_profile

//加入以下变量，变量之间用英文冒号隔开
/usr/local/php-fpm/bin:/usr/local/php-fpm/sbin

[root@steve ~]# source !$
source .bash_profile
```


## 参考资料

* [十分钟搞懂什么是CGI](https://www.cnblogs.com/xueweihan/p/5319893.html)

* [FastCgi与PHP-fpm之间是个什么样的关系](https://segmentfault.com/q/1010000000256516)

* [如何通俗地解释 CGI、FastCGI、php-fpm 之间的关系](https://www.zhihu.com/question/30672017)

* [LAMP与LNMP架构的区别及其具体的选择说明](http://blog.sina.com.cn/s/blog_626998030102wofn.html)

* [Mysql的基础使用之MariaDB安装方法详解](https://www.teakki.com/p/57e2271aa16367940da623d8)

* [LNMP架构介绍、安装php-fpm](https://my.oschina.net/u/3497124/blog/1505447)

* [CentOS安装和设置MariaDB的教程](https://www.jb51.net/article/72428.htm)

