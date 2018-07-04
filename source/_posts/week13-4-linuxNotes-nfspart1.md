---
title: 'linux学习笔记13-4：NFS的基本使用（一）'
date: 2018-06-21 14:13:26
categories: linux
tags: [linux, mysql, notes]
---


## 写在前面

这则笔记开始，整理NFS服务的相关知识。

## 什么是NFS？

NFS 是Network File System的缩写，即网络文件系统。一种使用于分散式文件系统的协定，由Sun公司开发，于1984年向外公布。**它独立于操作系统，通过网络让不同的机器、不同的操作系统能够彼此分享数据。** 

比如，NFS 服务器可以让你将网络远程的 NFS 服务器分享的目录，挂载到本地主机上，在本地端的机器看起来，那个远程主机的目录就好像是自己的一个磁盘分区槽一样。

对于一个大型网站而言，之所以需要NFS，主要是基于两点需求：

1. 实现数据信息共享（数据库内容）；

2. 保持数据信息一致 （网站代码、展示内容）；

#### NFS共享系统原理

![](https://farm2.staticflickr.com/1782/42058712455_71b07c3df0_o.jpg)

<!--more-->

1. 在NFS服务端创建共享目录；

2. 通过mount网络挂载，将NFS客户端本地目录挂载到NFS服务端共享目录上；

3. NFS客户端挂载目录上创建、删除、查看数据操作，等价于在服务端进行的创建 删除 查看操作；  


#### NFS的工作流程

1. 由程序在NFS客户端发起存取文件的请求，客户端本地的RPC(rpcbind)服务会通过网络向NFS服务端的RPC的111端口发出文件存取功能的请求。`

2. NFS服务端的RPC找到对应已注册的NFS端口，通知客户端RPC服务。

3. 客户端获取正确的端口，并与NFS daemon联机存取数据。

4. 存取数据成功后，返回前端访问程序，完成一次存取操作。


### 准备工作：准备两台虚拟机

因为nfs是在两台以上的机器之间共享文件的，因此，要完成这个实验，必须先至少有两台机器。这里的采用的方法是克隆一台虚拟机。将原来的虚拟机作为客户端local-Linux01，克隆虚拟机作为服务器端local-linux02.

#### 第一步：克隆虚拟机：点击菜单栏 virtual machine-> create full clone

**注意：**这里有两个前提，负责无法创建快照。

1. 把所有的快照snapshots都删除

2. 虚拟机shutdown，

#### 第二步：修改静态ip地址

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33

//将静态ip改为172.16.155.132，之后保存退出

systemctl restart network.service
```

-  使用ping命 令检查网络是否联通

![](https://farm2.staticflickr.com/1838/42910911962_4ab708e5ec_o.png)

####  第三步：修改主机名 
 
这一步不是必须 ，不过原来的虚拟机和克隆主机的hostname目前都是一样的，区分起来比较费劲，所以将原主机改为`local-linux01` ， 克隆主机改为` local-lin  ux02`，好进行区分。


```
//原虚拟机hostname改为 local-linux01
steveyuan@steve:~|⇒  hostnamectl set-hostname local-linux01


//克隆虚拟机hostname改为 local-linux02
steveyuan@steve:~|⇒  hostnamectl set-hostname local-linux02
```

#### 第四步：配置虚拟机ssh登陆

##### 1. 编辑配置文件 

```
vi /etc/ssh/sshd_config

//去掉下面两行的井号注释
#PubkeyAuthentication yes
#AuthorizedKeysFile .ssh/authorized_keys

然后，修改下面的内容的 yes 为 no

PasswordAuthentication yes
```

##### 2. 生成密匙对（在虚拟机一上操作）

```
ssh-keygen     //生成密钥

cat /root/.ssh/id_rsa.pub    //复制的到的字符串
```

##### 3. 编辑认证文件  （在虚拟机二上操作）

```
vi /root/.ssh/authorized_keys   点击I选择编辑，加上刚才复制的代码_
```

##### 4. 远程登陆 （在虚拟机一上操作）

```
ssh root@172.16.155.132    
```

![](https://farm2.staticflickr.com/1765/42227884834_9a59ecb821_o.png)


登陆之后，跟在mac上用iterm2登陆虚拟机是一样样的。

至此，两台虚拟机已经准备好，下面开始将local-linux01作为服务器；local-linux02作为客服端，安装nfs共享服务。

### 服务器端配置

- 关闭防火墙

```
systemctl stop firewalld

systemctl disable firewalld
```

- 关闭selinux

```
vi /etc/selinux/config

//找到SELINUX=enforcing这一行，将enforcing 改为 disabled，保存退出并重启虚拟机
```

- 安装`nfs-utils`、 `rpcbind`

```
yum install -y nfs-utils rpcbind

```

- nfs配置

```
vi /etc/exports

//写如下面的内容
/home/sharedir/ 172.16.155.128(rw,sync,all_squash,anonuid=1000,anongid=1000)

```

- 创建共享文件夹

```
//创建共享文件夹
mkdir -p  /home/sharedir

//给文件夹授权
chmod 766 /home/sharedir
```

- 设置开机启动

```
//开启NFS服务
systemctl start nfs

//把NFS设为开机启动
systemctl enable nfs

//开启RPC服务
systemctl start rpcbind

//将RPC设为开机自启
systemctl enable rpcbind

//校验是否启动
systemctl status nfs
```

如下所示，说明成功

![](https://farm2.staticflickr.com/1780/42898650762_ef95e08552_o.png)

#### nfs配置解读

如前所示，nfs的配置文件为/etc/exports，他的配置文件只有一行，如下所示：

![](https://farm2.staticflickr.com/1819/42948210481_c2ec853380_o.png)

共分为三个部分：

**第一部分：**/home/nfstestdir/ 表示共享目录

**第二部分：**192.168.188.0/24  允许哪些主机访问，这里既可以是ip，也可以是ip段；

**第三部分：**相关权限设置

* `rw` 读写权限；

* `ro` 只读权限；

* `sync` 同步模式，内存数据实时写入磁盘；

* `async` 异步模式，内存数据定期写入磁盘；

* `no_root_squash` 表示root用户对共享目录具有最高权限，安全性较差；

* `root_squash` root用户权限受限，只有普通用户权限；

* `all_squash` 所有用户均为限定为普通用户身份；

* `anonuid/anongid` 与`root_squash`或者`all_squash`连用，指定用户被限定权限后的uid/gid。

### 客户端配置

#### 第一步：安装`nfs-utils`

```
[root@local-linux01 ~]# yum install -y nfs-utils
```

#### 第二步：关闭防火墙firewalld，禁止selinx

```
//关闭firewalld
[root@local-linux01 ~]# systemctl stop firewalld
[root@local-linux01 ~]# systemctl disable firewalld

//编辑 /etc/selinux/config
selinux 设置为disabled
```

#### 第三步：挂在共享文件夹

- 检测服务端开放情况

```
[root@local-linux01 ~]# showmount -e 172.16.155.132
clnt_create: RPC: Port mapper failure - Unable to receive: errno 113 (No route to host)
```

##### `TroubleShooting`：无法建立联系

之前已经关闭了firewalld 和 selinx，但是iptables还在启用，网上查了一下，果然还是端口配置的问题。

原来RPC在NFS服务启动的时候，会随机给nfs服务分配端口，但这些端口在iptables当中并没有放行，所以就无法建立联系。

**解决思路**无非两条：

1. 关闭iptables；

2. 配置iptables，开放端口；

想想看，如果在现实当中，为了运行一个共享服务，把防火墙统统干掉，还是很不安全的。所以，思路一被否决。于是，只能想办法配置iptables。

**解决方法：**

- 查看端口信息

这里使用`rpcinfo`命令， `rpcinfo -p [host]`列出所有在host用portmap注册的RPC程序，如果没有指定host，就查找本机上的RPC程序。

```
[root@local-linux02 ~]# rpcinfo -p localhost
```

![](https://farm2.staticflickr.com/1831/29075690808_ee53bd07ce_o.png)

- 编辑`/etc/sysconfig/nfs`，去掉注释：

![](https://farm2.staticflickr.com/1827/41138526230_c0ab43653f_o.png)

- 编辑/etc/modprobe.d/lockd.conf，进行如下设置

![](https://farm2.staticflickr.com/1839/42899719842_a7a171fe66_o.png)

- 重启 rpcbind和nfs

```
//重启rpcbind
[root@local-linux02 ~]# service rpcbind restart
Redirecting to /bin/systemctl restart rpcbind.service

//重启nfs
[root@local-linux02 ~]# service nfs restart
Redirecting to /bin/systemctl restart nfs.service

//还需要重启机器，改动才能生效
[root@local-linux02 ~]# init 6

```

如图所示：

![](https://farm2.staticflickr.com/1830/42047265845_0a2577968f_o.png)

- 设置iptables，加入以下规则

```
vi /etc/sysconfig/iptables

-A INPUT -p tcp -m tcp --dport 111 -j ACCEPT
-A INPUT -p udp -m udp --dport 111 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 2049 -j ACCEPT
-A INPUT -p udp -m udp --dport 2049 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 892 -j ACCEPT
-A INPUT -p udp -m udp --dport 892 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 10006 -j ACCEPT
-A INPUT -p udp -m udp --dport 10007 -j ACCEPT
   
//保存退出
```

- 校验结果

```
[root@local-linux01 ~]# showmount -e 172.16.155.132
Export list for 172.16.155.132:
/home/sharedir 172.16.155.128
```

**重要补充：** 既然iptables不必关闭，其实centos7默认开启了firewalld，也不用关闭，同样按照这个方法进行配置就可以。只是配置端口这个环节略有不同。

```
//编辑firewalld中nfs服务的配置文件
cp /usr/lib/firewalld/services/nfs.xml /etc/firewalld/services/

//设置开放端口
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>NFS4</short>
  <description>The NFS4 protocol is used to share files via TCP networking. You will need to have the NFS tools installed and properly configure your NFS server for this option to be useful.</description>
  <port protocol="tcp" port="111"/>
  <port protocol="tcp" port="662"/>
  <port protocol="tcp" port="892"/>
  <port protocol="tcp" port="2049"/>
  <port protocol="tcp" port="32803"/>
  <port protocol="udp" port="111"/>
  <port protocol="udp" port="662"/>
  <port protocol="udp" port="892"/>
  <port protocol="udp" port="2049"/>
  <port protocol="udp" port="32769"/>
</service>

//开启nfs
firewall-cmd --permanent --zone=public --add-service=nfs

firewall-cmd --reload
```

#### 第四步：挂载共享文件夹

```
//新建一个空文件夹作为挂载点
[root@local-linux01 mnt]# mkdir -p /mnt/share

//使用mount命令挂载共享文件夹
[root@local-linux01 mnt]# mount -t nfs 172.16.155.132:/home/sharedir /mnt/share

说明：

`-t vfstype` 指定文件系统的类型，常用类型有： 

* iso9660： 光盘或光盘镜像

* msdosDOS： fat16文件系统 

* vfat： Windows 9x fat32文件系统

* ntfs： Windows NT ntfs文件系统

* smbfs：Mount Windows文件网络共享

* nfs：UNIX(LINUX) 文件网络共享 

`-o options` 主要用来描述设备或档案的挂接方式。常用的参数有： 

* loop：用来把一个文件当成硬盘分区挂接上系统；

* ro：采用只读方式挂接设备；

* rw：采用读写方式挂接设备；

* iocharset：指定访问文件系统所用字符集； 

```

- 校验结果

```
[root@local-linux01 mnt]# mount |grep share
172.16.155.132:/home/sharedir on /mnt/share type nfs4 (rw,relatime,vers=4.1,rsize=131072,wsize=131072,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=172.16.155.128,local_lock=none,addr=172.16.155.132)
```

TroubleShooting: 无法读写共享文件夹

![](https://farm2.staticflickr.com/1777/42949741291_f44dc380cf_o.png)

查看一下服务器端权限设置：

![](https://farm2.staticflickr.com/1789/42900094212_9d2e56ed5f_o.png)

估计还是文件夹权限设置的问题，突然想起来，在配置权限时设置了`all_squash`这一项，不论登入NFS 的使用者身份为何， 他的身份都会被降权成为普通用户，**后面跟的`anonuid anongid`，必须存在于你的/etc/passwd 当中对这个普通用户进行指定，换句话说，我们规定了来自客户端的访问等同于本地指定用户。**

在我们的实验当中，指定的uid1000的用户，对应的用户名是user1，而这个文件是由本地root用户创建的，所属用户是root，难怪没有权限进行访问。如下所示：

```
//查看uid为1000的用户
[root@local-linux02 ~]# cat /etc/passwd | grep 1000
user1:x:1000:1000::/home/user1:/bin/bash

//当前共享文件夹的所属用户和属组是root
[root@local-linux02 ~]# ls -l /home
total 0
drwx------. 2 php-fpm php-fpm 62 Jun  8 15:24 php-fpm
drwxrwxrwx  2 root    root     6 Jun 22 14:07 sharedir
drwx------. 2 user1   user1   62 Apr  3 22:58 use1
drwx------. 2 user1   user1   96 Apr  4 03:32 user1
drwx------. 2 user2   user2   62 Apr  3 23:13 user2
```

于是，尝试更改文件夹所属用户

```
[root@local-linux02 ~]# chown -R user1:user1 /home/sharedir
[root@local-linux02 ~]# ls -l /home
total 0
drwx------. 2 php-fpm php-fpm 62 Jun  8 15:24 php-fpm
drwxrw-rw-  2 user1   user1    6 Jun 22 14:07 sharedir
drwx------. 2 user1   user1   62 Apr  3 22:58 use1
drwx------. 2 user1   user1   96 Apr  4 03:32 user1
drwx------. 2 user2   user2   62 Apr  3 23:13 user2
```

再次尝试,果然成功

```
// 客户端操作
[root@local-linux01 mnt]# cd share
[root@local-linux01 share]# touch 1.txt

//服务器端验证
[root@local-linux02 ~]# ls -l /home/sharedir
total 0
-rw-r--r-- 1 user1 user1 0 Jun 22 17:38 1.txt
```

另外，还可以通过设置777权限来解决问题。不过，为什么是777，而不是766，目前还是不清楚。

```
chmod 777 /home/sharedir
```

### 补充知识点：自动挂载

AutoFs服务程序与Mount命令不同之处在于它是一种守护进程，只有检测到用户试图访问一个尚未挂载的文件系统时才自动的检测并挂载该文件系统。

#### 安装autofs

[root@local-linux01 ~]# yum -y install autofs

#### 修改配置文件（主配置+子配置）

sutofs 的配置文件是按照主配置+子配置的方式来部署的。先看主配置文件部署.

##### 主配置文件部署

autofs服务程序的主配置文件中需要按照“挂载目录 子配置文件”的格式写入参数。挂载目录是设备要挂载位置的上一级目录，例如之前客户端local-linux01的挂载点是/mnt/share/,此处写/mnt/，对应的子配置文件是/etc/auto.nfs ,意思是系统访问dir1下面的文件的时候，去/etc/auto.nfs 去查找nfs的配置。

```
vim /etc/auto.master
  #
  # Sample auto.master file
  # This is an automounter map and it has the following format
  # key [ -mount-options-separated-by-comma ] location
  # For details of the format look at autofs(5).
  #
  /mnt /etc/auto.nfs
```

##### 子配置文件

子配置文件中应按照“挂载目录 挂载文件类型及权限:设备名称”的格式写入参数

```
///etc/auto.nfs 该配置文件需新建
vim /etc/auto.nfs

dir2 -fstype=nfs 172.16.155.132:/home/sharedir
```

#### 设置开机启动

```
systemctl restart autofs

systemctl enable autofs
```


## 参考资料


* [Linux centos 7 安装NFS服务](https://www.e-learn.cn/content/linux/289912)

* [NFS服务介绍](https://www.jianshu.com/p/3cc66a8e355a)

* [nfs服务的使用指南](http://blog.51cto.com/oldma/2052230)

* [第十三章、文件服务器之一：NFS 服务器](http://cn.linux.vbird.org/linux_server/0330nfs.php)

* [Linux NFS服务器的安装与配置](https://www.cnblogs.com/mchina/archive/2013/01/03/2840040.html)

* [nfs设置固定端口并添加防火墙规则](https://www.centos.bz/2017/12/nfs%E8%AE%BE%E7%BD%AE%E5%9B%BA%E5%AE%9A%E7%AB%AF%E5%8F%A3%E5%B9%B6%E6%B7%BB%E5%8A%A0%E9%98%B2%E7%81%AB%E5%A2%99%E8%A7%84%E5%88%99/)

* [CentOS7安装配置 NFS](http://wangshengzhuang.com/2017/06/07/Linux/Linux%E5%9F%BA%E7%A1%80/CentOS%207%20%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE%20NFS/)


