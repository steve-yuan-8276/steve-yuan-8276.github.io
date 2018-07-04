---
title: 'linux运维学习笔记9-5：httpd的基本配置(二)'
date: 2018-05-29 21:59:47
categories: linux
tags: [linux, LAMP, php, notes] 
---

## 写在前面

这则笔记继续整理Lamp相关知识，主要包括apache用户认证、域名跳转、访问日志。

## Apache用户认证

有时候，我们不希望自己的网站所有人都能看到或者访问，就可以使用密码认证，也就是只有特定的用户才能够访问，apache提供了这样一种用户认证的功能。

### 全站用户用户认证

具体的设置分三步：

#### 第一步：编辑虚拟主机配置文件

```
vim /usr/local/apache2/conf/extra/httpd-vhosts.conf

//此处修改的是www.abc.com的配置文件
<VirtualHost *:80>
    DocumentRoot "/data/wwwroot/www.abc.com"
    ServerName www.abc.com
    <Directory /data/wwwroot/www.abc.com>  //指定认证目录
    AllowOverride AuthConfig   //认证开关
    AuthName "steveY"    //认证用户提示
    AuthType Basic       //认证类型
    AuthUserFile /data/.htpasswd   //认证文件路径
    require valid-user   //指定所有用户都必须认证
    </Directory>
</VirtualHost>

```

<!--more-->


#### 第二步：创建访问用户密码文件

```
//htppasswd 命令为密码创建工具，-c表示create创建，-m指定加密方式为MD5；/data/.htpasswd 为密码文件，steveY 为用户名

/usr/local/apache2.4/bin/htpasswd -cm /data/.htpasswd steveY    
New password: *************
Re-type new password: *************
Adding password for user steveY

```


#### 第三步：重启Apache

```
//验证格式是否有错误\
/usr/local/apache2.4/bin/apachectl  -t

//重启apache服务
/usr/local/apache2.4/bin/apachectl  restart

//或者重新加载配置文件
/usr/local/apache2.4/bin/apachectl graceful

```

#### 第四步：修改客户端host

这里的客户端指的是要用那个电脑去访问www.abc.com,那笔者来说，lamp环境搭建在本地mac电脑上的虚拟机，之后在mac电脑访问虚拟机，此时mac就是客户端。

mac的host配置文件是/etc/hosts ，我们就来修改这个文件

```
steve:~ steveyuan$ sudo vim /etc/hosts
//增加一行
172.16.155.128 www.abc.com

```

### 目录或文件认证

#### 目录认证

跟上面的全站认证类似，只需要修改一个地方，即将认证目录改为需要进行密码认证加密的文件夹目录，修改保存后重新加载配置文件即可。

![](https://farm1.staticflickr.com/897/40645749890_9be7a72c65_o.png)

#### 文件认证

与目录认证类似，只不过这里要将Directory 改为 FilesMatch，其他步骤不变。修改保存后，重新加载配置文件。

```
<FilesMatch admin.php>  //指定认证文件
    AllowOverride AuthConfig   //认证开关
    AuthName "steveY"    //认证用户提示
    AuthType Basic       //认证类型
    AuthUserFile /data/.htpasswd   //认证文件路径
    require valid-user   //指定所有用户都必须认证
</FilesMatch>
```

## 域名跳转

这样做的好处：

* 增加SEO权重；

* 如果老的域名被放弃了，可以很方便地重定向到新的域名；

实现步骤如下：

### 第一步：修改虚拟主机配置

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

//仍然对www.abc.com进行修改

<VirtualHost *:80>
    DocumentRoot "/data/wwwroot/www.abc.com"
    ServerName www.abc.com
    ServerAlias abc.com
    <IfModule mod_rewrite.c>   //需要mod_rewrite模块支持
        RewriteEngine on  //打开rewrite功能
        RewriteCond %{HTTP_HOST} !^www.abc.com$   //定义rewrite的条件，主机名（域名）不是www.123.com满足条件
        RewriteRule ^/(.*)$ http://www.abc.com/$1 [R=301,L]   //定义rewrite规则，当满足上面的条件时，这条规则才会执行
    </IfModule>
</VirtualHost>

```

### 第二步：修改apache主配置文件，加载rewrite模块

```
//检测是否已加载rewrite模块
[root@stevey ~]# /usr/local/apache2.4/bin/apachectl -M | grep -i rewrite   //没有输出结果，说明未加载

//编辑主配置文件
vim /usr/local/apache2.4/conf/httpd.conf 

//找到rewrite，把前面注释#去掉
LoadModule rewrite_module modules/mod_rewrite.so
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

出现以下结果说明模块已成功加载
![](https://farm2.staticflickr.com/1733/41731447664_b89857a169_o.png)

### 第四步：测试结果

```
[root@stevey ~]# curl -x127.0.0.1:80 -I abc.com
HTTP/1.1 301 Moved Permanently
Date: Wed, 30 May 2018 14:46:03 GMT
Server: Apache/2.4.33 (Unix) PHP/7.2.6
Location: http://www.abc.com/   //表示永久跳转至www.abc.com
Content-Type: text/html; charset=iso-8859-1

```

## Apache访问日志

网站日志文件保存在`/usr/local/apache2.4/logs/`，通过访问日志，能给了解系统的运行情况。

### 查看系统访问日志

```
[root@stevey ~]# cat /usr/local/apache2.4/logs/abc.com-access_log
127.0.0.1 - - [30/May/2018:22:56:15 +0800] "HEAD HTTP://abc.com/ HTTP/1.1" 301 - "-" "curl/7.29.0"
```

### 修改日志配置文件

#### 查看配置文件执行以下命令

```
cat /usr/local/apache2.4/conf/httpd.conf
```

![](https://farm2.staticflickr.com/1736/41709808504_b375256290_o.png)

这里要关注的是记录格式说明

标示|含义
---|---
%h|访问网站的IP;
%l|访问远程登录名,这个字段基本上为"-";
%u|用户名,当使用用户认证时,这个字段为认证的用户名;
%t|时间;
%r|请求的动作(比如用ctrl-I是就为HEADE);
%s|请求的状态,写成%>s为最后的状态码;
%b|传输数据大小;
%{Referer}i|referer信息(访问来源网页信息,比如通过百度找到阿铭论坛,那么referer就是baidu);
%{User-Agent}i|浏览器标识;

**注：**在示例文件中给出了两种日志格式：common和combined，后者比前者多了访问来源`%{Referer}i`和浏览器标示信息`%{User-Agent}i`。

#### 配置虚拟主机配置文本日志格式

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/wwwroot/www.abc.com"
    ServerName www.abc.com
    ServerAlias abc.com
    <IfModule mod_rewrite.c>
        RewriteEngine on
        RewriteCond %{HTTP_HOST} !^www.abc.com$
        RewriteRule ^/(.*)$ http://www.abc.com/$1 [R=301,L]
    </IfModule>
    //增加一行
    CustomLog "logs/abc.com-access_log" combined
</VirtualHost>

//退出保持后测试语法错误
/usr/local/apache2.4/bin/apachectl -t

//重新加载
/usr/local/apache2.4/bin/apachectl graceful

```

