---
title: 'linux学习笔记: tomcat的基础知识(一)'
date: 2018-06-25 11:05:19
categories: linux
tags: [linux, tomcat, notes]
---

## 写在前面

这则笔记开始，整理tomcat的相关知识。

## 什么是tomcat？

目前，有很多的大型网站都是用java编写的，所以，解析java程序也需要由相关的程序来完成，tomcat就是其中之一。

Tomcat 类似与一个apache的扩展型，属于apache软件基金会的核心项目，属于开源的轻量级Web应用服务器，是开发和调试JSP程序的首选，主要针对Jave语言开发的网页代码进行解析，**具有技术先进、性能稳定、免费开源的特点，**是目前比较流行的web服务器。

Tomcat虽然具有处理HTML页面的功能，然而由于其处理静态HTML的能力远不及Apache或者Nginx，所以Tomcat通常做为一个Servlet和JSP容器单独运行在后端，运行JSP 页面和Servlet。

![](https://farm1.staticflickr.com/887/28125102597_008c4a1f95_o.jpg)

<!--more-->

Tomcat 支持：

* 使用 Apache Axis servlet 的 Web 服务；

* 开发框架，如 Apache Struts；

* 模板引擎，如 Apache Jakarta Velocity；

* 对象关系映射技术，如 Hibernate；

## Tomcat 安装与设置

想要安装tomcat，必须先安装JDK，即`java development kit`，这是Sun Microsystems 针对java开发做的一个开发者软件包，包括java运行环境、java工具和java基础库，是目前java开发的核心。

### 安装jdk


#### 第一步：下载安装包

[jdk官网下载地址在这里](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，由于这里不能使用wget命令，因此只能通过浏览器下载，之后通过ftp程序上传到虚拟机当中，进行相关的安装配置。

这里选择了linux64位的版本来下载。

![](https://farm2.staticflickr.com/1768/41206433620_6f2e0d2ba6_o.png)

#### 第二步：上传安装包到虚拟机

因为之前已经配置好了ftp服务，所以这里刚好用上。

笔者在mac端使用的ftp软件是`yummy ftp pro` ，打开之后，依次选取传输协议、客户端ip、登陆用户名、密码，点击connect即可连接。

![](https://farm2.staticflickr.com/1807/43016919531_9b2f907d87_o.png) 

之后，选择文件，右键uoload即可。

#### 第三步：移动安装包

```
// 进入ftp目录
[root@local-linux02 ~]# cd /data/ftp

//查看目录，该文件已成功传输到虚拟机当中
[root@local-linux02 ftp]# ls
a.txt  b.txt  jdk-8u171-linux-x64.tar.gz  test1  test2

//移动文件夹
[root@local-linux02 ftp]# mv jdk-8u171-linux-x64.tar.gz /usr/local/src/

//进入源码包保存目录
[root@local-linux02 ftp]# cd /usr/local/src/
[root@local-linux02 src]# ls
jdk-8u171-linux-x64.tar.gz  nginx-1.14.0.tar.gz    php-7.2.6
nginx-1.14.0                nginx-1.14.0.tar.gz.1  php-7.2.6.tar.gz

//解压
[root@local-linux02 src]# tar -zxvf jdk-8u171-linux-x64.tar.gz 

//查看解压文件包
[root@local-linux02 src]# ls
jdk1.8.0_171                nginx-1.14.0.tar.gz    php-7.2.6.tar.gz
jdk-8u171-linux-x64.tar.gz  nginx-1.14.0.tar.gz.1
nginx-1.14.0                php-7.2.6

//移动重命名
mv jdk1.8.0_171 /usr/local/jdk1.8
```

#### 第四步：配置系统环境变量

系统环境变量的配置文件为 `/etc/profile`，下面要更改系统配置文件。

```
vi /etc/profile

//在末尾增加如下配置：

JAVA_HOME=/usr/local/jdk1.8/
JAVA_BIN=/usr/local/jdk1.8/bin
JRE_HOME=/usr/local/jdk1.8/jre
PATH=$PATH:/usr/local/jdk1.8/bin:/usr/local/jdk1.8/jre/bin
CLASSPATH=/usr/local/jdk1.8/jre/lib:/usr/local/jdk1.8/lib:/usr/local/jdk1.8/jre/lib/charsets.jar 

//保存退出后执行如下命令生效
source /etc/profile
```


#### 校验配置是否正常

![](https://farm2.staticflickr.com/1808/41223584750_3bef8808ff_o.png)


### 安装Tomcat

#### 第一步：下载二进位制安装包


tomcat的官网网址为：https://tomcat.apache.org/

这里下载的镜像站点的二进位制安装包，版本号是8.5.31。同样还是采用在mac下用chrome浏览器下载，之后再通过ftp上传到虚拟机的方式。因为在虚拟机使用wget命令下载实在是太慢了。

#### 第二步：移动文件夹

```
//此处设置了alias ftp='cd /data/ftp/'
[root@local-linux02 src]# ftp
[root@local-linux02 ftp]# ls
apache-tomcat-8.5.31.tar.gz  a.txt  b.txt  test1  test2

//移动文件
[root@local-linux02 ftp]# mv apache-tomcat-8.5.31.tar.gz /usr/local/src/

//进入源码包目录，此处也设置的alias
[root@local-linux02 ftp]# src
[root@local-linux02 src]# ls
apache-tomcat-8.5.31.tar.gz  nginx-1.14.0.tar.gz    php-7.2.6.tar.gz
jdk-8u171-linux-x64.tar.gz   nginx-1.14.0.tar.gz.1
nginx-1.14.0                 php-7.2.6

//解压
[root@local-linux02 src]# tar -zxvf apache-tomcat-8.5.31.tar.gz
```

#### 第三步：安装

由于下载的是二进位制安装包，因此就省去了编译环节。


- 移动文件夹到自定义安装目录

```
[root@local-linux02 src]# mv apache-tomcat-8.5.31 /usr/local/tomcat
```

- 运行启动脚本

```
[root@local-linux02 bin]# /usr/local/tomcat/bin/startup.sh
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk1.8
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.
```
 

- 校验是否成功 `ps -aux |grep java`

![](https://farm2.staticflickr.com/1782/42115998285_deb8c686d2_o.png)

- 检查端口情况 `netstat -lntp |grep java`

![](https://farm2.staticflickr.com/1796/42299096684_7bf6d5eca1_o.png)

说明：正常会有三个以下端口开放

* 8080提供wed服务的端口；

* 8005为管理端口；

* 8009为第三方服务调用端口（如httpd和Tomcat结合时会用到）；


#### 第四步：设置开机启动

```
echo "/usr/local/tomcat/bin/startup.sh" >> /etc/rc.d/rc.local

chmod a+x /etc/rc.d/rc.local
```

#### 第五步：配置iptables端口

```
[root@local-linux02 bin]# vi /etc/sysconfig/iptables

//增加一条
-A INPUT -p tcp -m tcp --dport 8080 -j ACCEPT

[root@local-linux02 bin]# systemctl restart iptables
```

#### 校验成功

浏览器登录

![](https://farm2.staticflickr.com/1800/42116368715_8ee4b3cc42_o.png)

最后，如需关闭tomcat执行如下脚本

```
/usr/local/tomcat/bin/shutdown.sh
```


