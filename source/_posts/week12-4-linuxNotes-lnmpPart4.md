---
title: 'linux学习笔记12-4：lnmp环境的搭建和配置（四）'
date: 2018-06-08 13:41:37
categories: linux
tags: [linux, LNMP, notes]
---


## 写在前面

继续整理lnmp环境的常见配置，今天主要整理的是nginx防盗链、访问控制、解析php相关配置、Nginx代理等知识。


## nginx中location的基本语法

### 语法

命令行格式：

```
location [=|~|~*|^~] patt {

}
```

常用符号含义：

* `= `开头表示精确匹配；

* `^~ `开头表示uri以某个常规字符串开头，不是正则匹配；

* `~ `开头表示区分大小写的正则匹配；

* `~*` 开头表示不区分大小写的正则匹配；

* `!~` 区分大小写不匹配；

* `!~*`不区分大小写不匹配的正则；

* `/ `通用匹配，任何请求都会匹配到。

### 匹配顺序

* 精确匹配 location = /abc { }

* 匹配路径的前缀，如果找到停止搜索 location ^~ /abc { }

* 不区分大小写的正则匹配 location ~* /abc { }

* 正则匹配 location ~ /abc { }

* 普通路径前缀匹配 location /abc { }

<!--more-->

举例说明：

#### 第一类：精准匹配

```

location  = / {
  # 只匹配"/".
  [ configuration A ] 

```

#### 第二类：一般匹配

```
ocation ^~ /images/ {
  # 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条。
  [ configuration D ]
}

location /documents/ {
  # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索
  # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
  [ configuration C ]
}

location  / {
  # 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求
  # 但是正则和最长字符串会优先匹配
  [ configuration B ]
}

```

#### 第三类：正则匹配

```
location ~* \.(gif|jpg|jpeg)$ {
  # 匹配所有以 gif,jpg或jpeg 结尾的请求
  # 然而，所有请求 /images/ 下的图片会被 config D 处理，因为 ^~ 到达不了这一条正则
  [ configuration E ]
}
```

## Nginx防盗链

防盗链的目的在于节约服务器资源，防止揩油行为。

### 第一步：编辑配置文件

vim /usr/local/nginx/conf/vhost/test.com.conf 

```
//增加如下配置：

location ~* ^.+\.(gif|jpg|png|swf|flv|rar|zip|doc|pdf|gz|bz2|jpeg|bmp|xls)$
{
    expires 7d;
    valid_referers none blocked server_names  *.test.com ; //定义白名单
    if ($invalid_referer) {
        return 403;
    } //如果不是白名单里就返回403
    access_log off;
}
```

### 第二步：重新加载 nginx 服务

```
nginx -t

nginx -s reload
```

### 第三步：测试效果

**注：**这里有个问题，为什么定义的是403，返回的是404呢？

```
steve:steveBlog steveyuan$ curl -e "http://www.test.com/1.txt" -x 172.16.155.128:80 -I test.com/1.gif
HTTP/1.1 404 Not Found
Server: nginx
Date: Mon, 11 Jun 2018 05:37:47 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive


steve:steveBlog steveyuan$ curl -e "http://www.test.com/" -x 172.16.155.128:80 -I test.com/1.gif
HTTP/1.1 404 Not Found
Server: nginx
Date: Mon, 11 Jun 2018 05:39:34 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive

```


## Nginx访问控制

### 第一步：编辑配置文件（根据实际情况，白名单或黑名单任选其一）

- 配置白名单

```
vi /usr/local/nginx/conf/vhost/test.com.conf 

//增加白名单，只允许以下几个ip访问

location /admin/
{
    allow 127.0.0.1;
    deny all;
}
```

- 配置黑名单

```
vi /usr/local/nginx/conf/vhost/test.com.conf 

//增加白名单，只允许以下几个ip访问

location /admin/
{
    deny 172.16.157.0/24;    //禁止一个网段
}

```

- 针对user-agent 进行限制

```
vi /usr/local/nginx/conf/vhost/test.com.conf 

//增加白名单，只允许以下几个ip访问
//这里制定目录，该配置针对全站生效

if ($http_user_agent ~ 'spider|romato')
{
    return 403;    //等同于deny all
}

```

**注：**在配置httpd的时候，需先定义order顺序；在Nginx里并没有这个选项，它就会从上到下逐一去匹配，一旦找到匹配项，就会做deny 处理，返回403错误。

### 第二步：重新加载 nginx 服务

```
nginx -t

nginx -s reload
```

### 第三步：测试效果

```
//白名单测试(虚拟机本机访问)
[root@stevey admin]# curl -x 127.0.0.1:80 test.com/admin/test.html
123,ok

//白名单测试（宿主机访问，此ip不在白名单上）
steve:steveBlog steveyuan$ curl -x 172.16.155.128:80 -I test.com/admin/test.html
HTTP/1.1 403 Forbidden
Server: nginx
Date: Mon, 11 Jun 2018 06:09:38 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive

//黑名单测试
[root@stevey admin]# curl -x 127.0.0.1:80 test.com/admin/test.html
123,ok

//限制user-agent访问的测试
steve:steveBlog steveyuan$ curl -A 'spider' -x 172.16.155.128:80 test.com/admin/test.html
<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx</center>
</body>

```

#### 目录访问限制

除了对全站进行配置，还可以对指定的目录进行配置，主要用到location语句，配置语句类似于

```
location ～ [目录]
{
    配置规则
}

```


示例：

**目的：**upload目录下的.php文件不能访问，但是除了.php的其他后缀文件就能访问。

```
vim /usr/local/nginx/conf/vhost/test.com.conf

//增加如下内容：

location ~ .*(upload|image)/.*\.php$  //意思是匹配upload或者image目录下的.php文件
{
        deny all;
}
```


## Nginx解析php

在LAMP中，PHP是作为httpd的一个模块出现的；而在LNMP中，PHP是以一个服务（php-fpm）的形式存在的，首先要启动php-fpm服务，然后Nginx再和php-fpm通信。也就是说，处理PHP脚本解析的工作是由php-fpm处理完成后把结果传递给Nginx，Nginx再把结果返回给用户。

### 第一步：修改配置文件


```
vim /usr/local/nginx/conf/vhost/test.com.conf

//增加配置如下：

location ~ \.php$
    {
        include fastcgi_params;
        fastcgi_pass unix:/tmp/php-fcgi.sock; 
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /data/nginx/test.com$fastcgi_script_name;
    }

```

**说明：**

* fastcgi_pas用来指定php-fpm的地址,如果php-fpm监听的是127.0.0.1:9000,那么这里也要改成相同地地址，否则会报502错误。

* factcgi_parm SCRIPT_FILENAME后面的路径为该站点根目录,与之前root定义的路径一致，否则会报404错误。

#### 502错误常见原因：

- 一是配置错误：nginx找不到php-fpm了，所以报错；

**正确做法：**astcgi_pass后面可以是socket或者是ip:port

```
fastcgi_pass unix:/tmp/php-fcgi.sock; 
//或者
fastcgi_pass 127.0.0.1:9000;
```

- 二是资源耗尽：nmp架构在处理php时，nginx直接调取后端的php-fpm服务，如果nginx的请求量偏高，又没有给php-fpm配置足够的子进程，那么一旦资源耗尽，就会出现502错误，

**正确做法：**调整php-fpm.conf中的pm.max_children数值，一般4G内存机器可以设置为150，8G为300以此类推；

### 第二步：重新加载 nginx 服务

```
nginx -t

nginx -s reload
```

### 第三步：准备测试文件

```
//进入文件目录
cd /data/nginx/test.com

//建立测试php网页
touch 1.php

//编辑测试网页，加入以下内容
<?php
    echo 'this is a php test';
?>
```

### 第四步：测试结果

```
steve:steveBlog steveyuan$ curl -x 172.16.155.128:80 test.com/1.php
this is a php
```


## Nginx代理

ipV4时代，公网ip资源稀缺，假设公司有多台服务器，但只有一个ip，就可以通过代理的方式，让没有公网ip的服务器提供web服务。

### 第一步：修改配置文件

```
cd /usr/local/nginx/conf/vhost

vim proxy.conf

//增加如下内容：

server
{
    listen 80;
    server_name test.com;

    location /
    {
        proxy_pass      http://172.16.155.128/; //指定要代理的域名所在的服务器IP，即Web服务器的地址
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

注：该配置文件没有设置root文件目录的路径，因为它是代理服务器，不需要访问本地服务器上的任何文件；


### 第二步：重新加载 nginx 服务

```
nginx -t

nginx -s reload
```


## 拓展知识：

### Rebot协议：

robots.txt（统一小写）是一种存放于网站根目录下的ASCII编码的文本文件，它不是一个正式规范，而只是约定俗成的君子协定，会告诉网络搜索引擎的漫游器（又称网络蜘蛛），此网站中的哪些内容是不应被搜索引擎的漫游器获取的，哪些是可以被漫游器获取的。

语法：

```
//允许所有的机器人：
User-agent: *
Allow:/

//仅允许特定的机器人：（name_spider用真实名字代替）
User-agent: name_spider
Allow:

//拦截所有的机器人：
User-agent: *
Disallow: /

//禁止所有机器人访问特定目录：
User-agent: *
Disallow: /cgi-bin/
Disallow: /images/
Disallow: /tmp/
Disallow: /private/

//仅禁止坏爬虫访问特定目录（BadBot用真实的名字代替）：
User-agent: BadBot
Disallow: /private/

//禁止所有机器人访问特定文件类型：
User-agent: *
Disallow: /*.php$
Disallow: /*.js$
Disallow: /*.inc$
Disallow: /*.css$

```


### 百度的rebot协议

```
steve:steveBlog steveyuan$ curl baidu.com/robots.txt
User-agent: Baiduspider
Disallow: /baidu
Disallow: /s?
Disallow: /ulink?
Disallow: /link?

User-agent: Googlebot
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: MSNBot
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: Baiduspider-image
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: YoudaoBot
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: Sogou web spider
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: Sogou inst spider
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: Sogou spider2
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: Sogou blog
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: Sogou News Spider
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: Sogou Orion spider
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: JikeSpider
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: Sosospider
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: PangusoSpider
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: yisouspider
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: EasouSpider
Disallow: /baidu
Disallow: /s?
Disallow: /shifen/
Disallow: /homepage/
Disallow: /cpro
Disallow: /ulink?
Disallow: /link?

User-agent: *
Disallow: /

```


## 参考资料

* [nginx的目录和配置语法](https://www.jianshu.com/p/9dee493be448)

* [Nginx基本配置语法](https://www.myfreax.com/nginx-base-configuration-syntax/)

* [Nginx配置文件语法教程](http://www.cnblogs.com/EasonJim/p/7823284.html)

* [Nginx Location指令URI匹配规则详解](https://blog.csdn.net/xyang81/article/details/51989079)

* [Nginx的location详解](https://www.kancloud.cn/curder/nginx/97541)

* [Nginx配置文件Rewrite语法](https://www.cnyunwei.cc/archives/1128)

* [nginx配置location总结及rewrite规则写法](http://seanlook.com/2015/05/17/nginx-location-rewrite/)

