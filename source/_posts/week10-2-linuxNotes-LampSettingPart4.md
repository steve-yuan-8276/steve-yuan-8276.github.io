---
title: 'linux运维学习笔记10-2：httpd的基本配置(四)'
date: 2018-05-31 15:55:38
categories: linux
tags: [linux, LAMP, php, notes]
---

## 写在前面

接着整理关于Lamp环境配置的知识，主要包括防盗链、访问控制等，

## 配置防盗链

### 盗链和防盗链

**盗链**，就是未经授权使用了别人的服务器资源、损害别人利益的行为。比如，A未经同意使用了B服务器上的图片、视频等资源，提供给了C；对于资源所有者B而言，这种行为白白占用了我的带宽，又没有从中获取什么好处，就定义为盗链行为。

**防盗链**，这是针对盗链行为的一种策略，就是通过技术手段防止别人揩油盗链。简单说，就是通过设置黑名单或者白名单，有选择性地对允许或者拒绝。

HTTP 协议支持的 Referer 机制，因此可以通过Header 中的 referer 字段识别访问请求来源，从而采取相应的措施。

<!--more-->

配置方法：

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/wwwroot/www.abc.com"
    ServerName www.abc.com
    ServerAlias abc.com
    CustomLog "logs/abc.com-access_log" combined
    <Directory /data/wwwroot/www.abc.com>   //定义防盗链目录
        SetEnvNoCase Referer "http://www.abc.com" local_ref
        SetEnvNoCase Referer "http://abc.com" local_ref
        SetEnvNoCase Referer "^$" local_ref
        <filesmatch "\.(txt|doc|mp3|zip|rar|jpg|gif|pdf)">
            Order allow,deny
            Allow from env=local_ref
        </filesmatch>
    </Directory>
</VirtualHost>
```

**注：**这是一个白名单策略，分为三个部分：

* 第一部分：referer可以是`http://www.abc.com` 、`http://abc.com`，也可以为`^$`，意思是为空；

* 第二部分：filesMatch，这里采用了通配符，意思是说该策略对后缀为txt、doc、mp3、zip、rar、jpg、gif、pdf的文件生效；

* 第三部分：执行什么操作，`Order Allow,Deny`，说明默认操作是允许，也就是这是符合上述条件的请求允许访问；

修改完成后，检查语法错误，重新加载即生效。

## 访问控制

前面已经学习了用户认证限制访问，除此之外还可以对来源ip、执行权限或者user_agent进行限制。

### 限制ip

限制访问者ip，常用的使用的使用场景，比如企业网站后台只允许公司内网或者指定的外网ip段可以访问，防止黑客访问企业网站后台获取数据。

配置方法：

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/wwwroot/www.abc.com"
    ServerName www.abc.com
    ServerAlias abc.com
    CustomLog "logs/abc.com-access_log" combined
    <Directory /data/wwwroot/www.abc.com>   //定义防盗链目录
        Order deny,allow    //默认拒绝  
        Deny from all       //先拒绝所有访问请求
        Allow from 127.0.0.1   //再对指定ip开放
    </Directory>
</VirtualHost>
```

保存退出后，检查语法错误，httpd重新加载生效。

### 限制php解析

这个在之前学习前端课程的时候学过一点。如果不做限制，我们用html、js或者php写的脚本会被解析执行。为了避免这种情况的出现，我们可以在上传目录禁止php代码解析。

配置方法：

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/wwwroot/www.abc.com"
    ServerName www.abc.com
    ServerAlias abc.com
    CustomLog "logs/abc.com-access_log" combined
    <Directory /data/wwwroot/www.abc.com/upload>   //定义目录
        php_admin_flag engine off     //禁止php解析
    </Directory>
</VirtualHost>
```

### 限制user_agent

User Agent中文名为用户代理，简称 UA，它是一个特殊字符串头，使得服务器能够识别客户使用的操作系统及版本、CPU 类型、浏览器及版本、浏览器渲染引擎、浏览器语言、浏览器插件等。

利用google chrome浏览器的DevTools，可以查看user0agent等信息。在网络传输的过程中，这些信息都是明文传输的，服务器可以通过这些内容来判断是真实用户访问，还是网络爬虫在访问。

![](https://farm1.staticflickr.com/890/41748119204_a574b12851_o.png)

限制user_agent，这个操作的应用场景就是为了防止网络爬虫。所谓“爬虫”，就是一种程序，比如我们可以用python写一个爬虫，把知乎上的好友名单都抓取出来。这当然是一种很便捷的获取信息的手段。但是，站在知乎网站管理员的校对，频繁访问加大了服务器的压力、站用了有限的带宽，当然是不鼓励的，所以就需要限制。

配置方法：

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/wwwroot/www.abc.com"
    ServerName www.abc.com
    ServerAlias abc.com
    CustomLog "logs/abc.com-access_log" combined
    <Directory /data/wwwroot/www.abc.com/upload>   //定义目录
        RewriteEngine on
        RewriteCond %{HTTP_USER_AGENT} .*curl.* [NC,OR]
        RewriteCond %{HTTP_USER_AGENT} .*baidu.com.* [NC]
        RewriteRule .* - [F]
    </Directory>
</VirtualHost>
```

保存退出后，检查语法错误后，重新加载服务生效。

注：这里用到的是hettp的rewrite模块，%{HTTP_USER_AGENT}是user_agent的内置变量，方括号中的OR表示或者，NC表示不区分大小写。在示例中，当匹配规则时，就会触发下面的规则，F表示forbidden禁止。

## 参考资料

* [什么是User Agent?简单了解一下](https://www.jianshu.com/p/023f7cd1927c)

* [浏览器User-Agent（用户代理）的介绍](http://blog.51cto.com/mingkrcom/1434091)

* [防盗链和反盗链的原理](https://blog.csdn.net/djd1234567/article/details/52210055)

* [谈谈网站防盗链](http://blog.51cto.com/windyli/315283)	

* [图片和视频防盗链简单介绍](https://www.cnblogs.com/saysmy/p/8647808.html)

