---
title: 'linux运维学习笔记9-4：httpd的基本配置(一)'
date: 2018-05-28 16:23:52
categories: linux
tags: [linux, LAMP, php,notes] 
---

## 写在前面

LAMP是由一系列操作系统/开发软件名称首字母集合而成的一种说法，它包括Linux、Apache、MySQL、PHP，通过这一系列组合而成的开发环境能够让你顺利地开发Web应用。

这则笔记继续整理LAMP环境的基本配置相关知识。

## 配置httpd支持PHP

### 编辑httpd配置文件

httpd 的主配置文件为 /usr/local/apache2.4/conf/httpd.conf,编辑该文件并进行修改。

```
vim /usr/local/apache2.4/conf/httpd.conf
```

修改以下几处：

- 去掉servername前面的#号注释

![](https://farm2.staticflickr.com/1757/41682258574_d7eeb0dcf4_o.png)

<!--more-->

- `denied` 改为 `granted`，允许所有请求；否则访问会报错

![](https://farm2.staticflickr.com/1727/42355315452_4cae40a125_o.png)

- 新增两行 `AddType application/x-httpd-php .php` 、`AddType application/x-httpd-php-source  .phps`

![](https://farm2.staticflickr.com/1729/42355348932_531f84d34a_o.png)

- 在`index，html` 后面增加 `index.php`

![](https://farm2.staticflickr.com/1724/28532079978_c8c0442aab_o.png)

之后保存退出。

### 启动httpd服务

```
//检查配置文件是否正确
[root@stevey ~]# /usr/local/apache2.4/bin/apachectl -t      //命令要写绝对路径
Syntax OK         //说明配置正确

//启动httpd
[root@stevey ~]# /usr/local/apache2.4/bin/apachectl start

//查看httpd是否启动
[root@stevey ~]# netstat -lnp | grep httpd
tcp6       0      0 :::80                   :::*                    LISTEN      14882/httpd       //说明正常启动

//也可以通过curl命令进行检测
[root@stevey ~]# curl localhost
<html><body><h1>It works!</h1></body></html>
```

或者也要在浏览器输入虚拟机网址

![](https://farm2.staticflickr.com/1753/41725827034_c7f020b4fa_o.png)

### 检查http是否支持php

```
//新建一个测试文件，并添加内容
vim /usr/local/apache2.4/htdocs/test.php

<?php
    echo 'php格式能够被正确解析'
?>
```

#### TroubleShooting ：php未能正确解析，404错误

解决方案：

网上查了一下，说是因为php7安装的时候忘记了配置`--with-apxs2=/usr/local/apache2.4/bin/apxs ` ，只能重新编译安装

```
//编译参数

[root@stevey php-7.2.6]# ./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2.4/bin/apxs --with-config-file-path=/etc --enable-fpm --with-fpm-user=nginx  --with-fpm-group=nginx --enable-inline-optimization --disable-debug --disable-rpath --enable-shared  --enable-soap --with-libxml-dir --with-xmlrpc --with-openssl --with-mhash --with-pcre-regex --with-sqlite3 --with-zlib --enable-bcmath --with-iconv --with-bz2 --enable-calendar --with-curl --with-cdb --enable-dom --enable-exif --enable-fileinfo --enable-filter --with-pcre-dir --enable-ftp --with-gd --with-openssl-dir --with-jpeg-dir --with-png-dir --with-zlib-dir  --with-freetype-dir --enable-gd-jis-conv --with-gettext --with-gmp --with-mhash --enable-json --enable-mbstring --enable-mbregex --enable-mbregex-backtrack --with-libmbfl --with-onig --enable-pdo --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-zlib-dir --with-pdo-sqlite --with-pdo-sqlite --enable-session --enable-shmop --enable-simplexml --enable-sockets  --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-wddx --with-libxml-dir --with-xsl --enable-zip --enable-mysqlnd-compression-support --with-pear --enable-opcache


```

重新编译安装，又遇到错误提示：

```
make: *** [sapi/cli/php] Error 1
```

解决方案：


```
make clean 

make && make install

```

## Apache默认虚拟主机

### 预备动作

编辑http主配置文件，并删除vhost这一行的注释#

![](https://farm2.staticflickr.com/1753/40600481960_8926de1eec_o.png)

### 编辑虚拟机配置文件 http/-vhosts.conf 

```
[root@stevey ~]# vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

```

![](https://farm2.staticflickr.com/1726/42405954121_559de966ba_o.png)

目录|注释
---|---
ServerAdmin|管理员邮箱
DocumentRoot|虚拟主机站点的根目录，网站的程序就放在该目录下
ServerName|网站域名，只能写一个
ServerAlias|网站别名，可以写很多个，中间用空格分开
ErrorLog|错误日志
CustomLog|站点访问日志

```
// 编辑虚拟机配置文件
[root@stevey ~]# vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

//修改一下内容
  <VirtualHost *:80>
      ServerAdmin webmaster@steve.com
      DocumentRoot "/data/wwwroot/steve.com"
      ServerName steve.com
      ServerAlias www.steve.com
      ErrorLog "logs/steve.com-error_log"
      CustomLog "logs/steve.com-access_log" common
  </VirtualHost>

  <VirtualHost *:>
      DocumentRoot "/data/wwwroot/www.abc.com"
      ServerName www.abc.com
  </VirtualHost>
```

通过上述设置，steve.com 这个域名就成为了默认虚拟主机，如果有第三个域名同样指向该主机，默认会访问steve.com

```
//文件准备 
[root@stevey ~]# mkdir -p /data/wwwroot/steve.com /data/wwwroot/www.abc.com
[root@stevey ~]# echo 'steve.com' > /data/wwwroot/steve.com/index.html
[root@stevey ~]# echo 'abc.com' > /data/wwwroot/www.abc.com/index.com
```

### 重新加载apache服务

```
//语法检查
[root@stevey ~]# /usr/local/apache2.4/bin/apachectl -t

//重新加载httpd服务
[root@stevey ~]# /usr/local/apache2.4/bin/apachectl graceful

```

#### TroubleShooting:403拒绝访问

![](https://farm2.staticflickr.com/1742/28551736278_ba3960391a_o.png)

**解决方案：**

查了一下，发现是把安装目录写错了，修改后即可正常访问

```
DocumentRoot "/data/wwwroot/steve.com"
```

### 校验配置成果

#### 测试虚拟主机

```
[root@stevey www.abc.com]# curl -x127.0.0.1:80 steve.com
steve.com
[root@stevey www.abc.com]# curl -x127.0.0.1:80 www.abc.com
abc.com
//注：将第三个域名指向该服务器时，就会访问默认主机域名
[root@stevey www.abc.com]# curl -x127.0.0.1:80 www.123.com
steve.com

```


#### 测试php是否能正确解析

- 文件准备：

```
[root@stevey steve.com]# vim test.php

<?php
    echo 'php格式能够被正确解析'
?>
```

- curl命令测试

```
[root@stevey steve.com]# curl -x127.0.0.1:80 steve.com/test.php
php格式能够被正确解析

```

- 浏览器测试

![](https://farm2.staticflickr.com/1735/40642555020_9c1e0b35b7_o.png)

