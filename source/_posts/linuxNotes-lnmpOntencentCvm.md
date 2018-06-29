---
title: 'linux笔记：在腾讯云主机配置lnmp环境'
date: 2018-06-15 14:22:03
categories: linux
tags: [linux, mysql, notes]
---

## 写在前面

实践出真知，在腾讯云主机实际搭建lnmp环境。

## 系统环境准备

### 必装软件：

* vim    `yum install -y vim-enchanced`

* tree   `yum install -y tree`

* wget   `yum install -y wget`

* gcc    `yum install -y gcc gcc+`

### 语言环境变量设置

```
vim /etc/envirment

//增加两行
LANG=en_US.utf-8
LC_ALL=en_US.utf-8
```

### 关闭firewalld

```
//关闭firewalld
systemctl stop firewalld.service
//禁止开机启动
systemctl disable firewalld.service
```

### 安装配置iptables

```
//安装iptables
yum install -y iptables-services
//编辑配置文件，重点是开放80、3306端口
vim /etc/sysconfig/iptables
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

<!--more-->

### 关闭selinux

```
vim /etc/selinux/config

//找到SELINUX=enforcing这一行，将enforcing 改为 disabled，保存退出并重启虚拟机
```

### 阶段成果

![](https://farm2.staticflickr.com/1738/42091924874_2d5e7089f0_o.png)

## 软件环境安装

### mysql 安装

#### 第一步：下载二进位制包

```
wget http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz

//解压
tar -zvxf mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz

//进入安装包
cd mysql-5.6.36-linux-glibc2.5-x86_64

```

#### 第二步：创建mysql账户及目录

```
创建mysql安装目录
mkdir -p /usr/local/mysql

//创建mysql用户 -s指定shell
useradd -s /sbin/nologin mysql

//创建数据库文件
mkdir -p /data/mysql

//更改用户名和所属组权限
chown -R mysql:mysql /data/mysql
```

#### 第三步：安装依赖项

```
//安装依赖包perl-Module-Install
yum install -y perl-Module-Install

//安装依赖包libaio
yum install -y libaio
```

#### 第四步：编译安装

```
//将编译好的二进位制包移动到/usr/local/mysql目录下
mv /usr/local/src/mysql-5.6.36-linux-glibc2.5-x86_64/*  /usr/local/mysql/

//进入mysql目录
cd /usr/local/mysql/

//编译参数
./scripts/mysql_install_db --user=mysql --datadir=/data/mysql

//校验结果
[root@VM_0_14_centos mysql]# echo $?
0      //返回0说明成功
```

#### 第五步：修改环境变量

```
//修改配置文件
vim ~/.bash_profile

//将mysql路径加入环境变量
PATH=$PATH:$HOME/bin:/usr/local/mysql/bin
```

![](https://farm2.staticflickr.com/1738/40999972160_91230fe703_o.png)

#### 第六步：配置mysql

```
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

#### 第七步：配置启动脚本

```
cd /usr/local/mysql

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

#### 第八步：设置开机启动

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

#### 校验是否启动

- 方法一：查看进程，结果应大于2行

```
ps -aux |grep  mysqld 
``` 

![](https://farm1.staticflickr.com/895/42810933531_e2dfd8a056_o.png)


方法二：查看端口监听情况

```
netstat -lnp |grep mysqld
```

![](https://farm2.staticflickr.com/1754/42762119442_ee0a3ea3a1_o.png)

TroubleShooting：mysqld服务未启动

![](https://farm2.staticflickr.com/1788/42871890012_9a799a526c_o.png)

重启服务即可

![](https://farm2.staticflickr.com/1826/29048510808_40fdf645a9_o.png)

### nginx 安装

#### 第一步：下载源码包

```
//源码包放到/usr/local/src目录
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
  
//编译安装
make && make install
```

#### 第五步：启动nginx

```
//设置软链接
ln -s /usr/local/nginx/sbin/nginx /usr/sbin

//语法检查
[root@VM_0_14_centos nginx-1.14.0]# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

//启动nginx
nginx
```

校验成果

```
[root@VM_0_14_centos nginx-1.14.0]# ps -aux | grep nginx
root     12551  0.0  0.1  45928  1120 ?        Ss   17:09   0:00 nginx: master process nginx
nginx    12552  0.0  0.1  48460  1976 ?        S    17:09   0:00 nginx: worker process
root     12557  0.0  0.0 112704   976 pts/3    R+   17:09   0:00 grep --color=auto nginx
[root@VM_0_14_centos nginx-1.14.0]# netstat -lnp | grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      12551/nginx: master
```

#### 第六步：编写启动脚本并加入系统服务

- 编写启动脚本

```
vim /etc/init.d/nginx

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

- 更改脚本权限

```
chmod a+x /etc/init.d/nginx
```

- 启动脚本

```
chkconfig --add nginx
chkconfig nginx on
```

**注：**这里遇到一个很奇怪的错误，nginx启动失败；`restart`也不行，后来尝试`init 6` 重启主机后成功。

测试结果如下，nginx成功启动

![](https://farm1.staticflickr.com/884/28938073238_e6cffef511_o.png)

### php安装

#### 第一步：下载并解压源码包

```
cd /usr/local/src/

//wget命令下载
wget http://cn2.php.net/distributions/php-7.2.6.tar.gz

//解压源码包
tar -zxvf php-7.2.6.tar.gz
```

#### 第二步：安装依赖包

```
//安装依赖包
yum install -y libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel
```

#### 第三步：创建账号

```
useradd -s /sbin/nologin php-fpm
```

#### 第四步：编译安装

```
//进入安装目录
cd /usr/local/src/php-7.2.6

//编译参数
./configure --prefix=/usr/local/php-fpm --with-config-file-path=/usr/local/php-fpm/etc --with-fpm-user=php-fpm --with-fpm-group=php-fpm --enable-mysqlnd --with-mysqli --with-pdo-mysql --enable-fpm --with-gd --with-iconv --with-zlib --enable-xml --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --enable-mbstring --enable-ftp --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-curl --with-jpeg-dir --with-freetype-dir --enable-opcache

//编译安装
make && make install
```

##### TroubleShooting：虚拟内存不足导致make失败

因为php的安装包比较大，因此经常会在make的时候出现虚拟内存不够报错的情况。笔者在本地虚拟机和云主机的配置都是1G内存，也都遇到了这个情况，解决的办法是手动增加swap交换空间。

![](https://farm2.staticflickr.com/1756/42212015644_7be9599ca2_o.png)

手动增加swap空间的步骤：

```
//建立swapfile
[root@tencentcvm php-7.2.6]#  mkdir -p /usr/img/
[root@tencentcvm php-7.2.6]# rm -rf /usr/img/swap
[root@tencentcvm php-7.2.6]# dd if=/dev/zero of=/usr/img/swap bs=1024 count=2048000
2048000+0 records in
2048000+0 records out
2097152000 bytes (2.1 GB) copied, 19.5733 s, 107 MB/s

//格式化swap
[root@tencentcvm php-7.2.6]# mkswap /usr/img/swap
Setting up swapspace version 1, size = 2047996 KiB
no label, UUID=0d8a402b-11d0-4e0d-afa0-05c002b3df07

//挂载swap
[root@tencentcvm php-7.2.6]# swapon /usr/img/swap
swapon: /usr/img/swap: insecure permissions 0644, 0600 suggested.

修改文件夹权限
[root@tencentcvm php-7.2.6]# chmod 0600 /usr/img/swap
```

修改完成后，在重新执行make

```
make clean

make && make install
```

注：安装完成别忘记修改环境变量,将`/usr/local/php-fpm/sbin` 和 `/usr/local/php-fpm/bin` 加入环境变量。

这两个目录下分别有这些命令：
![](https://farm2.staticflickr.com/1802/41120332280_1883e71697_o.png)

```
vi .bash_profile

//加入以下变量，变量之间用英文冒号隔开
/usr/local/php-fpm/bin:/usr/local/php-fpm/sbin

[root@steve ~]# source !$
source .bash_profile
```

#### 第五步：修改 php配置文件

```
//查看php配置信息
php -i | head

//移动php.ini文件
[root@steve php-7.2.6]# mv php.ini-production /usr/local/php-fpm/etc/php.ini

//进入配置文件
cd /usr/local/php-fpm/etc

//重命名配置文件
cp php-fpm.conf.default ./php-fpm.conf

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

//检查语法错误
php-fpm -t
```

**注意：**这里有一个提示：

![](https://farm2.staticflickr.com/1731/42212410964_aca9ee2d54_o.png)

之所以出现这一行，是因为下面红框中的内容，即`include=/usr/local/php-fpm/etc/php-fpm.d/*.conf`。 lnmp环境支持虚拟主机，也就是同一台服务器上运行多个网站，每个网站的配置信息，联通php的配置文件都可以分开写，`/usr/local/php-fpm/etc/php-fpm.d/`目录下就是关于php-fpm的配置文件。


#### 第六步：设置开机启动

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


**至此，安装部分暂告一段落，下面是配置部分**

-------


## nginx 配置

### nginx 的基础配置

nginx的配置文件在 /usr/local/nginx/conf/nginx.conf

第一步：清空配置文件

```
[root@tencentcvm ~]# > /usr/local/nginx/conf/nginx.conf
```

第二步：编辑基础配置

```
vi /usr/local/nginx/conf/nginx.conf

//加入以下内容
user nginx nginx;
worker_processes 2;
error_log /usr/local/nginx/logs/nginx_error.log crit;
pid /usr/local/nginx/logs/nginx.pid;
worker_rlimit_nofile 65535;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
               '$status $body_bytes_sent "$http_referer" '
               '"$http_user_agent" "$http_x_forwarded_for"';
    server_names_hash_bucket_size 3526;
    server_names_hash_max_size 4096;
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 65;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_max_body_size 8m;
    server_tokens off;
    client_body_buffer_size 512k;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    fastcgi_intercept_errors on;
    gzip off;
    gzip_min_length 1k;
    gzip_buffers 32 4k;
    gzip_http_version 1.0;
    gzip_comp_level 5;
    gzip_types text/css text/xml application/javascript application/atom+xml application/rss+xml text/plain application/json;
    gzip_vary on;

    server
    {
        listen       80;
        server_name  localhost;
        index index.html index.htm index.php;
        root /usr/local/nginx/html;
        #charset koi8-r;

        location ~ \.php$
        {
            include fastcgi_params;
            fastcgi_pass   unix:/tmp/php-fcgi.sock;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME /usr/local/nginx/html$fastcgi_script_name;
        }
    } 
}

```

第三步：检查语法并重新加载服务


```
//语法检查
[root@tencentcvm ~]# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful

//重新加载服务
[root@tencentcvm ~]# nginx -s reload
```


### 配置虚拟主机

同一台服务器，可以设置多个网站，每个网站采用单独的配置文件；

第一步：编辑配置文件

```
vi /usr/local/nginx/conf/nginx.conf

//在最后一个大括号之前加一行
include vhost/*.conf;
```


第二步：新建虚拟主机配置文件夹

```
//新建虚拟主机文件夹
mkdir -p /usr/local/nginx/conf/vhost //该目录下所有配置文件均会被加载

cd /usr/local/nginx/conf/vhost
```

