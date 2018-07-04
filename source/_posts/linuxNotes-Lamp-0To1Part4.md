---
title: 'linux运维学习笔记10-3：httpd的基本配置(六)'
date: 2018-06-04 11:21:12
categories: linux
tags: [linux, LAMP, php, notes]
---


## 写在前面

本章节LAMP最后一个部分，php动态模块的安装。

## php动态模块的安装

当php安装完成后，如果发现没有某个模块，解决思路有两种：一是重新编译安装；而是安装动态模块。

php常用的动态模块源码包，都在安装目录ext目录 

![](https://farm2.staticflickr.com/1757/42553044511_b54fc7a1d0_o.png)

<!--more-->

查询已安装模块执行命令：

```
php -m
```

![](https://farm2.staticflickr.com/1751/40742483330_00f8a0f6a6_o.png)

查询模块安装目录：

```
[root@stevey no-debug-zts-20170718]# php -i | grep extension_dir

extension_dir => /usr/local/php/lib/php/extensions/no-debug-zts-20170718 => /usr/local/php/lib/php/extensions/no-debug-zts-20170718
```

### 安装动态模块：

#### 源码包安装

##### 安装redis模块：网络下载源码包安装。

```
//下载redis
wget https://codeload.github.com/phpredis/phpredis/zip/develop

//文件包重命名
mv develop phpredis-develop.zip

//进入安装包
cd phpredis-develop

//安装依赖项
yum install -y autoconf

//生成configure文件
usr/local/php/bin/phpize

//编译参数
./configure --with-php-config=/usr/local/php/bin/php-config

//编译安装
make && make install
```

出现如下提示，说明安装成功

![](https://farm2.staticflickr.com/1751/40743986190_2f54b59455_o.png)

redis被安装在`/usr/local/php/lib/php/extensions/no-debug-zts-20170718/`，查看结果：

![](https://farm2.staticflickr.com/1760/28678360688_74b0692a6a_o.png)

安装成功后，还需要更改配置文件

```
[root@stevey src]# vim /usr/local/php/etc/php.ini

//搜索extension，在末尾增加一行后保存退出
extension = redis.so

//重启服务
[root@stevey src]# apachectl -t
Syntax OK
[root@stevey src]# apachectl graceful

//检查redis是否加载
[root@stevey src]# php -m | grep redis
redis
```

##### 安装soap：本地源码包

本地源码包php源码包目录下的`ext `目录下，

![](https://farm2.staticflickr.com/1726/41831490514_cc73d952ff_o.png)

```
//进入源码包
cd ext/ldap

//生成configure文件
/usr/local/php/bin/phpize

//编译参数
./configure --with-php-config=/usr/local/php/bin/php-config --enable-soap

//编译安装
make && make install

//生成一个目录来存放扩展的模块并复制soap.so到模块目录
mkdir /usr/local/php/etc/php/ext
cp /usr/local/php/lib/php/extensions/no-debug-zts-20170718/soap.so /usr/local/php/etc/php/ext

//编辑php.ini文件,指定PHP到哪个目录读模块
vim /usr/local/php/etc/php.ini
//修改以下内容
extension_dir="/usr/local/php/etc/php/ext"
extension=soap.so

//检查语法错误并重启httpd服务
apachectl -t
apachectl graceful

//检查是否已加载
/usr/local/php/bin/php -m|grep ftp

```

#### 拓展库安装

除了采用源码包安装拓展，还可以使用拓展库进行安装，常见的拓展库包括pecl、pear两个库：

* `PECL` PHP的标准扩展库，它提供了一系列已知的扩展库，由C++等其他语言编写而成，多数以Dll（动态链接库）的形式体现；主页地址：pecl.php.net


* `PEAR` PHP的官方开源类库（the PHP Extension and Application Repository）的缩写，涵盖了页面呈面、数据库访问、文件操作、数据结构、缓存操作、网络协议等许多方面，用户可以很方便地使用。主页地址：pear.php.net

##### `pecl` 拓展库安装

**语法：**

```
//搜索
pecl search [软件包名]

//安装
pecl install [软件包名]

//列出已经安装的软件
pecl list

```

**示例：(安装apc为例)**

APC，全称是Alternative PHP Cache，官方翻译叫”可选PHP缓存”。它为我们提供了缓存和优化PHP的中间代码的框架。 APC的缓存分两部分:系统缓存和用户数据缓存。

- 搜索

![](https://farm2.staticflickr.com/1731/42611967001_e02a532384_o.png)

- 安装

pecl install apc

- 安装完成后，编辑php.ini，加入一行：

```
extension = apc.so
```

-重新加载apache配置

```
apachectl graceful
```

pear 跟pecl的用法类似，不再赘述。

## 参考资料

* [apache rewrite教程](http://coffeelet.blog.163.com/blog/static/13515745320115842755199/ ) 

* [Apache Rewrite 规则详解](http://www.cnblogs.com/top5/archive/2009/08/12/1544098.html)

* [php错误日志级别参考](http://ask.apelearn.com/question/6973) 

* [php开启短标签 ](http://ask.apelearn.com/question/120)

* [php.ini详解](http://legolas.blog.51cto.com/2682485/493917) 

* [PHP扩展模块Pecl、Pear以及Perl的区别](http://www.phpernote.com/php-function/1358.html)

