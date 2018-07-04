---
title: 'linux学习笔记：lamp的基础配置'
date: 2018-06-01 14:07:23
categories: linux
tags: [linux, LAMP, php, mysql, notes]
---

## 写在前面

按照计划，lamp环境安装配置这部分课程，我打算重新整理成三个部分：

1. [lamp环境的安装](https://www.steve-yuan.com/2018/05/30/linuxNotes-Lamp-0To1/)

2. lamp的基础配置（本地安装discuz及其配置）

3. 网络云主机实践

今天是第二部分的内容，在本地虚拟机安装discuz 并进行相关的配置测试。

## 准备工作：安装discuz

discuz 官网下载包地址：https://gitee.com/ComsenzDiscuz/DiscuzX/attach_files

**注：**使用discuz需要先在官方论坛注册。

### 第一步：新建discuz安装目录

```
mkdir /data/discuz.com

cd /data/discuz.com
```

<!--more-->

### 第二步：下载安装包

```
//下载安装包
wget http://files.git.oschina.net/group1/M00/02/9B/PaAvDFpFnk2AEQzMALUt346QSGo609.ziptoken=1250298e943fe82fcca4870032ddc7d8&ts=1527834100&attname=Discuz_X3.4_GIT_SC_UTF8.zip

//这里遇到一个小错误，文件被重命名成一堆乱码，使用mv命令改一下名字
mv PaAvDFpFnk2AEQzMALUt346QSGo609.zip?token=1250298e943fe82fcca4870032ddc7d8 Discuz_X3.4_GIT_SC_UTF8.zip

//解压
unzip Discuz_X3.4_GIT_SC_UTF8.zip

//移动安装文件（只需移动upload即可）
mv upload/* /data/discuz.com/ 

//删除无用的文件
m -rf dir_SC_UTF8 Discuz_X3.4_GIT_SC_UTF8.zip
```


### 第三步：编辑配置文件

#### 编辑主文件 /usr/local/apache2.4/conf/httpd.conf ：

- 去掉servername前面的#号注释

![](https://farm2.staticflickr.com/1757/41682258574_d7eeb0dcf4_o.png)

- 编辑http主配置文件，并删除vhost这一行的注释#

![](https://farm2.staticflickr.com/1753/40600481960_8926de1eec_o.png)

#### 编辑虚拟机文件

```
[root@stevey ~]# vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

//修改以下内容
<VirtualHost *:80>
    ServerAdmin webmaster@steve-discuz-test.com
    DocumentRoot "/data/discuz.com"   //填写刚才新建的discuz文件目录
    ServerName steve-discuz-test.com   //域名，必须填写
    ServerAlias www.steve-discuz.com   //网站别名
    #ErrorLog "logs/steve.com-error_log"
    #CustomLog "logs/steve.com-access_log" common
</VirtualHost>

```

### 第四步：启动httpd服务

```
//检查错误
/usr/local/apache2.4/bin/apachectl -t

//启动服务
/usr/local/apache2.4/bin/apachectl start

//重新加载
/usr/local/apache2.4/bin/apachectl graceful

```

### 第五步：配置hosts文件

mac的hosts配置文件在/etc/hosts，可以直接修改；这里借助了一个叫做ihosts的软件，方便实验。

![](https://farm2.staticflickr.com/1731/28617032248_8e2212553e_o.png)

### 第六步：浏览器安装

在浏览器输入刚刚的网址steve.com就可以访问测试页面了。这里有一个小窍门，因为笔者的chrome浏览器安装了一个插件，强制所有访问https，所以这里需要手动改成http前缀，就可以正常进行安装了。

![](https://farm2.staticflickr.com/1752/41767013534_f386865b22_o.png)

这里还有一个插件检查，会提示权限错误。

![](https://farm2.staticflickr.com/1747/41587058725_4c92b3f077_o.png)

解决方法：这里的错误主要是由于文件权限不足造成的，我们使用chown目录将文件的owner做一下更改即可。

```
[root@stevey discuz.com]# chown -R daemon config/ data/ uc_client/data/ uc_server/data/ 
```

**注：**这里要使用-R选项，将目录及其字目录的用户名同时更改。

![](https://farm2.staticflickr.com/1747/40682253520_cf4ddc7355_o.png)

选择 全新安装

![](https://farm2.staticflickr.com/1732/42489947501_20ba3f5743_o.png)

这里需要创建数据库、用户

```
//创建数据库
mysql> create database discuz;
//创建用户steve及密码steve-test
mysql> grant all on discuz.* to 'steve'@'localhost' identified by 'steve-test'；
```

之后继续安装，填写上面设置的信息

![](https://farm1.staticflickr.com/878/28617530078_f9145df332_o.png)

看到下面的提示，说明安装成功。

![](https://farm2.staticflickr.com/1743/42490213361_81c9dfa2f7_o.png)

![](https://farm2.staticflickr.com/1749/41587482275_7cfec7d685_o.png)

## lamp 环境的基本配置

### 禁止默认虚拟主机

apache当中的默认虚拟主机，就是配置文件中的第一个主机。假如我们在hosts当中做如下改动，增加一个服务器上并不存在的域名www.thisisateststeve.com，同样会访问steve-discuz-test.com。

```
172.16.155.128 steve-discuz-test.com www.steve-discuz.com www.thisisateststeve.com
```

如下图所示：

![](https://farm2.staticflickr.com/1727/28617711698_c62061dd65_o.png)

这显然不是我们希望看到的效果。为了避免这种情况我们就要禁止默认虚拟主机

实现步骤：

#### 第一步：修改虚拟主机配置

```
<VirtualHost *:80>
     DocumentRoot "/data/dddcom"
     ServerName whatDoYouWant.com
</VirtualHost>

<VirtualHost *:80>
    ServerAdmin webmaster@steve-discuz-test.com
    DocumentRoot "/data/discuz.com"
    ServerName steve-discuz-test.com
    ServerAlias www.steve-discuz.com
    #ErrorLog "logs/steve.com-error_log"
    #CustomLog "logs/steve.com-access_log" common
</VirtualHost>

```

#### 第二步：建立目录

```
//创建目录
[root@stevey discuz.com]# mkdir -p /data/dddcom
//设置权限，不允许除root用户之外的人访问
[root@stevey discuz.com]# chmod 600 /data/dddcom
```

#### 第三步：重新加载服务

```
[root@stevey discuz.com]# /usr/local/apache2.4/bin/apachectl -t
Syntax OK
[root@stevey discuz.com]# /usr/local/apache2.4/bin/apachectl graceful

```

#### 成果检测

![](https://farm2.staticflickr.com/1730/27620081727_270a9d6d3c_o.png)

### 域名跳转

这样做的好处：

* 增加SEO权重；

* 如果老的域名被放弃了，可以很方便地重定向到新的域名；

实现步骤如下：

#### 第一步：编辑虚拟主机配置文件

```
<VirtualHost *:80>
    ServerAdmin webmaster@steve-discuz-test.com
    DocumentRoot "/data/discuz.com"
    ServerName steve-discuz-test.com
    ServerAlias www.steve-aaa.com
    ServerAlias www.steve-bbb.com
    #ErrorLog "logs/steve.com-error_log"
    #CustomLog "logs/steve.com-access_log" common
    <IfModule mod_rewrite.c>    //使用rewrite模块
        RewriteEngine on
        以下两行是条件 [OR]表示或者
        RewriteCond %{HTTP_HOST} !^www.steve-aaa.com$ [OR]
        RewriteCond %{HTTP_HOST} !^www.steve-bbb.com$
        RewriteRule ^/(.*)$ http://steve-discuz-test.com/$1 [R=301,L] 这里使用的是301永久跳转
    </IfModule>
</VirtualHost>
```

#### 第二步：修改apache主配置文件，加载rewrite模块

```
//检测是否已加载rewrite模块
[root@stevey ~]# /usr/local/apache2.4/bin/apachectl -M | grep -i rewrite   //没有输出结果，说明未加载

//编辑主配置文件
vim /usr/local/apache2.4/conf/httpd.conf 

//找到rewrite，把前面注释#去掉
LoadModule rewrite_module modules/mod_rewrite.so
```


#### 第三步：重新加载模块

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

#### 成果检验

![](https://farm2.staticflickr.com/1749/41588492855_7fc3f05d65_o.png)

### 访问日志切割

#### 修改日志设置

这里是利用了apache的rotatelogs命令对文件进行切割

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    ServerAdmin webmaster@steve-discuz-test.com
    DocumentRoot "/data/discuz.com"
    ServerName steve-discuz-test.com
    ServerAlias www.steve-aaa.com
    ServerAlias www.steve-bbb.com
    ErrorLog "logs/steve-discuz-test.com-error_log"
    //定义访问日志切割
    CustomLog "|/usr/local/apache2.4/bin/rotatelogs -l logs/steve-discuz-test.com-access_%Y%m%d.log 86400" combined env=!img-request    
    <IfModule mod_rewrite.c>
        RewriteEngine on
        RewriteCond %{HTTP_HOST} !^www.steve-aaa.com$ [OR]
        RewriteCond %{HTTP_HOST} !^www.steve-bbb.com$
        RewriteRule ^/(.*)$ http://steve-discuz-test.com/$1 [R=301,L]
    </IfModule>
</VirtualHost>

```

注：这里使用了管道符，将文件交由rotatelogs命令按照时间单位进行切割，86400单位是秒，相当于一天保存一个文档；

#### 访问日志查看

日志文档保存在 /usr/local/apache2.4/logs 目录下，打开盖文件夹就能看到刚刚的访问记录

![](https://farm1.staticflickr.com/901/28618980208_54c7b13b50_o.png)


使用cat 命令就可以查看日志

![](https://farm2.staticflickr.com/1756/27621164977_03a1f90fc6_o.png)

#### 访问日志不记录指定类型的静态文件

这样做的目的是控制访问日志的大小，便于后期的查询维护。

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    ServerAdmin webmaster@steve-discuz-test.com
    DocumentRoot "/data/discuz.com"
    ServerName steve-discuz-test.com
    ServerAlias www.steve-aaa.com
    ServerAlias www.steve-bbb.com
    ErrorLog "logs/steve-discuz-test.com-error_log"
    //不记录指定类型的静态文件
    SetEnvIf Request_URI ".*\.gif$" img-request
    SetEnvIf Request_URI ".*\.jpg$" img-request
    SetEnvIf Request_URI ".*\.png$" img-request
    SetEnvIf Request_URI ".*\.bmp$" img-request
    SetEnvIf Request_URI ".*\.swf$" img-request
    SetEnvIf Request_URI ".*\.js$" img-request
    SetEnvIf Request_URI ".*\.css$" img-request
    CustomLog "|/usr/local/apache2.4/bin/rotatelogs -l logs/steve-discuz-test.com-access_%Y%m%d.log 86400" combined env=!img-request    
    <IfModule mod_rewrite.c>
        RewriteEngine on
        RewriteCond %{HTTP_HOST} !^www.steve-aaa.com$ [OR]
        RewriteCond %{HTTP_HOST} !^www.steve-bbb.com$
        RewriteRule ^/(.*)$ http://steve-discuz-test.com/$1 [R=301,L]
    </IfModule>
</VirtualHost>
```

#### 设置静态缓存

设置静态缓存的目的是优化网页设置，提升访问体验。

##### 第一步：修改主机配置


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

##### 第二步：修改apache主配置文件，加载expires模块

```
vim /usr/local/apache2.4/conf/httpd.conf

//取消注释
LoadModule expires_module modules/mod_expires.so
```

##### 第三步：重启加载apache服务

```
//检查语法错误
[root@stevey ~]# /usr/local/apache2.4/bin/apachectl -t

//重新加载配置文件
[root@stevey ~]# /usr/local/apache2.4/bin/apachectl graceful

//查看是否加载模块
[root@gary-tao local]# /usr/local/apache2.4/bin/apachectl -M|grep -i rewrite  

```

#### 防盗链

**防盗链**是针对盗链行为的一种预防策略，就是通过设置黑名单或者白名单，有选择性地对允许或者拒绝。

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/discuz.com"
    ServerName steve-discuz-test.com
    ServerAlias www.steve-aaa.com
    ServerAlias www.steve-bbb.com
    CustomLog "logs/abc.com-access_log" combined
    <Directory /data/discuz.com>   //定义防盗链目录
        SetEnvNoCase Referer "http://steve-discuz-test.com" local_ref [NC]
        SetEnvNoCase Referer "http://www.steve-aaa.com" local_ref  [NC]
        SetEnvNoCase Referer "http://www.steve-bbb.com" local_ref  [NC]
        SetEnvNoCase Referer "^$" local_ref
        <filesmatch "\.(txt|doc|mp3|zip|rar|jpg|gif|pdf)">
            Order allow,deny
            Allow from env=local_ref
        </filesmatch>
    </Directory>
</VirtualHost>
```

#### 限制ip

配置方法：

```
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/discuz.com"
    ServerName steve-discuz-test.com
    ServerAlias www.steve-aaa.com
    ServerAlias www.steve-bbb.com
    CustomLog "logs/abc.com-access_log" combined
    <Directory /data/discuz.com>   //定义防盗链目录
        Order deny,allow    //默认拒绝  
        Deny from all       //先拒绝所有访问请求
        Allow from 127.0.0.1   //再对指定ip开放
        <filesmatch "(.*)admin(.*)">   //admin请求只允许在本地访问
            Order deny,allow    //默认拒绝  
            Deny from all       //先拒绝所有访问请求
            Allow from 127.0.0.1   //再对指定ip开放
        </filesmatch>
    </Directory>
</VirtualHost>
```

#### 安全设置：限制php解析

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

#### 限制user_agent

``` 
vim /usr/local/apache2.4/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/data/discuz.com"
    ServerName steve-discuz-test.com
    ServerAlias www.steve-aaa.com
    ServerAlias www.steve-bbb.com
    CustomLog "logs/discuz.com-access_log" combined
    <Directory /data/discuz.com>   //定义目录
        RewriteEngine on
        RewriteCond %{HTTP_USER_AGENT} ^.*curl.* [NC,OR]
        RewriteCond %{HTTP_USER_AGENT} ^.*baidu.com.* [NC]
        RewriteRule .* - [F]
    </Directory>
</VirtualHost>
```



