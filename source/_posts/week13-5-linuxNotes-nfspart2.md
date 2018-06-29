---
title: 'linux学习笔记13-5：NFS的基本使用（二）'
date: 2018-06-22 10:24:25
categories: linux
tags: [linux, mysql, notes]
---


##写在前面

这则笔记继续分两部分：一是继续整理nfs相关操作；二是开始整理ftp相关知识。

## NFS挂载补充

### exportfs命令

前面已经配置好了nfs文件夹共享服务，但是这里还有一个隐患，就是当服务器端端配置更新改动的时候，就需要以restart的方式使改动生效。但是，在生产环境中，频繁重启nfs可能带来不必要麻烦和错误，这时，就需要借助exportnfs命令来实现不重启，但却能重新加载服务的目的。

#### 安装exportfs

```
yum install -y exportnfs
```

#### 语法

命令行格式

```
exportnfs [参数]
```

常用参数如下：

* `-a`：全部挂载（或卸载）`/etc/exports`文件内的设定；

* `-r`：重新挂载`/etc/exports`中的设置，此外同步更新/`etc/exports`及`/var/lib/nfs/xtab`中的内容；

* `-u`：卸载某一目录；

* `-v`：在export时将共享的目录显示在屏幕上；

<!--more-->

#### 示例

##### 第一步：新建一个共享目录

```
//新建共享文件夹
[root@local-linux02 ~]# mkdir -p /tmp/share2

//更改目录为nfs配置指定用户，也就是下面uid1000对应的user1
[root@local-linux02 ~]# chown -R user1:user1 /tmp/share2
```

##### 第二步：修改nfs配置，增加一个共享目录；

```
vi /etc/exports
//新增一行
/tmp/share2/ 172.16.155.128(rw,sync,all_squash,anonuid=1000,anongid=1000)
```

##### 第三步：重新加载服务（服务器端执行）

```
//服务器端执行
[root@local-linux02 ~]# exportfs -arv
exporting 172.16.155.128:/tmp/share2
exporting 172.16.155.128:/home/sharedir
```

##### 第四步：挂载服务

```
//创建挂载点
[root@local-linux01 ~]# mkdir -p /mnt/sharedir2

//挂载共享文件夹
[root@local-linux01 ~]# mount -t nfs -o nolock 172.16.155.132:/tmp/share2 /mnt/sharedir2
```

##### 校验结果

```
//客户端操作
[root@local-linux01 sharedir2]# touch 2.txt
[root@local-linux01 sharedir2]# echo 'bingo' > 2.txt

//服务器端操作
[root@local-linux02 share2]# cat /tmp/share2/2.txt
bingo
```

### NFS客户端问题及其解决办法

#### NFS的版本更迭

* `NFS V2` NFS最早实现的版本之一，基于UDP协议实现了一个无状态的服务器版本。仅仅支持32位的系统，且不大于2GB的文件；

* `NFS V3` 在2的基础之上做了大量的改进。支持了大于2GB的文件读写，使用了TCP协议来进行数据交互，支持了客户端的异步读写来提高文件系统的性能（同时也会产生我们头疼的一致性问题）；

* `NFS V4` 进一步提高了安全性，通过TCP协议实现了一个有状态的服务器版本，通过锁租约的机制来实现多客户端的读写同步。在4.1版本引入了pNFS，通过类似于一个HDFS架构来提供并行的一个分布式文件系统。

**问题描述：**在NFS4版本，客户端挂载共享目录后，不管是root用户还是普通用户，创建新文件时属主和属组都是nobody。

**解决思路：**

**方法一：**客户端挂载时加上-o nfsvers=3

```
[root@local-linux01 ~]# mount -t nfs -o nfsvers=3 172.16.155.132:/tmp/share2 /mnt/sharedir2
```

**方法二：**客户端服务端同时进行操作

```
vim /etc/idmapd.conf 

Domain = local.domain.edu 改为 Domain = xxx.com

//保存并重启rpcidmapd服务
service rpcidmapd restart
```

#### 客户端自动挂载NFS共享目录

##### 方法一：客户端将挂载命令写入/etc/profile

```

vim /etc/profile
//写入挂载命令
mount  -t nfs -o nolock 172.16.155.132:/tmp/share2 /mnt/sharedir2
```


##### 方法二：将要挂载的NFS目录写在客户端的/etc/fatab文件中，挂载时使用mount -a

```
vim /etc/fstab

//加入以下内容
172.16.155.132:/tmp/share2 /mnt/sharedir2 nfs    default,nolock  0  0
```


注意：**建议使用第一种。**因为如果NFS服务未启动，第二种方法可能会导致客户端无法开机。


## 搭建 FTP 文件服务

vsftpd是一款在Linux发行版中最受推崇的FTP服务器程序，全称是“very secure FTP daemon”，它可以运行在诸如 Linux、BSD、Solaris、 HP-UNIX等系统上，是一个完全免费的、开发源代码的ftp服务器软件，支持很多其他的 FTP 服务器所不支持的特征。比如：非常高的安全性需求、带宽限制、良好的可伸缩性、可创建虚拟用户、支持IPv6、速率高等。

### 安装vsftpd

```
//db4-utils 用来生成密码库文件
yum install -y vsftpd db4-utils
```

### 账号登陆

#### 第一步：建立与虚拟账号对应的系统账号

```
[root@local-linux02 ~]# useradd virftp -s /sbin/nologin
```

#### 第二步：修改虚拟账户密码文件

```
[root@local-linux02]#vim /etc/vsftpd/vsftpd_login

//内容如下：每两行为一对，奇数行为用户名，偶数行为密码
testuser1
12345678
```

#### 第三步：更改密码文件权限

```
[root@local-linux02 ~]# chmod 600 /etc/vsftpd/vsftpd_login
```

#### 第四步：生成对应的密码库文件

```
db_load -T -t hash -f /etc/vsftpd/vsftpd_login /etc/vsftpd/vsftpd_login.db
```

**说明：**

* `-T` 允许应用程序能够将文本文件转译载入数据库；

* `-t` hash使用hash码加密；

* `-f` 指定包含用户名和密码文本文件。此文件格式：奇数行用户名、偶数行密码；


#### 第五步：设置虚拟账户配置文件

```
//虚拟账户配置文件夹
[root@local-linux02 ~]# mkdir -p /etc/vsftpd/vsftpd_user_conf

//进入虚拟账户配置文件夹
[root@local-linux02 ~]# cd /etc/vsftpd/vsftpd_user_conf
[root@local-linux02 vsftpd_user_conf]#

//每个虚拟账户建立一个独立的配置文件
vi testuser1

//添加以下内容
//指定用户家目录
local_root=/home/virftp/testuser1
//指定是否允许匿名登陆
anonymous_enable=NO
//表示是否可写
write_enable=YES
//指定umask值
local_umask=022
//是否允许匿名账户上传文件
anon_upload_enable=NO
//匿名账户是否具有写权限
anon_mkdir_write_enable=NO
idle_session_timeout=600
data_connection_timeout=120
max_clients=10
```

![](https://farm1.staticflickr.com/883/41172806780_5dc31a326d_o.png)


#### 第六步：在virftp家目录下创建以用户名命名的家目录：

- 创建目录

```
mkdir /home/virftp/testuser1

touch /home/virftp/testuser1/test1.txt

chown -R virftp:virftp /home/virftp
```

- 修改配置文件

```
vi /etc/pam.d/vsftpd 

//在最前面加上
auth sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpd_login

account sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpd_login
```

#### 第七步：修改全局配置文件`/etc/vsftpd/vsftpd.conf`:

```
vi /etc/vsftpd/vsftpd.conf

//以下三项yes，改为no
anonymous_enable=YES
anon_upload_enable=YES
anon_mkdir_write_enable=YES

//去掉下面一行的注释符号：
chroot_local_user=YES

//新增以下内容
guest_enable=YES
guest_username=virftp
virtual_use_local_privs=YES
user_config_dir=/etc/vsftpd/vsftpd_user_conf
allow_writeable_chroot=YES
```

#### 第八步：启动ftp服务

```
systemctl start vsftpd
```

#### 校验结果

```
//检查是否启动
[root@local-linux02 vsftpd_user_conf]# ps -aux | grep vsftpd
root       3226  0.0  0.0  53260   576 ?        Ss   22:13   0:00 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf
root       3229  0.0  0.0 112704   976 pts/0    S+   22:14   0:00 grep --color=auto vsftpd
```

![](https://farm1.staticflickr.com/899/42081482315_4fda33fe0d_o.png)


#### 尝试连接

##### 安装ftp客户端

```
[root@local-linux01 sharedir2]# yum install -y lftp
```

lftp命令是一款优秀的文件客户端程序，它支持ftp、SETP、HTTP和FTPs等多种文件传输协议。lftp支持tab自动补全，记不得命令双击tab键，就可以看到可能的选项了。

**命令行格式：**

```
lftp [选项] [访问地址ip或域名]
```

**常用选项：**

* `-f`：指定lftp指令要执行的脚本文件；

* `-c`：执行指定的命令后退出；

* `--help`：显示帮助信息；

* `--version`：显示指令的版本号；

##### 测试链接

![](https://farm1.staticflickr.com/899/42264826064_0ba293a9df_o.png)


