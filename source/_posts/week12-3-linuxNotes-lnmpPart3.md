---
title: 'linux学习笔记12-3：lnmp环境的搭建和配置（三）'
date: 2018-06-08 10:33:31
categories: linux
tags: [linux, LNMP, notes]
---

## 写在前面

继续整理lnmp环境配置的学习笔记，主要是Nginx 日志配置，包括日志切割，以及静态文件过滤等。

## Nginx访问日志

跟apache一样，nginx也有访问日志的功能。相比较而言，nginx访问日志的设定还要更简洁明了一些。

### 日志格式

#### `combined log_format`

**注：**combined日志格式，相当于apache的combined日志格式。

```
log_format  combined   '$remote_addr - $remote_user  [$time_local]  '
                       ' "$request"  $status  $body_bytes_sent  '
                       ' "$http_referer"  "$http_user_agent" ';

```

<!--more-->

#### `main log_format`

**注：**lof_format的默认值

```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '"$status" $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for" '
                  '"$gzip_ratio" $request_time $bytes_sent $request_length';
```
                                  
#### `proxy log_format`

**注：**nginx位于负载均衡器，squid，nginx反向代理之后，web服务器无法直接获取到客户端真实的IP地址了。 $remote_addr获取反向代理的IP地址。反向代理服务器在转发请求的http头信息中，可以增加X-Forwarded-For信息，用来记录 客户端IP地址和客户端请求的服务器地址。

```
log_format  porxy   '$http_x_forwarded_for - $remote_user  [$time_local]  '
                    ' "$request"  $status $body_bytes_sent '
                    ' "$http_referer"  "$http_user_agent" ';
```

### 常用的配置参数

参数|注释
---|---
$remote_addr|客户端IP地址
$http_x_forwarded_for|代理服务器IP
$remote_user |客户端用户名称
$request |请求的URL和HTTP协议
$status |请求返回的状态码，例如：200、301、404等
$body_bytes_sent |发送给客户端的字节数，不包括响应头的大小； 
$bytes_sent |发送给客户端的总字节数。
$connection |连接的序列号。
$connection_requests |当前通过一个连接获得的请求数量。
$msec |日志写入时间，单位为秒。
$pipe |如果请求是通过HTTP流水线(pipelined)发送，pipe值为“p”，否则为“.”。
$http_referer |记录从哪个页面链接访问过来的，可以根据该参数进行防盗链设置
$http_user_agent |客户端浏览器相关信息
$request_length |请求的长度，包括请求行，请求头和请求正）。
$request_time |请求处理时间，单位为秒，精度毫秒； 从读入客户端的第一个字节开始，直到把最后一个字符发送给客户端后进行日志写入为止。
$time_iso8601 |ISO8601标准格式下的本地时间。
$time_local |通用日志格式下的本地时间。
$sent_http_content_range  |这是一个传递到客户端的头，这类头的前缀为"sent_http_" 。
$upstream_http_  |这是有upstream模块产生了日志，因此它的前缀将会是这个前缀。

### 日志相关命令

#### `access_log`

access_log 设定日志的路径、格式、buffer大小，是否压缩等。如果不想启用日志则access_log off ;

语法：

```
access_log  path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
access_log  off;
```

示例：

```
access_log logs/access.log combined;
```

**注：**该命令分为三个字段：

* 关键字：其中关键字access_log不能更改；

* 日志文件：可以指定存放日志的路径；

* 格式标签：给日志文件套用指定的日志格式；

#### log_format

设置日志格式

语法：

```
log_format name [escape=default|json] string ...;
```

示例：

```
log_format  combined   '$remote_addr - $remote_user  [$time_local]  '
                       ' "$request"  $status  $body_bytes_sent  '
                       ' "$http_referer"  "$http_user_agent" ';

```

**注：**该命令分为三个部分：

* 关键字：其中关键字error_log不能改变；

* 格式标签：格式标签是给一套日志格式设置一个独特的名字；

* 日志格式：日志设置格式具体包含的元素；

#### open_log_file_cache 

该命令为频繁使用的日志文件描述符所在的路径变量设置缓存。

语法：

```
open_log_file_cache max=N [inactive=time] [min_uses=N] [valid=time] | off
```

**指令选项：**

* max - 缓存中存储的最大文件描述符数；

* inactive - 设置缓存中在某个时间段内没有使用的文件描述符将被移除，默认为10秒；

* min_uses - 在一定时间内（inactive指定），一个文件描述符最少使用多少次后被放入缓存，默认为1；

* valid - 设置检查同名文件存在的时间，默认是60秒；

* off - 关闭 cache；

示例

```
open_log_file_cache max=1000 inactive=20s min_uses=2 valid=1m;
```

### nginx 日志配置：以test.com为例

#### 第一步：编辑配置文件

```
vi /usr/local/nginx/conf/vhost/test.com.conf

//加入一行配置
access_log /tmp/nginx/test.log combined
```

#### 第二步：新建日志目录

```
mkdir -p /tmp/nginx
```

#### 第三步：重新加载服务

```
nginx -t

nginx -s reload
```

#### 第四步：测试成果

```
//测试访问
steve:steveBlog steveyuan$ curl -x 172.16.155.128:80 test1.com/123.html -I
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Sun, 10 Jun 2018 14:30:21 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: http://test.com/123.html

steve:steveBlog steveyuan$ curl -x 172.16.155.128:80 test1.com/asdasd.html -I
HTTP/1.1 301 Moved Permanently
Server: nginx
Date: Sun, 10 Jun 2018 14:30:35 GMT
Content-Type: text/html
Content-Length: 178
Connection: keep-alive
Location: http://test.com/asdasd.html

//查看日志
[root@stevey test.com]# cat /tmp/nginx/test.log
172.16.155.1 - - [10/Jun/2018:22:30:21 +0800] "HEAD http://test1.com/123.html HTTP/1.1" 301 0 "-" "curl/7.54.0"
172.16.155.1 - - [10/Jun/2018:22:30:35 +0800] "HEAD http://test1.com/asdasd.html HTTP/1.1" 301 0 "-" "curl/7.54.0"

```


### Nginx日志切割

#### 第一步：撰写脚本

```
vi /usr/local/sbin/nginx_log_rotate.sh

#!/bin/bash
## 假设nginx的日志存放路径为/data/logs/
d=`date -d "-1 day" +%Y%m%d`   //这个日期是昨天的日期，因为日志切割是第二天才执行这个脚本的。
logdir="/data/logs"
nginx_pid="/usr/local/nginx/logs/nginx.pid"
cd $logdir
for log in `ls *.log`
do
    mv $log $log-$d
done
/bin/kill -HUP `cat $nginx_pid` //跟Nginx的-s重新加载配置文件一样
```

#### 第二步：修改脚本权限

```
chomod u+x /usr/local/sbin/nginx_log_rotate.sh
```

#### 第三步：添加系统任务

```
crontab -e //添加任务计划

//增加如下内容：

0 0 * * * /bin/bash /usr/local/sbin/nginx_log_rotate.sh

```

### 静态文件不记录日志和过期时间

#### 第一步：修改虚拟主机配置文件

```
vim /usr/local/nginx/conf/vhost/test.com.conf

//增加如下内容：

location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$  //匹配脱义静态文件
    {
          expires      7d;   //配置过期时间
          access_log off;
    }
location ~ .*\.(js|css)$    //匹配js,css文件
    {
          expires      12h;
          access_log off;
    }
```

如下图所示：

![](https://farm2.staticflickr.com/1730/40903195100_638901636b_o.png)

#### 第二步：重新加载nginx服务

```
nginx -t

nginx -s reload
```

#### 第三步：测试结果

```
//创建测试文件
cd /data/nginx/test.com

//创建测试文件
[root@stevey test.com]# echo 'js' > /data/nginx/test.com/1.js
[root@stevey test.com]# echo 'jpeg' > /data/nginx/test.com/1.jpeg

//测试访问
steve:steveBlog steveyuan$ curl -x 172.16.155.128:80 test1.com/1.js
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
steve:steveBlog steveyuan$ curl -x 172.16.155.128:80 test1.com/1.jpeg
<html>
<head><title>301 Moved Permanently</title></head>
<body bgcolor="white">
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>

//查看访问日志 (没有js和jpeg的访问记录)
[root@stevey test.com]# cat /tmp/nginx/test.log
172.16.155.1 - - [10/Jun/2018:22:30:21 +0800] "HEAD http://test1.com/123.html HTTP/1.1" 301 0 "-" "curl/7.54.0"
172.16.155.1 - - [10/Jun/2018:22:30:35 +0800] "HEAD http://test1.com/asdasd.html HTTP/1.1" 301 0 "-" "curl/7.54.0"

```


## 参考资料

* [Nginx访问日志（access_log）配置及信息详解](https://blog.csdn.net/czlun/article/details/73251723)

* [nginx访问日志和错误日志](https://blog.csdn.net/zhezhebie/article/details/73302089)

* [Nginx访问日志](https://www.jianshu.com/p/a3916e1f3a5d)

* [nginx access_log日志](https://lanjingling.github.io/2016/03/14/nginx-access-log/)

* [nginx日志格式及自定义日志配置](https://blog.csdn.net/javaloveiphone/article/details/74238866)

* [Nginx 日志配置详情解析](https://juejin.im/post/59f94f626fb9a045023af34c)

* [nginx日志模块及日志格式](https://my.oschina.net/u/259976/blog/832536)

