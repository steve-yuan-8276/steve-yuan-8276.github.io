---
title: 'linux运维学习笔记10-1：httpd的基本配置(三)'
date: 2018-05-30 23:01:47
categories: linux
tags: [linux, LAMP, php, notes]
---

## 写在前面

这则笔记紧接上一则笔记内容，主要整理日志管理的两个小技巧，以及静态元素过期时间设置。

## 日志管理的两个小技巧

### 技巧一：访问日志不记录指定类型的静态文件

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

//仍然对www.abc.com进行修改

<VirtualHost *:80>
    DocumentRoot "/data/wwwroot/www.abc.com"
    ServerName www.abc.com
    ServerAlias abc.com
    <IfModule mod_rewrite.c>           
        RewriteEngine on  
        RewriteCond %{HTTP_HOST} !^www.abc.com$   
        RewriteRule ^/(.*)$ http://www.abc.com/$1 [R=301,L]   
    </IfModule>
    //增加以下内容
    SetEnvIf Request_URI ".*\.gif$" img-request
    SetEnvIf Request_URI ".*\.jpg$" img-request
    SetEnvIf Request_URI ".*\.png$" img-request
    SetEnvIf Request_URI ".*\.bmp$" img-request
    SetEnvIf Request_URI ".*\.swf$" img-request
    SetEnvIf Request_URI ".*\.js$" img-request
    SetEnvIf Request_URI ".*\.css$" img-request
    CustomLog "logs/111.com-access_log" common env=!img-request
</VirtualHost>

```

<!--more-->

**注：此处更改主要包括两个部分：**

* `SetEnvIf` 语句，先建立img-request，把gif、jpg、png、bmp、js、css等文件全部归到img-request；

* 第二部分定义日志名，这里使用！取反，将这些img-request都排除在外，相当于不记录这些信息。

配置修改完成后检查语法错误，重新加载即生效，不再赘述。


### 技巧二：访问日志切割


```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

在不记录静态文件的配置语句后面再加上一条
CustomLog "|/usr/local/apache2.4/bin/rotatelogs -l logs/abc.com-access_%Y%m%d.log 86400" combined env=!img-request
```

注：这里使用了管道符，将文件交由rotatelogs命令按照时间单位进行切割，86400单位是秒，相当于一天保存一个文档；

## 静态元素过期时间

### 第一步：编辑配置文件

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

//增加配置
<IfModule mod_expires.c>
    ExpiresActive on  //打开该功能的开关
    ExpiresByType image/gif  "access plus 1 days"
    ExpiresByType image/jpeg "access plus 24 hours"
    ExpiresByType image/png "access plus 24 hours"
    ExpiresByType text/css "now plus 2 hour"
    ExpiresByType application/x-javascript "now plus 2 hours"
    ExpiresByType application/javascript "now plus 2 hours"
    ExpiresByType application/x-shockwave-flash "now plus 2 hours"
    ExpiresDefault "now plus 0 min"
</IfModule>
```

### 第二步：修改apache主配置文件，加载expires模块

```
vim /usr/local/apache2.4/conf/httpd.conf

//取消注释
LoadModule expires_module modules/mod_expires.so
```

### 第三步：重新加载模块

```
//检查语法错误
[root@stevey ~]# /usr/local/apache2.4/bin/apachectl -t
Syntax OK

//重新加载配置文件
[root@stevey ~]# /usr/local/apache2.4/bin/apachectl graceful

//查看是否加载模块
[root@gary-tao local]# /usr/local/apache2.4/bin/apachectl -M|grep -i rewrite  
 rewrite_module (shared)
```


