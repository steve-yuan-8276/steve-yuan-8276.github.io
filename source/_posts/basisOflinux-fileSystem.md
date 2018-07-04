---
title: 'linux笔记：用户权限和文件权限'
date: 2018-06-29 15:24:20
categories: linux
tags: [linux, centos, notes]
---

## 写在前面

linux是一个多用户、多任务系统，具有很好的稳定性和安全性，而这与linux的用户和权限管理机制密不可分。

我们在linux系统中的操作，实际上都是操作进程访问文件，这里的访问权限是基于Linux的安全管理机制的。linux上最初的安全模型，叫做DAC，`Discretionary Access Contorl`，主动访问控制；后来在此基础上又增加了额外的权限管理机制，成为MAC，`Mandatory Access Contorl`，强制访问机制。为了区分两者，将支持MAC的linux系统成为selinux，意思是linux的安全加强系统。

在实际运行过程中，系统会先做DAC检查，如不通过则拒绝执行；如果通过DAC检查且支持MAC模块，则再做MAC检查。

这里整理的用户和权限管理，主要是针对DAC而言的。


## 用户权限管理

### 用户权限概念

#### `user 用户：`

在linux当中，每个用户有一个唯一的uid (user identification)用来标示用户身份，这个道理大概就跟每个中国人有一个唯一的身份证号码一样。

在centos 7 中，有以下三种用户：

身份|uid |说明
---|---|---
系统管理员|0|系统超级管理员，即root，拥有最高权限
系统用户|1-999|系统预留，linux默认服务程序都会有独立的系统用户进行运行，这部分账户由系统预留，我们设置普通账户的时候，不能将uid设为这个数字段
普通用户|1000以上|系统普通用户，系统权限受限，由root用户管理，uid从1000开始

#### `group 组：`

group的概念，跟班级的概念类似，主要是方便Linux系统权限管理。每个用户只能从属于一个基本组，但可以加入多个拓展组。做个类别，不如张三是三年二班（基本组）的学生，但可以同时加入天文组、计算机组、唱诗班（拓展组），大概是这么个意思。
 
### 用户权限管理的配置文件

* `/etc/passwd ` 存放用户信息，每条信息依次为：用户名/口令（这里只有星号，真正的密码在`/etc/shadow`中）/uid/gid（默认与uid相同）/描述注释/home目录/登陆shell；

* `/etc/group` 存放组信息，每条信息依次为：组名/口令/gid/组成员列表；

* `/etc/shadow`，存放密码信息，每条信息依次为：登录名/加密口令/最后一次修改时间/最小修改间隔/最大修改间隔/警告时间/不活动时间；

### 用户管理相关的命令

#### `useradd` 命令 ：新增用户

命令行格式 

```
useradd [options] [username]
```

常用参数

```
//-d 指定用户家目录
useradd -d /home/steve  steve

//-u 指定uid
useradd -u 1000 steve

//-g 指定所属组
useradd -g steve steve

//-G 指定拓展组
useradd -G steve,IT,music  steve

//-s 指定登陆shell，/sbin/nogin 表示不允许登陆
useradd -s /sbin/nologin steve
```

#### groupadd 命令：增加组

命令行格式：

```
groupadd [options] [groupname]
```

常见用法：

```
//增加一个名为work的组
groupadd work  
```

#### userdel 命令：删除用户

命令行格式：

```
userdel [options] [username]
```

常见用法：

```
//-f 选项，强制删除某用户
userdel -f steve

//-r 选项，同时删除用户及其家目录
userdel -r steve
```


#### passwd命令： 设置密码

命令行格式：

```
passwd [options] [uername]
```
 
常见用法

```
//直接输入passwd，按照提示输入两次密码，即可设施密码
[root@local-linux02 ~]# passwd

//--stdin 参数，只需一行更改密码
//newpasswd表示需要设置的新密码
echo 'newpasswd' | passwd --stdin username

//-l 表示锁定用户，禁止该用户登陆
passwd -l steve

//-u 表示解锁，允许登陆
passwd -u steve
```



## 文件权限管理

在linux当中，一切都是文件。文件主要包括以下三个部分：

1. 普通权限

2. 特殊权限

3. 隐藏权限

### linux文件目录结构：

![](https://farm1.staticflickr.com/787/40206810334_bb89977f05_o.jpg)

### 常用文件类型：

文件类型| 描述 |说明
---|---|---
-  |普通文件，包括文本文件|可用touch命令创建
d  |文件目录|可用 mkdir 命令创建
l  |链接文件(指向另一个文件,类似于windows下的快捷方式)|可用ln 命令创建
s  |套接字文件，用于进程通信|一般由应用程序在执行过程中创建
b  |块设备文件,二进制文件
c  |字符设备文件，如鼠标、键盘、终端
p  |管道文件|可用mkfifo命令创建


### 文件基本权限

文件的基本权限包括读read、写write和执行execute，它与用户密不可分，分为三组进行管理：

* `user：`对文件/目录所属用户设定权限；

* `group：`对文件/目录所属组设定权限；

* `other：`对文件/用户其他的组和用户设定权限；

##### 对于文件而言：

* Read: 表示可读取文件内容；

* Write：表示可编辑、新增、修改、删除文件内容；

* execute：表示可执行一个脚本程序；

##### 对于目录而言：

* Read: 表示可读取目录内的文件列表；

* Write：表示可在目录内编辑、新增、修改、删除文件内容；

* execute：表示能够进入目录；

#### 权限的数字表示方法：

read读、write写、execute执行，分别可以用4、2、1来表示；

### 文件的特殊权限

#### `suid` 

这是一种对二进位制程序设置的特殊权限，临时允许二进位制程序执行者用户文件所属主的权限，这是一种临时的、特殊的、有条件的授权方法。

例如，前文提到的/etc/shadow 这个文件，用来保存用户信息，处于系统安全的考虑，这个文件读、写、执行的权限都没有；但是passwd命令拥有suid的特殊权限位，如下图所示，出现在x位置的那个s，即表示suid，这就好像尚方宝剑，赋予它临时修改密码配置文件，也就是更改密码的权限。

![](https://farm1.staticflickr.com/846/29281844568_90c20b3e97_o.png)

#### sgid

sgid 与suid相似，有一个重要特性，就是一旦对某个目录设置了sgid，该目录中创建的文件将继承该目录的所属组。

这就意味着如果我们要创建一个部门共享目录，可以早创建目录后，设置sgid权限，这样部门群组内的任何用户创建的文件都会属于该目录所属组。如此一来，该部门所有用户都能够顺利读取共享目录文件。








