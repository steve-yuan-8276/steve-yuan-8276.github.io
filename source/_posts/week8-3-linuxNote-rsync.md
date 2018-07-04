---
title: 'Linux运维学习笔记8-3：数据备份工具rsync'
date: 2018-05-11 14:46:22
categories: linux
tags: [linux, rsync, notes] 
---

### 写在前面

这则笔记重点整理rsync 数据同步备份工具的相关知识，这也是linux学习的重点。

### `rsync` 工具

rsync 命令是一个远程数据同步工具，可以在本地和远程之间通过shell或rsync服务互相拷贝文件。

它的优点在于：

* 可以镜像保存整个目录树和文件系统；

* 可以很容易做到保持原来文件的权限、时间、软硬链接等；

* 无须特殊权限即可安装；

* 快速：第一次同步时 rsync 会复制全部内容，但在下一次只传输修改过的文件。rsync 在传输数据的过程中可以实行压缩及解压缩操作，因此可以使用更少的带宽；

* 安全：可以使用 scp、ssh 等方式来传输文件，当然也可以通过直接的 socket 连接；

* 支持匿名传输，以方便进行网站镜象；

####  软件安装：

```
yum install -y rsync

```

<!--more-->

#### rsync语法：

##### 命令行格式：

```
//本地文件备份 Local:   
    rsync [OPTION...] SRC... [DEST]

//通过ssh远程同步备份 Access via remote shell:
    //pull
    rsync [OPTION...] [USER@]HOST:SRC... [DEST]
    
    //Push:
    rsync [OPTION...] SRC... [USER@]HOST:DEST

//通过服务远程同步备份 Access via rsync daemon:
    //Pull: 
    rsync [OPTION...] [USER@]HOST::SRC... [DEST]
    rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
    
    //Push: 
    rsync [OPTION...] SRC... [USER@]HOST::DEST
    rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST

```

##### 常用参数

常用参数|说明
---|---
-v，--verbose|详细模式输出；
-a，--archive|归档模式，表示以递归的方式传输文件，并保持所有文件属性不变，相当于使用了组合参数-rlptgoD; 后面跟--no-OPTION，option可以是rlptgoD中的任意一个，表示将这个选项去除。
-r，--recursive|对子目录以递归模式处理,传输目录时必须要加；
-l, --links |小写l，copy软链接文件;
-L|大写L，使用该选项后，会将软链接指向的目标文件复制到DST；
-p, --perms |保持文件权限;
-t, --times |保持文件时间信息;
-g, --group |保持文件属组信息;
-o, --owner |保持文件属主信息;
-D, --devices |保持设备文件信息;
-H, --hard-links |保留硬链接;
-u|把目标文件夹DST 中比原文件SRC还新的文件排除掉，不会同步；
-P, -partial|显示同步过程，如速率等，比-v显示更详细;
-z, --compress |对备份的文件在传输时进行压缩处理；
--delete |删除那些DST中SRC没有的文件;
--exclude=PATTERN |排除满足特定条件的文件，这里的pattern也是具体的文件名，也可以是带有通配符，比如*.txt
--progress |显示同步的过程状态，包括同步文件数量、传输速度等；


#### 本地文件同步备份

- 文件准备

```
//文件准备

[root@stevey ~]# cd rsync
[root@stevey rsync]# pwd
/root/rsync
[root@stevey rsync]# mkdir test
[root@stevey rsync]# cd test
[root@stevey test]# touch {1,2,3}.txt /root/b.txt
[root@stevey test]# ln -s /root/b.txt ./a.txt
[root@stevey test]# ll
total 0
-rw-r--r--. 1 root root  0 May 22 22:03 1.txt
-rw-r--r--. 1 root root  0 May 22 22:03 2.txt
-rw-r--r--. 1 root root  0 May 22 22:03 3.txt
lrwxrwxrwx. 1 root root 11 May 22 22:10 a.txt -> /root/b.txt
```

##### `-a` 选项：表示以递归的方式传输文件，并保持所有文件属性不变，相当于使用了组合参数-rlptgoD

**如果目录`test` 不加斜杠，会新建一个`test2` 的目录，将`test1`放到新建目录下；加一个斜杠，才能得到我们想要的结果**

- 示例1: 目录test不加斜杠

```
[root@stevey test2]# rsync -a test  test2
[root@stevey test2]# tree -L 2 test2
test2
└── test
    ├── 1.txt
    ├── 2.txt
    ├── 3.txt
    └── a.txt -> /root/b.txt

1 directory, 4 files
```

- 示例2: 目录test加斜杠

```
[root@stevey rsync]# rsync -a test/ test2
[root@stevey rsync]# ls
test  test2
[root@stevey rsync]# tree -L 2 test
test
├── 1.txt
├── 2.txt
├── 3.txt
└── a.txt -> /root/b.txt

0 directories, 4 files
[root@stevey rsync]# tree -L 2 test2
test2
├── 1.txt
├── 2.txt
├── 3.txt
└── a.txt -> /root/b.txt
```

`-a`选项相当于使用了组合参数`-rlptgoD`；后面可以跟`--no-OPTION`，排除某个选项；

```
[root@stevey test]# ll
total 0
-rw-r--r--. 1 root root  0 May 22 22:03 1.txt
-rw-r--r--. 1 root root  0 May 22 22:03 2.txt
-rw-r--r--. 1 root root  0 May 22 22:03 3.txt
lrwxrwxrwx. 1 root root 11 May 22 22:10 a.txt -> /root/b.txt
[root@stevey test]# cd ..
[root@stevey test2]# rsync -a --no-l test/ test2/   //--no-l 表示排除软链接文件
skipping non-regular file "a.txt"
[root@stevey test2]# tree -L 2 test2/
test2/
├── 1.txt
├── 2.txt
└── 3.txt

0 directories, 3 files
```

##### `-L` 选项：使用该选项后，会将软链接指向的目标文件复制到DST

```
//文件准备
[root@stevey rsync]# echo 'this is s test' > /root/b.txt
[root@stevey rsync]# cat !$
cat /root/b.txt
this is s test

//-v 选项表示详细模式输出；-L则会把软链接的目标文件一起同步到DST
[root@stevey rsync]# rsync -avL test/ test2/
sending incremental file list
created directory test2
./
1.txt
2.txt
3.txt
a.txt

sent 298 bytes  received 123 bytes  842.00 bytes/sec
total size is 15  speedup is 0.04
[root@stevey rsync]# cd test2
[root@stevey test2]# ll
total 4
-rw-r--r--. 1 root root  0 May 22 22:03 1.txt
-rw-r--r--. 1 root root  0 May 22 22:03 2.txt
-rw-r--r--. 1 root root  0 May 22 22:03 3.txt
-rw-r--r--. 1 root root 15 May 22 22:42 a.txt
[root@stevey test2]# cat a.txt
this is s test
```

##### `-u` 选项：	仅仅进行更新，跳过所有已经存在于 DST，如果文件时间晚于要备份的文件，不覆盖更新的文件

```
//文件准备

//test/1.txt与test2/1.txt的创建时间是一样的
[root@stevey rsync]# ll test/1.txt test2/1.txt
-rw-r--r--. 1 root root 0 May 22 22:03 test/1.txt
-rw-r--r--. 1 root root 0 May 22 22:03 test2/1.txt

//touch命令修改test2/1.txt的创建时间
[root@stevey rsync]# touch test2/1.txt
[root@stevey rsync]# ll test/1.txt test2/1.txt
-rw-r--r--. 1 root root 0 May 22 22:03 test/1.txt
-rw-r--r--. 1 root root 0 May 22 22:50 test2/1.txt

//不使用-u选项，源目录文件会覆盖目标文件
[root@stevey rsync]# rsync -a test/ test2/
[root@stevey rsync]# ll test/1.txt test2/1.txt
-rw-r--r--. 1 root root 0 May 22 22:03 test/1.txt
-rw-r--r--. 1 root root 0 May 22 22:03 test2/1.txt

//使用-u选项，源目录不会覆盖目标文件，因为目标文件更新
[root@stevey rsync]# rsync -au test/ test2/
[root@stevey rsync]# ll test/1.txt test2/1.txt
-rw-r--r--. 1 root root 0 May 22 22:03 test/1.txt
-rw-r--r--. 1 root root 0 May 22 22:57 test2/1.txt
```

##### `--delete` 选项：假设DST中有一个文件`c.txt`，而SRC中没有；执行同参数同步后，DST文件夹中的`c.txt`也会被删除

```
//准备文件
[root@stevey rsync]# cd test2
[root@stevey test2]# touch c.txt
[root@stevey test2]# cd ..
[root@stevey rsync]# tree -L 2 test2/
test2/
├── 1.txt
├── 2.txt
├── 3.txt
├── a.txt -> /root/b.txt
└── c.txt

0 directories, 5 files

//--delete 选项，同步备份时会删掉源目录没有的文档
[root@stevey rsync]# rsync -av --delete test/ test2/
sending incremental file list
deleting c.txt

sent 132 bytes  received 21 bytes  306.00 bytes/sec
total size is 11  speedup is 0.07
[root@stevey rsync]# tree -L 2 test2/
test2/
├── 1.txt
├── 2.txt
├── 3.txt
└── a.txt -> /root/b.txt

0 directories, 4 files
```

##### `--exclude` 选项：排除满足特定条件的文件，可以是具体的文件名，也可以包含通配符，如`*.txt`

- 排除具体文件 `c.txt`
 
```
[root@stevey rsync]# rsync -av --exclude='1.txt' test/ test2/
sending incremental file list
created directory test2
./
2.txt
3.txt
a.txt -> /root/b.txt

sent 204 bytes  received 88 bytes  584.00 bytes/sec
total size is 11  speedup is 0.04
[root@stevey rsync]# tree -L 2 test2
test2
├── 2.txt
├── 3.txt
└── a.txt -> /root/b.txt

0 directories, 3 files

```

- 排除选项与通配符连用 *.txt

```
[root@stevey rsync]# rsync -av --exclude="*.txt" test/ test2/
sending incremental file list
created directory test2
./

sent 43 bytes  received 47 bytes  180.00 bytes/sec
total size is 0  speedup is 0.00
[root@stevey rsync]# tree -L 2 test2
test2

0 directories, 0 files
```

##### `--progress`,显示同步的过程状态，包括同步文件数量、传输速度等

```
[root@stevey rsync]# rsync -a --exclude="1.txt" --progress test/ test2/
sending incremental file list
2.txt
              0 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=2/4)
3.txt
              0 100%    0.00kB/s    0:00:00 (xfr#2, to-chk=1/4)
a.txt -> /root/b.txt
```

#### rsync通过ssh同步

**前提：**

准备两台机器，一台是我的mac，ip是172.16.155.1;另一台是虚拟机，ip是172.16.155.128;mac可以通过ssh访问虚拟机。

- push：把本地目录同步到远程主机：

```
//mac操作
steve:rsync steveyuan$ rsync -a /Users/steveyuan/documents/rsync/test1/ root@172.16.155.128:/root/rsync

//虚拟机验证效果
[root@stevey rsync]# ls
a.txt
[root@stevey rsync]# cat a.txt
rsync test
```

- pull：把远程主机目录同步到本地：

```

//准备文件
[root@stevey rsync]# touch b.txt
[root@stevey rsync]# echo 'hello,steve' > b.txt

//mac操作
steve:rsync steveyuan$ rsync -av root@172.16.155.128:/root/rsync/ /Users/steveyuan/documents/rsync/
receiving incremental file list
./
a.txt
b.txt
sent 65 bytes  received 209 bytes  182.67 bytes/sec
total size is 23  speedup is 0.08

//校验效果
steve:rsync steveyuan$ ls
a.txt	b.txt
steve:rsync steveyuan$ cat b.txt
hello,steve
```

#### `rsync`通过服务同步

这种方式相当于在远程主机上建一个rsync的服务器，在服务器上配置好之后，将本地主机作为rsync的一个客户端，在本地机器和服务器之间进行文件同步备份。

要实现这个目标，首先要在在服务器端进行配置：

##### 配置文件说明：

```
# cat /etc/rsyncd.conf  

port = 873   //端口号
uid = nobody  //指定当模块传输文件的守护进程UID
gid = nobody  //指定当模块传输文件的守护进程GID
use chroot = no  //安全选项，设定为true，会将同步目录限定在path指定的目录当中；如果有软链接文件到别的文件目录，就会报错
max connections = 10  //最大并发连接数
strict modes = yes  //指定是否检查口令文件的权限

pid file = /usr/local/rsyncd/rsyncd.pid  //指定PID文件
lock file = /usr/local/rsyncd/rsyncd.lock  //指定支持max connection的锁文件，默认为/var/run/rsyncd.lock
motd file = /usr/local/rsyncd/rsyncd.motd  //定义服务器信息的，自己写 rsyncd.motd 文件内容
log file = /usr/local/rsyncd/rsync.log  //rsync 服务器的日志

log format = %t %a %m %f %b
syslog facility = local3
timeout = 300

[conf]  //自定义模块名，这里写的名称在同步时需要用到
path = /usr/local/nginx/conf  //用来指定要备份的目录
comment = Nginx conf
ignore errors  //可以忽略一些IO错误
read only = no  //设置no，客户端可以上传文件，yes是只读
write only = no  //no为客户端可以下载，yes 不能下载
hosts allow = 192.168.2.0/24  //可以连接的IP段 
hosts deny = *  //禁止连接的IP
list = false  //客户请求时，使用模块列表
uid = root
gid = root
auth users = backup  //连接用户名，和linux系统用户名无关系
secrets file = /etc/rsyncd.pass  //验证密码文件
```

如上所示，服务器配置一般包括两个部分：全局配置、局部配置，具体如下：


###### 全局配置：

* `port`：指定从rsyncd服务启动端口，默认从873端口启动；

* `log file`：指定日志文件；

* `pid file`：指定pid 文件，该文件设计服务的启动】停止等进程管理操作；

* `address`：指定启动rsync服务的ip；如果服务器有多个ip，就需要知道；如果不指定，默认从全部ip启动。

###### 局部配置：

* `[ftp]` 表示模块，模块名用 `[]` 括起来，这里的模块名在后面执行同步操作的时候需要用到。

* `path`：数据存放路径；

* `use chroot`：表示在传输文件前，先chroot到path指定的目录下，实现额外的安全保护。该服务默认开启状态，缺点是需要root权限，且不支持软链接同步；如果需要同步的数据中有软链接文件，建议设为false。

* `max connections`： 最大链接数，默认为0，表示没有限制。

* `read only`：该目录是否只读；

* `list`：表示当前用户查询服务器可用模块时，该模块是否列出；

* `uid/gid`：传输文件时，以那个用户/组的身份传输；

* `auth users`：指定传输时使用的用户名；

* `secret file`：指定密码文件，该文件权限要设为600；

* `host allow`：允许连接该模块的主机，可以是网段，也可以是具体的ip地址。


下面，来实际操作一下：

##### 第一步：服务器端配置：

服务器端端配置文件在 `/etc/rsyncd.conf`,首先修改配置文件：

```
vim /etc/rsyncd.conf    //编辑 rsync 的配置文件，加入以下内容

port=8730   //自定义端口号
log file=/var/log/rsync.log
pid file=/var/run/rsyncd.pid
address=172.16.155.128   //此处写的是服务器端ip

[test]
	path=/tmp/rsync
	use chroot=true  //安全选项，如果此项设置为true，同步目录会被限定在path目录，软链接到其他目录的文件不会被同步；如果需要同步软链接文件，此项建议设为false
	max connections=4   //最大链接数，同时允许几台机器远程备份同步
	read only=false     //如果设置为true，就不能执行远程同步操作
	list=true
	uid=root
	gid=root
	auth users=test    //指定用来同步的用户名，与本地主机名无关
	secrets file=/etc/rsyncd.passwd     //安全性选项，指定同步用户密码，此文件需要新建，内容格式为 name：passwd
	host allow=172.16.155.1    //指定允许那一个ip访问，也可以设置为ip段，如果是0.0.0.0/24就是全网段
```

保存退出后，执行以下命令开启 `rsync` 服务

```
//启动服务
rsync --daemon  --config=/etc/rsyncd.conf

//检查服务是否开启
[root@stevey rsync]# ps -aux | grep rsync
root      80564  0.0  0.0 114740   564 ?        Ss   05:34   0:00 rsync --daemon --config=/etc/rsyncd.conf
root      82057  0.0  0.0 112704   972 pts/2    R+   13:34   0:00 grep --color=auto rsync

//netstat命令检查端口状况；-l 列出listen端口状态，-n显示数字，-t显示tcp相关，-p显示建立链接的程序名
[root@stevey rsync]# netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      7057/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      971/master
tcp        0      0 172.16.155.128:873      0.0.0.0:*               LISTEN      80564/rsync
tcp6       0      0 :::22                   :::*                    LISTEN      7057/sshd
tcp6       0      0 ::1:25                  :::*                    LISTEN      971/master
```

此外，配置文件中还提到了几个文件，需要创建：

```
//创建同步文件夹路径并更改权限
[root@stevey rsync]# mkdir -p /tmp/rsync/
[root@stevey rsync]# chmod 777 !$
chmod 777 /tmp/rsync/

```

##### 第二步：同步操作

命令行格式：

```
rsync -avP [源文件夹] 远程主机ip::[模块名] [文件名称]
```

**注：**

* 模块名，就是在/etc/rsyncd.conf 中定义的模块名，文件灰白放在该模块指定的path路径下；

* `rsync -P` 选项，表示断点续传，链接服务器是很有用；

* `[文件名称]`指的是，可以将同步过来的文件重命名，当然此项也可以省略；

尝试同步后，出现错误提示:connection refused

```
/Users/steveyuan/documents/rsync
steve:rsync steveyuan$ rsync -avP /Users/steveyuan/documents/rsync 172.16.155.128::test
rsync: failed to connect to 172.16.155.128 (172.16.155.128): Connection refused (61)
rsync error: error in socket IO (code 10) at clientserver.c(127) [sender=3.1.3]

```

###### TroubleShooting:

- 先看看网络是否能ping通

```
//如下所示，网络没有问题
steve:rsync steveyuan$ ping 172.16.155.128
PING 172.16.155.128 (172.16.155.128): 56 data bytes
64 bytes from 172.16.155.128: icmp_seq=0 ttl=64 time=0.417 ms
64 bytes from 172.16.155.128: icmp_seq=1 ttl=64 time=0.424 ms
64 bytes from 172.16.155.128: icmp_seq=2 ttl=64 time=0.423 ms
64 bytes from 172.16.155.128: icmp_seq=3 ttl=64 time=0.566 ms

--- 172.16.155.128 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.417/0.458/0.566/0.063 ms
```

- 使用telnet检查端口情况

telnet 需单独安装

```
// centos7
yum install -y telnet

// mac
brew install telnet
```

命令行格式：

```
telnet [IP] [端口号]   //注意没有短横线
```

```
//测试端口情况
steve:rsync steveyuan$ telnet 172.16.155.128 873
Trying 172.16.155.128...
telnet: connect to address 172.16.155.128: Connection refused
telnet: Unable to connect to remote host   //说明端口没有开放
```

既然已经知道了原因是端口没有开放，那就增加一条iptables规则

```
[root@stevey rsync]# iptables -I INPUT -s 172.16.155.0/24 -p tcp --dport 873 -j ACCEPT
```

再检查端口情况：，ok了

```
steve:rsync steveyuan$ telnet 172.16.155.128 873
Trying 172.16.155.128...
Connected to bogon.
Escape character is '^]'.
@RSYNCD: 31.0
```

在执行同步操作，成功！

```
steve:rsync steveyuan$ rsync -avP /Users/steveyuan/documents/rsync 172.16.155.128::test
sending incremental file list
rsync/
rsync/a.txt
             11 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=1/3)
rsync/b.txt
             12 100%   11.72kB/s    0:00:00 (xfr#2, to-chk=0/3)

sent 232 bytes  received 58 bytes  580.00 bytes/sec
total size is 23  speedup is 0.08
```

##### 附加步骤：同步文件设置密码

仅仅是这样设置，意味着任何人只要知道主机ip、端口号和mudule名，都可以执行同步操作，这显然是不安全的。所以，还需要设置密码。

- 创建服务器密码文件

```
//创建密码文件
touch /etc/rsyncd.passwd

//mkpasswd命令生成一个密码
[root@stevey rsync]# mkpasswd -l 14
rexGlkmvh97xI#

//编辑密码文件
vim /etc/rsyncd.passwd
//密码格式为 name：passwd
test：rexGlkmvh97xI#

//修改文件权限
chmod 600 /etc/rsyncd.passwd
```

- 修改服务端rsync 配置文件

```
vim /etc/rsyncd.conf
```

![](https://farm1.staticflickr.com/885/41392949015_e5a9f223a0_o.png)

- 创建本地密码文件

```
steve:~ steveyuan$ touch desktop/rsync_passwd
steve:~ steveyuan$ vim !$
vim desktop/rsync_passwd    //注意这个文件内容只包含密码

steve:~ steveyuan$ chmod 600 !$    //修改文件权限
chmod 600 desktop/rsync_passwd
```

- 再次尝试执行同步，就需要手动输入密码文件

![](https://farm1.staticflickr.com/960/42247831092_922f257b94_o.png)

- `--password-file=[passwdfile]` 指定密码文件，就不用手动输入密码了。注：这里的文件要写绝对路径。

```
steve:desktop steveyuan$ rsync -avP /Users/steveyuan/documents/rsync test@172.16.155.128::test --password-file=/Users/steveyuan/desktop/rsync_passwd
sending incremental file list

sent 120 bytes  received 17 bytes  274.00 bytes/sec
total size is 23  speedup is 0.17

```

最后，这里的密码强度不够，同时保存路径也太随意了。在真正的生产环境中，不可以保存在桌面，太不安全了。

### 参考资料

* [使用rsync命令同步本地目录和远程主机目录](http://blog.topspeedsnail.com/archives/2668)

* [《rsync同步的艺术》–linux命令五分钟系列之四十二](http://roclinux.cn/?p=2643)

* [rsync 命令整理](https://www.jianshu.com/p/258ceb7b2223)

* [rsync命令](http://man.linuxde.net/rsync)

* [Rsync命令详解](https://blog.csdn.net/u010391029/article/details/51746641)


