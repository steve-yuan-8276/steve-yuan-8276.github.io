---
title: 'linux运维学习笔记10-3：httpd的基本配置(五)'
date: 2018-06-03 16:05:15
categories: linux
tags: [linux, LAMP, php, notes]
---

## 写在前面

继续学习http配置的知识，主要包括禁止解析php、限制user_agent、php相关配置。

## 禁止解析php

从系统安全的角度，我们不希望用户上传的脚本能够被解析执行，就需要在配置中进行设置。

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/discuz.com"
    ServerName steve-discuz-test.com
    ServerAlias www.steve-aaa.com
    ServerAlias www.steve-bbb.com
    CustomLog "logs/abc.com-access_log" combined
    <Directory /data/discuz.com/data>   //定义目录
       php_admin_flag engine off     //禁止php解析
       <filesmatch "(.*)php">   
            Order deny,allow    
            Deny from all       
            Allow from 127.0.0.1   
        </filesmatch>
    </Directory>
</VirtualHost>
```

<!--more-->

## 限制user_agent

限制user-agent的主要目的是防止网络爬虫，从而节约带宽资源，优化网站体验。

### 全站限制user-agent

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/discuz.com"
    ServerName steve-discuz-test.com
    ServerAlias www.steve-aaa.com
    ServerAlias www.steve-bbb.com
    CustomLog "logs/discuz.com-access_log" combined
    <IfModule mod_rewrite.c>   
        RewriteEngine on
        RewriteCond %{HTTP_USER_AGENT} ^.*curl.* [NC,OR]
        RewriteCond %{HTTP_USER_AGENT} ^.*baidu.com.* [NC]
        RewriteRule .* - [F]
    </IfModule>
</VirtualHost>

```

### 限制某个目录访问

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/discuz.com"
    ServerName steve-discuz-test.com
    ServerAlias www.steve-aaa.com
    ServerAlias www.steve-bbb.com
    CustomLog "logs/discuz.com-access_log" combined
    <IfModule mod_rewrite.c>   
        RewriteEngine on
        RewriteCond %{REQUEST_URI} ^.*/tmp/.* [NC]
        RewriteRule .* - [F]
    </IfModule>
</VirtualHost>

```

## PHP配置

### php配置文件格式：

php的配置文件在`/usr/local/php/etc/php.ini`

配置文件语法：

```
directive = value

```

指令名(directive)是大小写敏感的！所以"foo=bar"不同于"FOO=bar"。

值(value)可以是：

* 用引号界定的字符串(如："foo")；

* 数字(整数或浮点数，如：0, 1, 34, -1, 33.55)；

* PHP常量(如：E_ALL, M_PI)；

* INI常量(On, Off, none)；

* 表达式(如：E_ALL & ~E_NOTICE)；

注释符号为`；`，以` ；`开头的行会被自动忽略。

### 查询php相关信息使用如下命令：

```
/usr/local/php/bin/php -i | head

```

![](https://farm2.staticflickr.com/1752/42531354861_a89a97f7a8_o.png)

**TroubleShooting:**找不到配置文件目录

如下图所示，php的配置文件目录找不到了，显示是none。

![](https://farm2.staticflickr.com/1731/40728706790_d595679b1d_o.png)

解决思路：重新编译安装

```
//重新编译安装
make clean

//编译参数如下：
./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2.4/bin/apxs --with-config-file-path=/usr/local/php/etc/ --enable-fpm --with-fpm-user=nginx --with-fpm-group=nginx --enable-inline-optimization --disable-debug --disable-rpath --enable-shared --enable-soap --with-libxml-dir --with-xmlrpc --with-openssl --with-mhash --with-pcre-regex --with-sqlite3 --with-zlib --enable-bcmath --with-iconv --with-bz2 --enable-calendar --with-curl --with-cdb --enable-dom --enable-exif --enable-fileinfo --enable-filter --with-pcre-dir --enable-ftp --with-gd --with-openssl-dir --with-jpeg-dir --with-png-dir --with-zlib-dir --with-freetype-dir --enable-gd-jis-conv --with-gettext --with-gmp --with-mhash --enable-json --enable-mbstring --enable-mbregex --enable-mbregex-backtrack --with-libmbfl --with-onig --enable-pdo --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-zlib-dir --with-pdo-sqlite --with-readline --enable-session --enable-shmop --enable-simplexml --enable-sockets --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-wddx --with-libxml-dir --with-xsl --enable-zip --enable-mysqlnd-compression-support --with-pear --enable-opcache

//编译安装
make && make install

```

如果过程中报错，很有可能是虚拟内存不足;

```
dd if=/dev/zero of=/swapfile bs=1024 count=262144
mkswap /swapfile
chmod 600 /swapfile
swapon /swapfile
```

增加虚拟内存之后，再来一遍

```
make clean
make && make install
```

重装完成，在检查php虚拟文件，一切正常

![](https://farm2.staticflickr.com/1737/28663980468_e6cc69d057_o.png)

### disable_functions设置

处于安全考虑，禁用一部分function

```
disable_functions = chdir,chroot,dir,getcwd,opendir,readdir,scandir,fopen,unlink,delete,copy,mkdir, 　　rmdir,rename,file,file_get_contents,fputs,fwrite,chgrp,chmod,chown
```

### 配置error_log

一般php在没有连接到数据库或者其他情况下会有提示错误，包含php脚本路径或者查询的SQL语句等，这类信息暴露在网络上是不安全的，所以一般建议禁止错误提示，具体配置如下：

```
//禁止错误提示
display_errors = Off

//定义错误日志
log_errors = On
//定义错误日志路径
error_log = /var/local/php/logs/php_errors.log
error_reporting = E_ALL & ~E_NOTICE

```

设置错误日志保存路径后，还需要修改权限

```
mkdir -p /var/local/php/logs

chmod -R 777 /var/local/php/logs

```

错误日志的级别

标示|说明
---|---
E_ALL        |     所有错误和警告（除E_STRICT外）
E_ERROR      |     致命的错误。脚本的执行被暂停。
E_RECOVERABLE_ERROR   | 大多数的致命错误。
E_WARNING      |   非致命的运行时错误，只是警告，脚本的执行不会停止。
E_PARSE       |     编译时解析错误，解析错误应该只由分析器生成。
E_NOTICE      |    脚本运行时产生的提醒（往往是我们写的脚本里面的一些bug，比如某个变量没有定义），这个错误不会导致任务中断。
E_STRICT        |  脚本运行时产生的提醒信息，会包含一些php抛出的让我们要如何修改的建议信息。
E_CORE_ERROR   |   在php启动后发生的致命性错误
E_CORE_WARNING  |  在php启动后发生的非致命性错误，也就是警告信息
E_COMPILE_ERROR |   php编译时产生的致命性错误
E_COMPILE_WARNING | php编译时产生的警告信息
E_USER_ERROR      | 用户生成的错误
E_USER_WARNING   | 用户生成的警告
E_USER_NOTICE    |  用户生成的提醒

同时，还支持逻辑符号

标示|说明
---|---
& |并且
~ |示非
| |或者

`error_reporting  =  E_ALL & ~E_NOTICE`  表示表示除了警告之外的所有错误日志；
 
### 配置open_basedir

这是一种网络安全策略：主要针对的是同一台服务器上同时有多个网站，防止一个网站被攻击之后殃及其他的网站。

`open_basedir` 将php所能打开的文件限制在指定的目录树中，包括文件本身。当程序要使用例如`fopen()`或`file_get_contents()`打开一个文件时，这个文件的位置将会被检查。当文件在指定的目录树之外，程序将拒绝打开。

配置方法：

#### 方法一：在`php.ini`进行限定

```
vim /usr/local/php/etc/php.ini

//修改以下内容
open_basedir = /tmp:/data/discuz.com
```

这种方法的局限性在于，如果一个服务器上拥有多台虚拟主机，没办法对多个虚拟主机分别进行设置。

方法二：在httpd配置文件进行设置

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    ServerAdmin webmaster@steve-discuz-test.com
    DocumentRoot "/data/discuz.com"   //填写刚才新建的discuz文件目录
    ServerName steve-discuz-test.com   //域名，必须填写
    ServerAlias www.steve-discuz.com   //网站别名
    ErrorLog "logs/steve.com-error_log"
    CustomLog "logs/steve.com-access_log" common
    //配置open——basedir
    php_admin_value open_basedir "/data/discuz.com/:/tmp/"     //仅对本虚拟主机生效
</VirtualHost>

```

## 拓展学习：rewrite 模块

Rewirte主要的功能就是实现URL的跳转，它的正则表达式是基于Perl语言。可基于服务器级的(httpd.conf)和目录级的 (.htaccess)两种方式。如果要想用到rewrite模块，必须先安装或加载rewrite模块。

方法有两种：

* 一是编译apache的时候就直接 安装rewrite模块；

* 二是编译apache时以DSO模式安装apache,然后再利用源码和apxs来安装rewrite模块。

### rewrite 语法：

包括两个部分

#### 一是条件：

```
//语法
RewriteCond TestString CondPattern

//示例
RewriteCond %{HTTP_HOST} ^www.example.net [NC]
```

默认当存在多个条件时，彼此之间是and的关系，rewrite会检测所有条件，只有前一条符合，才会继续检测后一条；如果不符合就会退出；

为此，还有两个重要的参数：

* `nocase|NC` 忽略大小写；

* `ornext|OR` 表示或者，而不是隐含的AND；


#### 二是指令：

```
//语法
RewriteRule Pattern Substitution

//示例
RewriteRule ^(.*)$ http://www.9med.net/$1 [R=permanent,L]
```

常用参数|含义
---|---
R [=code]|强制重定向，默认是302
F|强制URL为被禁止的 forbidden
P|强制代理；要使用这个功能，代理模块必须编译在Apache服务器中。 如果你不能确定，可以检查“httpd -l”的输出中是否有mod_proxy.c。 如果有，则mod_rewrite可以使用这个功能； 如果没有，则必须启用mod_proxy并重新编译“httpd”程序。


