---
title: 'linux运维学习笔记9-3：php安装'
date: 2018-05-25 11:04:49
categories: linux
tags: [linux, php,notes] 
---

## 写在前面

这则笔记的目的也很简单，就是安装php，按照课程要求，分别安装php5.5和php7两个版本。为了防止相互影响，分别在两台机器上实验。

## 源码包安装

### `php7` 安装

php7.2源码包地址：http://cn2.php.net/distributions/php-7.2.6.tar.gz

![](https://farm1.staticflickr.com/956/41613747194_d6667b7403_o.png)

<!--more-->

#### 第一步：下载并解压源码包

首先，还是来规矩，把源码包下载到/usr/local/src目录下，方便统一管理。当然，我还机智滴给这个路径设置了一个alias别名，这样又可以少敲几个按键了。

```
//wget命令下载
[root@steve src]# wget http://cn2.php.net/distributions/php-7.2.6.tar.gz

//解压源码包
[root@steve src]# tar -zxvf php-7.2.6.tar.gz
```

#### 第二步：安装依赖包

先看了一下INSTALL和README两个说明文档，但是好像也没什么用，直接开始编译安装吧。


```
//进入安装目录
[root@steve src]# cd php-7.2.6

//安装依赖包
[root@steve src]# yum install -y libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel

```

#### 第三步：编译参数


```
./configure \
--prefix=/usr/local/php \
--with-config-file-path=/etc \
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

![](https://farm1.staticflickr.com/953/40531118980_2557b00482_o.png)

ok,编译完成

#### 第四步：编译安装


```
[root@steve php-7.2.6]# make && make install 
```

##### 第五步：配置php

```
vim /etc/profile   //修改系统变量配置文件，加入以下内容

PATH=$PATH:/usr/local/php/bin
export PATH
```

保存退出后，执行命令生效

```
source /etc/profile
```

### 安装php5.5

#### 第一步：下载源码包

```
//下载源码包
[root@steve src] wget http://cn2.php.net/distributions/php-5.6.32.tar.bz2

//解压
[root@steve src] tar -jxvf php-5.6.32.tar.bz2
```

#### 第二步：安装依赖

```
yum install -y autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libpng libpng-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses curl openssl-devel gdbm-devel db4-devel libXpm-devel libX11-devel gd-devel gmp-devel readline-devel libxslt-devel expat-devel xmlrpc-c xmlrpc-c-devel

```

#### 第三步：编译参数

```
//进入源码包
cd php-5.6

//编译参数

./configure --prefix=/usr/local/php-5.6.0 
--with-mysql=/usr/local/mysql \
--with-mysql-sock  \
--with-mysqli=\usr\local\mysql\bin\mysql_config  \
--enable-fpm  \
--with-ncurses  \
--enable-soap  \
--with-libxml-dir  \
--with-XMLrpc  \
--with-openssl  \
--with-mcrypt  \
--with-mhash  \
--with-pcre-regex  \
--with-sqlite3  \
--with-zlib  \
--enable-bcmath  \
--with-iconv  \
--with-bz2  \
--enable-calendar  \
--with-curl  \
--with-cdb  \
--enable-dom  \
--enable-exif  \
--enable-fileinfo  \
--enable-filter  \
--with-pcre-dir  \
--enable-ftp  \
--with-gd  \
--with-openssl-dir  \
--with-jpeg-dir  \
--with-png-dir  \
--with-zlib-dir   \
--with-freetype-dir  \
--enable-gd-native-ttf  \
--enable-gd-jis-conv  \
--with-gettext  \
--with-gmp  \
--with-mhash  \
--enable-json  \
--enable-mbstring  \
--disable-mbregex  \
--disable-mbregex-backtrack  \
--with-libmbfl  \
--with-onig  \
--enable-pdo  \
--with-pdo-mysql  \
--with-zlib-dir  \
--with-pdo-sqlite  \
--with-readline  \
--enable-session  \
--enable-shmop  \
--enable-simplexml  \
--enable-sockets  \
--enable-sqlite-utf8  \
--enable-sysvmsg  \
--enable-sysvsem  \
--enable-sysvshm  \
--enable-wddx  \
--with-libxml-dir   \
--with-xsl  \
--enable-zip  \
--enable-mysqlnd-compression-support  \
--with-pear 

```

#### 第四步：编译安装

```
make && make install
```



## yum安装php

源码包安装太繁琐了，在网上找到使用yum安装到方法，一起测试一下。

### 安装流程


#### 第一步：安装php7的源

```
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

yum install https://centos7.iuscommunity.org/ius-release.rpm
```

#### 第二步：搜索新版 PHP

```
yum search php70u

```

#### 第三步： 安装php

```
yum install -y php70u-common php70u-fpm php70u-cli php70u-gd php70u-mysqlnd php70u-pdo php70u-mcrypt php70u-mbstring php70u-json php70u-opcache php70u-xml

```

#### 第四步：检查php版本

```
php -v
```

![](https://farm1.staticflickr.com/981/41616433524_9069c97606_o.png)

话说，被源码包蹂躏了半天，能够用yum安装，简直是神清气爽啊。

不过，为了继续做实验，我们还要把这个卸载掉。

### 卸载流程

#### 第一步：查看已安装PHP 相关的包

```
yum list installed | grep php
```

![](https://farm2.staticflickr.com/1753/41437103985_29aaedafb6_o.png)

第二步：卸载包

```
yum uninstall php70u-cli.x86_64  php70u-common.x86_64 php70u-fpm.x86_64 php70u-gd.x86_64    php70u-json.x86_64 php70u-mbstring.x86_64 php70u-mcrypt.x86_64 php70u-mysqlnd.x86_64 php70u-opcache.x86_64 php70u-pdo.x86_64   php70u-xml.x86_64

```

## 参考资料：

[CentOS 7 安装 PHP 7](https://www.jianshu.com/p/6d3b688cd0be)

[CentOS：删除旧版 PHP，安装新版 PHP](https://talk.ninghao.net/t/centos-php-php/4841)

