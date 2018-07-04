---
title: 'linux学习笔记12-2：lnmp环境的搭建和配置（二）'
date: 2018-06-07 16:32:49
categories: linux
tags: [linux, LNMP, notes]
---


## 写在前面

这则笔记继续整理lnmp环境的配置，主要包括配置虚拟主机、nginx用户认证和域名重定向。

### nginx的安装目录

路径|类型|作用
---|---|---
/etc/nginx/conf.d/default.conf|配置文件	|nginx主配置文件
/etc/logrotate.d/nginx|	配置文件|用于logrotate服务的日志切割
/etc/nginx/nginx.conf<br> /etc/nginx/fastcgi_params<br> /etc/nginx/uwsgi_params<br> /etc/nginx/scgi_params	|配置文件|cgi配置相关、fastcgi配置
/etc/nginx/koi-utf<br> /etc/nginx/win-utf<br> /etc/nginx/koi-win	|配置文件	|编码转换映射转化文件
/etc/nginx/mime.types	|配置文件|设置http协议的Content-Type与扩展名对应关系 ，处理一些识别不了的扩展名的时候需要用到
/usr/lib/systemd/system/nginx.service<br> /usr/lib/systemd/system/nginx-debug.service<br> /etc/sysconfig/nginx<br> /etc/sysconfig/nginx-debug|	配置文件	|用于配置出系统守护进程管理器管理方式
/usr/sbin/nginx<br> /usr/sbin/nginx-debug|	命令	|nginx服务终端命令
/usr/share/doc/nginx-1.12.2/COPYRIGHT<br> /usr/share/man/man8/nginx.8.gz	|文件、目录	|nginx的手册和帮助文件
/var/cache/nginx/|	目录|	nginx的缓存目录
/var/log/nginx/	|目录|	nginx的日志目录

<!--more-->

### 常用配置文件

nginx.conf 为主配置文件；

inlcude是一个指令，当一个服务器上有多个网站共享时，通过设置include可以很方便的对虚拟主机站点进行管理。


Standard name  |   Description
---|---
nginx.conf     |   主配置文件   
fastcgi.conf   |   FastCGI相关配置  
proxy.conf     |   代理相关配置  
sites.conf     |   虚拟主机配置  

修改配置文件后需要重启nginx

```
//重启nginx
systemctl restart nginx.service

//重新记载配置文件
systemctl reload nginx.service

```

## nginx的基础设置

nginx的配置文件为/usr/local/nginx/conf/nginx.conf

首先执行如下命令清空配置文件

```
[root@stevey etc]# > /usr/local/nginx/conf/nginx.conf
```

之后，编辑配置文件，按照如下内容进行修改

```
vi /usr/local/nginx/conf/nginx.conf

//配置文件如下
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
    // fastcgi
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    fastcgi_intercept_errors on;
    // gzip 
    gzip off;
    gzip_min_length 1k;    //表示只有1k以上才压缩
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


检查语法错误

```
nginx -t

```

重新加载nginx服务

```
nginx -s reload

```
   
## nginx常用设置

### 默认虚拟主机设定

#### 第一步：编辑nginx配置

```
vi /usr/local/nginx/conf/nginx.conf

//在结束符号} 之前添加一行配置
include vhost/*.conf;
```

如图所示：注意不要忘记句尾的冒号（英文标点）

![](https://farm2.staticflickr.com/1728/41767908315_e697ff4c06_o.png)

#### 第二步：新建配置文件夹

```
//新建虚拟主机文件夹
mkdir -p /usr/local/nginx/conf/vhost //该目录下所有配置文件均会被加载

cd /usr/local/nginx/conf/vhost

```

#### 第三步：配置虚拟主机

```
vi default.conf

//加入如下配置
server
{
    listen 80 default_server; //default_server 表示默认虚拟主机
    server_name steve.discuz.com;
    index index.html index.htm index.php;
    root /data/ngix/default;
}

//保存退出
```

#### 第四步：创建测试文件

```
//创建项目目录
mkdir -p /data/nginx/default/

//创建索引页
touch index.html

//添加内容到测试页面
echo 'default_server' > /data/nginx/default/index.html
```


#### 第五步：重新加载服务

```
//检查语法错误
nginx -t

//重新记载服务
nginx -s reload
```

#### 第六步：测试效果

```
//测试
curl -x172.16.155.128:80 steve-discuz.com
```

TroubleShooting:拒绝访问

![](https://farm2.staticflickr.com/1746/40859061100_67922a547b_o.png)

**解决思路：**

* 首先，检查了防火设置，firewalld已关闭，iptables已安装启动，并已开放80、3306端口；

* 其次，selinux已关闭，重启虚拟机问题依旧；

* 再次，检查虚拟机配置文件，指定监听80端口；

* 最后，突然想到会不会是网络的问题，surge改为直连模式 direct mode，之后开启ihost，问题解决。

如下所示：

```
steve:~ steveyuan$ curl -x 172.16.155.128:80 steve-discuz.com/index.html
default_server

```

再测一个并不存在的域名,如下图所示，依然返回默认虚拟主机的页面内容，说明虚拟主机配置成功。

![](https://farm2.staticflickr.com/1726/27798952297_5947583b22_o.png)

### 用户认证

#### 第一步：创建新的虚拟主机

```
cd /usr/local/nginx/conf/vhost

vi test.com.conf

//添加以下配置
server
{
    listen 80; 
    server_name steve-test.com;
    index index.html index.htm index.php;
    root /data/ngix/test.com;
    
    location /
    {
        auth_basic 'Auth';
        auth_basic_user_file /usr/local/nginx/conf/htpasswd;
    }
}
```

#### 第二步：创建项目目录

```
//创建项目文件夹
mkdir -p /data/nginx/test.com

//创建index.html
touch index.html

//安装httpd
yum install -y httpd

//生成用户密码文件
htpasswd -c /usr/local/nginx/conf/htpasswd steve
```


#### 第三步：重新加载服务

```
//语法检查
nginx -t

//重新加载服务
nginx -s reload
```

#### 第四步：校验成果

```
echo 'test' > /data/nginx/test.com/index.html

```

如下图所示：用户认证已配置成功

![](https://farm2.staticflickr.com/1758/40901979150_6fb25bb3a9_o.png)

### 域名重定向

**前提准备：**

- 清空虚拟主机配置文件

```
> /usr/local/nginx/conf/vhost/test.com.conf

```
#### 第一步：修改配置

```
vi test.com.conf

server
{
    listen 80; 
    server_name test.com test1.com test2.com;
    index index.html index.htm index.php;
    root /data/nginx/test.com;
    
    if ($host != 'test.com') {
        rewrite ^/(.*)$ http://test.com/$1 permanent;   //permanent为永久重定向，相当于httpd中的R=301；还有一个常用选项叫做redirect，相当于httpd 的 R=301；
    }
}
```

#### 第二步：重新加载服务

```
//检查语法
nginx -t

//重新加载服务
nginx -s reload
```

#### 第三步：校验结果

```
curl -x 172.16.155.128:80 test1.com

```

如下图所示，域名重定向成功

![](https://farm2.staticflickr.com/1754/40902430840_220ee85857_o.png)

### 拓展知识：

#### rewrtie的四种flag

上面例子当中的重定向也是通过rewrite模块实现的，这里补充一些rewrite模块的知识。

对于rewrtie有四种不同的flag，分别是redirect、permanent、break和last，具体如下：

flag | 说明 
---|---
redirect|临时跳转
permanent|永久跳转，一般是为了对搜索引擎友好
last|将rewrite后的地址重新在server标签执行
break|将rewrite后地址重新在当前的location标签执行


## 参考资料

* [关于nginx rewrtie的四种flag](http://www.netingcn.com/nginx-rewrite-flag.html)

* [Nginx基础——Rewrite规则](http://wangyapu.com/2016/03/10/nginx_rewirte/)

* [Nginx配置文件nginx.conf中文详解](http://www.ha97.com/5194.html)

* [搞懂 nginx 的 rewrite 模块](https://juejin.im/entry/58c4c37bac502e0062f6b97c)

