---
title: 'linux学习笔记13-1：lnmp环境的搭建和配置（六）'
date: 2018-06-11 22:03:36
categories: linux
tags: [linux, LNMP, notes]
---

## 写在前面

继续整理lnmp环境的配置知识，这则笔记主要是php-fpm相关的内容。

## 什么是php-fpm？

### php进程管理的基本概念：

先回顾四个概念：

* `CGI：`是 Web Server 与 Web Application 之间数据交换的一种协议；

* `FastCGI：`同 CGI，是一种通信协议，但比 CGI 在效率上做了一些优化。同样，SCGI 协议与 FastCGI 类似；

* `PHP-CGI：`是 PHP （Web Application）对 Web Server 提供的 CGI 协议的接口程序；

* `PHP-FPM：`是 PHP（Web Application）对 Web Server 提供的 FastCGI 协议的接口程序，额外还提供了相对智能一些任务管理；

<!--more-->

### `php-cgi`和`php-fpm`的关系

PHP-CGI就是PHP实现的自带的FastCGI管理器，但是它本身不给力，有明显的不足：

* 只能解析请求，返回结果，不会进程管理；

* php-cgi变更php.ini配置后，需重启php-cgi才能让新的php-ini生效，不可以平滑重启；

* 直接杀死php-cgi进程，php就不能运行了；

基于上述因素，就出现了一些第三方补充解决方案，比如PHP-FPM，它的优点包括：

* 用于调度管理PHP解析器php-cgi的管理程序；

* PHP-FPM通过生成新的子进程可以实现php.ini修改后的平滑重启；

![](https://farm2.staticflickr.com/1730/28861804958_3ef76e6ac3_o.png)

如上图所示，当nginx收到 index.php 这个请求后，会启动对应的 CGI 程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以规定CGI规定的格式返回处理后的结果，退出进程，Web server再把结果返回给浏览器。这就是一个完整的动态PHP Web访问流程。

### php-fpm进程管理

命令行格式：

```
nginx -s [signal]
```

其中,signal常用选项如下：

* quit - 优雅的关闭，即处理完当前请求再关闭;

* reload - 重新载入配置文件;

* reopen - 重新打开日志文件;

* stop - 立即关闭;


## php-fpm的pool

前面提到过，php-fpm的一个重要功能，就是可以管理进程，这里涉及到一个pool的概念，nginx启动服务后，会产生1个master进程+N个worker进程，共同构成一个进程池（pool），其中

* master进程主要负责读取配置文件，并控制管理workder进程；

* worker进程具体负责处理请求；

![](https://farm2.staticflickr.com/1732/42027870974_15373b7ffd_o.png)

先看目前的php-fpm 的pool，执行以下命令：

```
ps -aux | grep php-fpm
```

![](https://farm2.staticflickr.com/1742/28871953188_5274fdd2bb_o.png)

### 配置文件解读

php-fpm的配置文件在 `/usr/local/php-fpm/etc/php-fpm.conf`，查看该文件

```
cat /usr/local/php-fpm/etc/php-fpm.conf
```

![](https://farm1.staticflickr.com/892/27876333777_68de9587e7_o.png)

如上图所示：

#### [global] 全局配置

* `pid` 指定pid路径；

* `error_log` 指定php-fpm的错误日志；

#### [www] 当前的pool设定

* `listen = /tmp/php-fcgi.sock` 以sock的方式监听

* `user = php-fpm` 指定运行的用户

* `group = php-fpm` 指定运行的组

* `pm = dynamic` //定义`php-fpm`的子进程启动模式，`dynamic`为动态进程管理，一开始只启动少量的子进程，根据实际需求，动态地增加或者减少子进程，最多不会超过`pm.max_children`定义的数值。另外一种模式是`static`，这种模式下子进程数量由`pm.max_children`决定，一次性启动这么多，不会减少也不会增加；

* `pm.max_children = 50` //最大子进程数，`ps -aux`可以查看；

* `pm.start_servers = 20` //针对dynamic模式，它定义php-fpm服务在启动服务时产生的子进程服务时产生的子进程数量；

* `pm.min_spare_servers = 5` //针对dynamic模式，定义在空闲时段，子进程数的最少数量，如果达到这个数值时，php-fpm服务会自动派生新的子进程；

* `pm.max_spare_servers = 35` //针对dynamic模式，定义在空闲时段，子进程数的最大值，如果高于这个数值就开始清理空闲的子进程；

* `pm.max_requests = 500` //针对dynamic模式，定义一个子进程最多处理的请求数，也就是说在一个php-fpm的子进程最多可以处理这么多请求，当达到这个数值时，它会自动退出；


此外，在配置文件的末尾，还有一行很关键的配置；

![](https://farm2.staticflickr.com/1729/40934824860_9755ed068b_o.png)

与nginx虚拟主机的配置类似，当同一台服务器上多个网站是，我们可以把`;include=/usr/local/php-fpm/etc/php-fpm.d/*.conf` 注释分号去掉，这样就可以在`/usr/local/php-fpm/etc/php-fpm.d` 下，为每个虚拟主机配置不同的pool，各个网站之间的pool进程池彼此独立，互不影响。

### 设置多个pool

#### 方法一：在`/usr/local/php-fpm/etc/php-fpm`里面增加pool 

第一步：修改配置文件

```
[root@hanfeng etc]# vi /usr/local/php-fpm/etc/php-fpm.conf

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

[steve.com]
listen = /tmp/steve.sock
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

第二步：重新加载 nginx 服务

```
nginx -t

nginx -s reload
```

第三步：检测效果

```
ps -aux | grep php-fpm

```


第四步：在nginx 中启用pool

- 默认主机的pool设定

```
//进入nigix虚拟主机配置文件夹
[root@stevey etc]# cd /usr/local/nginx/conf/vhost/
[root@stevey vhost]# ls
default.conf  test.com.conf

//配置test.com.conf
vi test.com.conf

//相关配置
    location ~ \.php$
    {
        include fastcgi_params;
        fastcgi_pass unix:/tmp/php-fcgi.sock;    //这个就是www的默认pool
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /data/nginx/test.com$fastcgi_script_name;
    }

```

- 新建steve.com的pool设定（假设已经有新的steve.com虚拟主机）

```
[root@hanfeng vhost]# vim steve.com.conf

server
{
    listen 80 default_server;
    server_name steve.com;
    index index.html index.htm index.php;
    root /data/nginx/steve.com;
}
location ~ \.php$
    {
        include fastcgi_params;
        fastcgi_pass unix:/tmp/steve.com.sock;  
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /data/nginx/steve.com$fastcgi_script_name;
    }

```

#### 方法二：跟nginx虚拟主机类似，php-fpmye也支持把配置文件拆分出来。

第一步：修改php-fpm.conf

```
vi php-fpm.conf

//主文件只保留global设定
[global]
pid = /usr/local/php-fpm/var/run/php-fpm.pid
error_log = /usr/local/php-fpm/var/log/php-fpm.log
include = etc/php-fpm.d/*.conf   //这行很关键，修改后各个虚拟主机的pool设定会保存在 `etc/php-fpm.d/` 目录下。
```

第二步：创新配置目录

```
//创建/php-fpm.d/的目录
mkdir -p etc/php-fpm.d
进入配置目录
cd etc/php-fpm.d

//新建www.conf,并修改设定
vim www.conf
```

第三步：创建配置文件

修改www的配置文件
```
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

新建steve.com.conf,并修改相关设定

```
vim steve.com.conf

[steve.com]
listen = /tmp/steve.com.sock
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


第四步：重新加载服务

```
//检查语法
/usr/local/php-fpm/sbin/php-fpm -t
重启服务
/etc/init.d/php-fpm restart
```

第五步：校验结果

```
ps -aux | grep php-fpm

```

### php-fpm慢执行日志slow log

慢执行日志是php-fpm提供的一个很有用的功能，通过查询和分心慢执行日志，方便我们有针对性滴进行优化。

#### 开启slow log

第一步：编辑php-fpm配置文件

```
//PHP 5.3.3 之前设置如下：
<value name="request_slowlog_timeout">5s</value>
<value name="slowlog">logs/php-fpm-slowlog.log</value>

//PHP 5.3.3 之后设置以下如下：
request_slowlog_timeout = 5s
slowlog = /usr/local/php/log/php-fpm-slowlog.log

```

**注：**

* request_slowlog_timeout 是脚本超过多长时间 就可以记录到日志文件；

* slowlog 是日志文件的路径；

第二步：重启php-fpm服务

```
//检查语法
/usr/local/php-fpm/sbin/php-fpm -t
重启服务
/etc/init.d/php-fpm restart
```

第三步：查看slow log

```
cat /usr/local/php/log/php-fpm-slowlog.log

[19-Dec-2013 16:54:49] [pool www] pid 18575
script_filename = /home/web/htdocs/sandbox_canglong/test/tt.php
[0x0000000003a00dc8] curl_exec() /home/web/htdocs/sandbox_canglong/test/tt.php:2
[0x0000000003a00cd0] exfilter_curl_get() /home/web/htdocs/sandbox_canglong/test/tt.php:6

```

**说明：**

* `script_filename` 是入口文件；

* `curl_exec()` ： 说明是执行这个方法的时候超过执行时间的；

* `exfilter_curl_get()` ：说明调用curl_exec()的方法 ；

* 每行冒号后面的数字是行号；

**注：开启`slow log`后，在错误日志文件中也有相关记录**

### open_basedir

open_basedir 是php的一个安全设定，就是就访问权限限定在某个文件夹；就算该网站被黑，也只能在特定的文件夹内操作，而不会危及整个服务器的安全。

可以配置open_basedir的地方一共有以下三处，修改方法如下：

#### 方法一：编辑`php-fpm.conf`

```
php_admin_value[open_basedir]=/home/wwwroot/:/proc/:/tmp/
```

#### 方法二：编辑`nginx fastcgi_param`

```
# set php open_basedir 
fastcgi_param PHP_ADMIN_VALUE "open_basedir=$document_root/:/tmp/:/proc/";
```

#### 方法三：编辑`php.ini`

```
[HOST=www.iamle.com]
open_basedir=/home/wwwroot/www.iamle.com/:/proc/:/tmp/
[PATH=/home/wwwroot/www.iamle.com/]
open_basedir=/home/wwwroot/www.iamle.com/:/proc/:/tmp/
```

#### open_basedir 安全设置建议

三管齐下：

1. 先设置fpm-php中pool池中的总open_basedir；

2. 再对nginx中单个server 通过 fastcgi_param PHP_ADMIN_VALUE 设置；

3. 最后对php.ini设置 [HOST=XXX] [PATH=XXX] 


```
//第一步：在php-fpm.conf对应的pool池中行尾配置
php_admin_value[open_basedir]=/home/wwwroot/:/proc/:/tmp/

//第二步：在nginx fastcgi fastcgi_param配置
#这里用$document_root是一种取巧的方法,也可以设置绝对路径
# set php open_basedir
fastcgi_param PHP_ADMIN_VALUE "open_basedir=$document_root/:/tmp/:/proc/";

//第三步：在php.ini行尾配置
[HOST=www.iamle.com]
open_basedir=/home/wwwroot/www.iamle.com/:/proc/:/tmp/
[PATH=/home/wwwroot/www.iamle.com/]
open_basedir=/home/wwwroot/www.iamle.com/:/proc/:/tmp/

```

## 参考资料

* [浅谈PHP进程管理](http://www.manks.top/php-cgi-fpm.html)

* [CGI、FastCGI和PHP-FPM关系图解](https://www.awaimai.com/371.html)

* [php-fpm的pool](https://cloud.tencent.com/developer/article/1038373)

* [善用php-fpm的慢执行日志slow log，分析php性能问题](https://www.bo56.com/%E5%96%84%E7%94%A8php-fpm%E7%9A%84%E6%85%A2%E6%89%A7%E8%A1%8C%E6%97%A5%E5%BF%97slow-log%EF%BC%8C%E5%88%86%E6%9E%90php%E6%80%A7%E8%83%BD%E9%97%AE%E9%A2%98/)

* [php-fpm慢日志查询](https://blog.csdn.net/ty_hf/article/details/55504172)

* [PHP-FPM配置文件参数解释](http://www.ywnds.com/?p=5329)

* [LNMP架构——php-fpm的进程pool设置](https://blog.csdn.net/MrDing991124/article/details/79018917)

* [nginx+php(fpm-php fastcgi)open_basedir安全设置](https://my.oschina.net/jeeker/blog/616431)

