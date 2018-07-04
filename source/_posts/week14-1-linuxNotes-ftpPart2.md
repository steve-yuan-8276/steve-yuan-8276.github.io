---
title: 'linux学习笔记：ftp的架设使用'
date: 2018-06-25 09:47:14
categories: linux
tags: [linux, ftp, notes]
---


## 写在前面

接着上回的内容继续进行整理，还是ftp的相关知识。

## ftp的主动模式和被动模式

在ftp传输中，均包括命令连接和数据传输两个部分，根据传输方式的不同，分为以下两种：

### ftp被动模式

* 命令连接：客户端 >1024端口 -> 服务器 21端口 

* 数据连接：客户端 >1024端口 -> 服务器 >1024端口

被动模式是**FTP服务器返回数据传输需要的端口，FTP客户端去连接FTP服务端**。目前，绝大部分的互联网应用(比如Web/Http)，都是客户端向服务端发起连接。


![](https://farm2.staticflickr.com/1789/42942630462_e5b41054fa_o.jpg)

<!--more-->

#### 被动模式下iptables规则配置

- 修改vsftpd配置文件

```
vi /etc/vsftpd/vsftpd.conf

pasv_enable=YES
pasv_min_port=2222
pasv_max_port=2225
```

- iptables配置端口

```
vi /etc/vsftpd/vsftpd.conf

//添加规则
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 21 -j ACCEPT
-A INPUT -p tcp --dport 2222:2225 -j ACCEPT
```

### ftp主动模式

命令连接：客户端 >1024端口 -> 服务器 21端口  

数据连接：客户端 >1024端口 <- 服务器 20端口 

**主动模式是FTP客户端向FTP服务器发送数据传输需要的端口，FTP服务端去连接FTP客户端的端口，与被动模式刚好相反。**

![](https://farm2.staticflickr.com/1791/28124191167_eec09e019f_o.jpg)

#### 主动模式下`iptables`设置原则

```
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

iptables -A INPUT -p tcp  -m multiport --dport 20,21  -m state --state NEW -j ACCEPT
```

说明：

* `NEW`: 该包想要开始一个新的连接（重新连接或连接重定向）；

* `RELATED`:该包是属于某个已经建立的连接所建立的新连接。如FTP的数据传输连接和控制连接之间就是RELATED关系；

* `ESTABLISHED`：该包属于某个已经建立的连接；

* `INVALID`:该包不匹配于任何连接，通常这些包被DROP；

## 使用pure-ftpd搭建ftp服务

### 第一步：安装pure-ftpd

```
//安装拓展源
yum install -y epel-release

//安装pure-ftpd
yum install -y pure-ftpd
```


### 第二步：关闭vsftpd

**注：**由于在之前的实验中开启的vsftpd，为了避免端口冲突，需要先关闭vsftpd

```
//关闭vsftpd
systemctl stop vsftpd

//禁止vsftpd开机启动
systemctl disable vsftpd
```

- 校验结果，查看vsftpd状态

![](https://farm2.staticflickr.com/1784/42943085652_6424f80169_o.png)

### 第三步：编辑pure-ftp配置文件

```
vi /etc/pure-ftpd/pure-ftpd.conf

//查找PureDB /etc/pure-ftpd/pureftpd.pdb，将前面的#删除
```

![](https://farm2.staticflickr.com/1798/42274322624_58c0685df5_o.png)

- 启动pure-ftp

```
[root@local-linux02 ~]# systemctl start pure-ftpd

[root@local-linux02 ~]# systemctl enable pure-ftpd
Created symlink from /etc/systemd/system/multi-user.target.wants/pure-ftpd.service to /usr/lib/systemd/system/pure-ftpd.service.
```

- 校验结果

![](https://farm2.staticflickr.com/1814/41182572270_4a9a5296d0_o.png)


### 第四步：建立虚拟账号

**注：安全起见，pure-ftpd使用的账号并非linux的系统账号，而是虚拟账号。**

```
//建立ftp文件夹
[root@local-linux02 ~]# mkdir -p /data/ftp

//建立系统账户
[root@local-linux02 ~]# useradd -u 1010 pure-ftp

//更改ftp文件夹的所属用户和组
[root@local-linux02 ~]# chown -R pure-ftp:pure-ftp /data/ftp


//创建虚拟账户；
//-u是将虚拟用户ftp_usera与系统用户pure-ftp关联在一起，使用ftp_usera账号登录ftp后，会以pure-ftp的身份来读取和下载文件；
//-d是指定ftp_usera账户的家目录，这样可以使用户ftp_usera只能访问其家目录/data/ftp/
//提示，此处需要输入密码
[root@local-linux02 ~]# pure-pw useradd ftp_usera -u pure-ftp  -d /data/ftp
Password:
Enter it again:


//创建用户信息数据库文件
[root@local-linux02 ~]# pure-pw mkdb
```

**账号常见操作：**

```
//列出当前账号
[root@local-linux02 ~]# pure-pw list
ftp_usera           /data/ftp/./

//删除账号
pure-pw userdel ftp_usera  
```

### 第五步：测试连接

命令行格式：

```
lftp ftp://user:password@ip:port

其中，
- ftp:// 可省略；
- user 登录名；
- password 登陆密码，此处可不写，在后面交互界面输入；
- ip 即服务器端的ip地址；
- port 默认为21，也可另行指定。

```

示例：

![](https://farm2.staticflickr.com/1779/42294559944_a5025f80d1_o.png)

TroubleShooting: 链接错误

如下所示，虽然看起来连上了，但是执行ls命令的时候，却一直显示重新链接的状态。

![](https://farm2.staticflickr.com/1807/29140686268_e2b220634b_o.png)

**排查思路：**

**首先，**排查pure-ftpd服务器端是否启动，执行 `systemctl status   pure-ftpd` 是正常的，runing状态。

**其次，**估计是端口设置的问题，google了一下pure-ftpd被动模式端口设置的方法：

- 第一步：修改pure-ftpd配置文件

```
//查找pure0-ftpd配置文件
[root@local-linux02 ~]# find / -name 'pure-ftpd'
/run/pure-ftpd
/etc/pki/pure-ftpd
/etc/logrotate.d/pure-ftpd
/etc/xinetd.d/pure-ftpd
/etc/pam.d/pure-ftpd
/etc/pure-ftpd
/usr/sbin/pure-ftpd
/home/sosreport/sosreport/etc/logrotate.d/pure-ftpd
/home/sosreport/sosreport/etc/xinetd.d/pure-ftpd
/home/sosreport/sosreport/etc/pam.d/pure-ftpd


//进入配置文件夹
[root@local-linux02 pure-ftpd]# cd /etc/pure-ftpd
[root@local-linux02 pure-ftpd]# ls
pure-ftpd.conf      pureftpd-mysql.conf  pureftpd.pdb
pureftpd-ldap.conf  pureftpd.passwd      pureftpd-pgsql.conf

//编辑配置文件
[root@local-linux02 pure-ftpd]# vi pure-ftpd.conf

//找到#PassivePortRange 这一行，去掉注释，改为
PassivePortRange 20000 30000

//保存退出后，重启pure-ftpd服务
systemctl restart pure-ftpd
```

- 第二步：设置防火墙端口

```
iptabels -A INPUT -p tcp -m state --state NEW -m tcp --dport 21 -j ACCEPT

iptabels -A INPUT -p tcp --dport 20000:30000 -j ACCEPT

service iptables save
```

**个人更推荐第二种方法：直接修改`/etc/sysconfig/iptables`，将上述两行规则加入，保存退出并重启虚拟机。**


- 校验结果：再尝试重新连接ftp服务器

```
//客户端操作
[root@local-linux01 ~]# lftp ftp_usera@172.16.155.132
Password:
lftp ftp_usera@172.16.155.132:~> ls
drwxr-xr-x    2 1010       pure-ftp            6 Jun 26 08:48 .
drwxr-xr-x    2 1010       pure-ftp            6 Jun 26 08:48 ..
//测试上传一个文件
lftp ftp_usera@172.16.155.132:/> put -c ~/test/a.txt
26 bytes transferred

//服务器端测试
//存在a.txt，说明上传成功
[root@local-linux02 ~]# ll /data/ftp
total 4
-rw-r--r-- 1 pure-ftp pure-ftp 26 Jun 14 14:33 a.txt
```

### lftp的常见命令

#### ls 和 cd命令

- `ls` 命令用来列出文件和目录；

```
lftp ftp_usera@172.16.155.132:/> ls
drwxr-xr-x    4 1010       pure-ftp           45 Jun 26 10:25 .
drwxr-xr-x    4 1010       pure-ftp           45 Jun 26 10:25 ..
-rw-r--r--    1 1010       pure-ftp           26 Jun 14 14:33 a.txt
drwxr-xr-x    2 0          0                   6 Jun 26 10:25 test1
drwxr-xr-x    2 0          0                   6 Jun 26 10:25 test2
```

- `cd` 用来进入某个目录；

```
lftp ftp_usera@172.16.155.132:/> cd test1
lftp ftp_usera@172.16.155.132:/test1> pwd
ftp://ftp_usera@172.16.155.132/test1
```

#### lpwd 和 lcd：显示和切换本地文件夹

**注：local dir 本地文件夹，就是当前下载文件的保存目录，我们可以通过这两个目录来查看和修改。**

lpwd：显示目前的本地文件夹 local dir

```
lftp ftp_usera@172.16.155.132:~> lpwd
/root
```

lcd：切换文件夹

```
lftp ftp_usera@172.16.155.132:~> lcd /root/ftpsync/
lcd ok, local cwd=/root/ftpsync

```

#### 下载文件：get 和 mget

get：下载单个文件

```
//默认下载到当前用户的home目录下
lftp ftp_usera@172.16.155.132:/> get -c a.txt
```

mget: 下载多个文件

```
lftp ftp_usera@172.16.155.132:/> get -c a.txt b.txt
```

#### 上传文件：put、mput

- `put`: 上传单独文件

```
//put常用选项
//-c 选项，表示断点续传
//-O 选项，表示指定文件存放路径
lftp ftp_usera@172.16.155.132:/> put -c ~/test/a.txt
26 bytes transferred
```

- `mput`：上传多个文件

```
lftp ftp_usera@172.16.155.132:/> put -c ~/test/a.txt ~/test/b.txt
```

#### 同步目录

- 下载目录`mirror [dir]`：从服务器端下载到客户端

```
lftp ftp_usera@172.16.155.132:/> mirror test1
```

- 上传目录`mirror -R [dir]`:从客户端到服务器端

```
lftp ftp_usera@172.16.155.132:/> mirror -R /rrot/test1
```

#### 模式设置

```
//设置字符编码
set ftp:charset gbk/utf-8

//设置被动模式
set ftp:passive-mode 1
```


## 参考资料

* [lftp：优秀的文件客户端程序](http://wangchujiang.com/linux-command/c/lftp.html)

* [linux使用命令行操作ftp](https://www.jianshu.com/p/8aab3a1016f7)

* [使用 lftp 进行 FTP 数据下载和上传](http://www.chenlianfu.com/?p=2321)

* [lftp 使用方法](https://havee.me/linux/2010-08/how-to-use-lftp.html)

