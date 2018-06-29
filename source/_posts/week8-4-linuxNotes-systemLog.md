---
title: 'Linux运维学习笔记8-4：系统日志管理'
date: 2018-05-14 10:39:31
categories: linux
tags: [linux, rsync, notes] 
---

### 写在前面

这则笔记然后重点介绍系统日志管理方面知识。


### linux系统日志

系统日志是了解linux系统运行状况的重要途径，centos系统日志保存在`/var/log/`目录下：

![](https://farm1.staticflickr.com/982/42053118432_52f2b62a21_o.png)

上图中，日志文件主要包括以下几种类型：

<!--more-->

文件名|内容说明
---|---
message |系统启动后的信息和错误日志，是Red Hat Linux中最常用的日志之一 
secure |与安全相关的日志信息 
maillog |与邮件相关的日志信息 
cron |与定时任务相关的日志信息 
spooler |与UUCP和news设备相关的日志信息
boot.log |守护进程启动和停止相关的日志消息
access-log　　　|纪录HTTP/web的传输 
btmp　　　　　　|纪录失败的纪录 
lastlog　　　　 |纪录最近几次成功登录的事件和最后一次不成功的登录 
sudolog　　　　 |纪录使用sudo发出的命令 
sulog　　　　　 |纪录使用su命令的使用 
syslog　　　　　|从syslog中记录信息（通常链接到messages文件） 
utmp　　　　　　|纪录当前登录的每个用户 
wtmp　　　　　　|一个用户每次登录进入和退出时间的永久纪录 
xferlog　　　　 |纪录FTP会话 

/var/log/messages 是由rsysload 守护进程产生的，配置文件为/etc/rsyslog.conf。注：如无特殊需求，不要修改`/etc/rsyslog.conf`

#### `dmesg` 命令

`dmesg`命令显示linux内核的环形缓冲区信息，我们可以从中获得诸如系统架构、cpu、挂载的硬件，RAM等多个运行级别的大量的系统信息。

命令行格式：

```
dmesg [options]
```

参数|说明
---|---
-c 　|显示信息后，清除ring buffer中的内容
-s<缓冲区大小> 　|预设置为8196，刚好等于ring buffer的大小
-n 　|设置记录信息的层级

示例：

- 列出加载到内核中的所有驱动

```
dmesg | less
```

- 搜索包含特定字符串的被检测到的硬件，-i 表示忽略大小写

```
dmesg | grep -i usb
dmesg | grep -i tty
```

- 实时监控dmesg日志输出


```

watch 'dmesg | tail -f' 

或者

tail -f /var/log/dmesg
```

- 只输出dmesg命令开头/结尾20行日志

```
dmesg | head -n 20   //开头20行

dmesg | tail -n 20   //结尾20行
```

#### `last`命令

`last`命令，用来显示`/var/log/wtmp`文件，这是一个二进位制文件，内容是所有登录、登出的用户。

命令行格式：

```
last [options]
```

选项|说明
---|---
-a 　|把从何处登入系统的主机名称或IP地址，显示在最后一行；
-d |将IP地址转换成主机名称。
-f <记录文件> 　|指定记录文件，默认是显示/var/log目录下的wtmp文件的记录，但/var/log目录下得btmp能显示的内容更丰富，可以显示远程登录，例如ssh登录 ，包括失败的登录请求。
-n <显示列数>或-<显示列数> 　|设置列出名单的显示列数。
-R 　|不显示登入系统的主机名称或IP地址。
-x 　|显示系统关机，重新开机，以及执行等级的改变等信息。
-i |显示特定ip登录的情况
-t  |显示特定时刻之前的信息
-num |展示前 num 个
username |展示 username 的登入讯息
tty |限制登入讯息包含终端机代号

##### last 命令能查看哪些信息

![](https://farm1.staticflickr.com/946/28225927548_49efe7fb88_o.png)

字段|说明
---|---
第一列|用户名
第二列|终端位置
第三列|登录ip或者内核
第四列|开始时间
第五列|结束时间（still login in 还未退出  down 直到正常关机 crash 直到强制关机）
第六列|持续时间

**示例：**

- 显示特定的用户名

```
last root     //root可替换为需要查询的username
```

- 最后一列显示主机IP地址

```
last -n 5 -a -i
```

##### 拓展：lastb 查看失败登录
`lastb `命令记录失败的登录尝试。你必须拥有root权限才能运行lastb命令。

**示例：**

```
[root@stevey log]# lastb

btmp begins Mon May 14 09:58:37 2018
```

#### logrotate 命令

`logrotate`程序是一个日志文件管理工具。用于分割日志文件，删除旧的日志文件，并创建新的日志文件，起到“转储”作用，从而节省磁盘空间。

**日志轮转是系统自动完成的** 为了方便管理将每个服务类型的日志轮替配置信息形成独立文件存储在`/etc/logrotate.d/*`。

##### logrotate 配置文件

Logrotate的配置信息保存在`/etc/logrotate.conf`文件中。

```
[root@localhost ~]# cat /etc/logrotate.conf

=>下面为默认值。如不单独配置将才有下面的默认值
weekly    <==默认每周进行一次 rotate 的工作
rotate 4  <==默认保留4个登录文件 
create    <==以新创建文件继续存储日志文件
#compress <==被更动的登录文件是否需要压缩？如果登录文件太大则可考虑此参数启动

include /etc/logrotate.d

/var/log/wtmp {       <==仅针对 /var/log/wtmp 所配置的参数
    monthly           <==每个月一次，取代每周！
    minsize 1M        <==文件容量一定要超过 1M 后才进行
    create 0664 root utmp <==指定新建文件的权限与所属帐号/群组
    rotate 1          <==仅保留一个 
}

```

##### logrotate 语法

命令格式：

```
logrotate [OPTION] [configfile]
```

常用参数|说明
---|---
-d, --debug |debug模式，测试配置文件是否有错误；
-f, --force |强制转储文件；
-m, --mail=command |压缩日志后，发送日志到指定邮箱；
-s, --state=statefile |使用指定的状态文件；
-v, --verbose |显示转储过程；

常见用法：

```
//强制进行日志文件都进行轮替操作

logrotate -vf /etc/logrotate.conf

```

#### `xargs `命令

给其他命令传递参数的一个过滤器，也是组合多个命令的一个工具，擅长将标准输入数据转换成命令行参数，

- xargs的默认命令是echo，空格是默认定界符。这意味着通过xargs的处理，换行和空白将被空格取代。

```
[root@stevey ~]# cat a.txt    示例文件
 b c d e f g
 h i j k l m n
 o p q
 r s t
 u v w x y z


[root@stevey ~]# cat a.txt | xargs   //通过xargs的处理，换行和空白将被空格取代
b c d e f g h i j k l m n o p q r s t u v w x y z
```

- `-n`选项多行输出

```
[root@stevey ~]# cat a.txt | xargs -n 3
b c d
e f g
h i j
k l m
n o p
q r s
t u v
w x y
z

```

- `-d`选项:自定义一个定界符：

```
echo "nameXnameXnameXname" | xargs -dX

name name name name
```

- `-i`选项：将xargs的每项名称，一般是一行一行赋值给{}，可以用{}代替。 **注：**`-t `表示先打印命令，然后再执行。

```
[root@stevey test]# ls *.txt| xargs -t -i mv {} {}.bak
mv 1.txt 1.txt.bak
mv 3.txt 3.txt.bak
mv a.txt a.txt.bak
mv b.txt b.txt.bak
mv c.txt c.txt.bak
```

##### xargs结合find使用

- 查找所有的jpg 文件，并且压缩它们：

```
find . -type f -name "*.jpg" -print | xargs tar -czvf images.tar.gz

```

- 查询txt文件中所有的文件链接并下载

```
cat url-list.txt | xargs wget -c

```
#### `exec` 应用

`-exec`选项 将查到的文件执行command操作，`-exec   command   {} \;`  **注意：**{} 和 \;之间有空格


- 创建大于10天的文件删除

```
find . -mtime +10 -exec rm -rf {} \;
```

- 批量更名，与`xargs -i mv {} {}.bak` 作用类似

```
[root@stevey test]# ls
1.txt.bak  3.txt.bak  a.txt.bak  b.txt.bak  c.txt.bak
[root@stevey test]# find . -name *.bak -exec mv {} {}.txt \;
[root@stevey test]# ls
1.txt.bak.txt  a.txt.bak.txt  c.txt.bak.txt  test2.txt
3.txt.bak.txt  b.txt.bak.txt
```


### `screen`工具

screen是个用于命令行终端切换的窗口管理器,用户可以通过该软件同时连接多个本地或远程的命令行会话，并在其间自由切换。

安装：

```
yum install -y screen
```

语法：

```
screen [参数]
```

常用参数|说明
---|---
-A 　|将所有的视窗都调整为目前终端机的大小；
-d <作业名称> 　|将指定的screen作业离线；
-h <行数> 　|指定视窗的缓冲区行数；
-m 　|即使目前已在作业中的screen作业，仍强制建立新的screen作业；
-r <作业名称> 　|恢复离线的screen作业；
-R 　|先试图恢复离线的作业。若找不到离线的作业，即建立新的screen作业；
-s 　|指定建立新视窗时，所要执行的shell；
-S <作业名称> 　|指定screen作业的名称；
-v 　|显示版本信息；
-x 　|恢复之前离线的screen作业；
-ls或--list 　|显示目前所有的screen作业；
-wipe 　|检查目前所有的screen作业，并删除已经无法使用的screen作业；

#### Screen快捷键

在每个screen session 下，所有命令都以 ctrl+a(C-a) 开始，常用快捷键如下：

快捷键| 说明
---|---
C-a ? | 显示所有键绑定信息
C-a c | 创建一个新的运行shell的窗口并切换到该窗口
C-a n | Next，切换到下一个 window 
C-a p | Previous，切换到前一个 window 
C-a 0..9 | 切换到第 0..9 个 window
Ctrl+a [Space] | 由视窗0循序切换到视窗9
C-a C-a | 在两个最近使用的 window 间切换 
C-a x | 锁住当前的 window，需用用户密码解锁
C-a d | detach，暂时离开当前session，将目前的 screen session (可能含有多个 windows) 丢到后台执行，并会回到还没进 screen 时的状态，此时在 screen session 里，每个 window 内运行的 process (无论是前台/后台)都在继续执行，即使 logout 也不影响。 
C-a z | 把当前session放到后台执行，用 shell 的 fg 命令则可回去。
C-a w | 显示所有窗口列表
C-a t | Time，显示当前时间，和系统的 load 
C-a k | kill window，强行关闭当前的 window

`C-a [` 进入 copy mode，在 copy mode 下可以回滚、搜索、复制就像用使用 vi 一样

* C-b: Backward 
* C-f :Forward
* H(大写): High，将光标移至左上角 
* L :Low，将光标移至左下角 
* 0 :移到行首 
* $ :行末 
* w :forward one word，以字为单位往前移 
* b :backward one word，以字为单位往后移 
* Space :第一次按为标记区起点，第二次按为终点 
* Esc :结束 copy mode 

`C-a ]`  Paste，把刚刚在 copy mode 选定的内容贴上

示例：

```
screen vim a.txt     //创建一个窗口，运行vim a.txt

输入 `<ctrl + a> d` 暂时中断session

screen -ls    //显示目前所有的screen作业

screen -r 2066    //恢复session 2066

`<ctrl + a> ` ，然后输入 `:quit`   //关闭或杀死窗口

```

### 参考资料：

[Mac OS X:升级rsync到最新版本](https://blog.csdn.net/cneducation/article/details/4353523)

[macOS: rsync版本3.1.2安装使用以及其他备份工具](https://blog.csdn.net/cneducation/article/details/76168184)

[Linux系统中‘dmesg’命令处理故障和收集系统信息的7种用法](https://linux.cn/article-3587-1.html)

[运维中的日志切割操作梳理（Logrotate/python/shell脚本实现）](https://www.cnblogs.com/kevingrace/p/6307298.html)

[Linux内置的审计跟踪工具 - last命令](https://linux.cn/thread-12343-1-1.html)

[Linux系统日志及日志分析](http://c.biancheng.net/cpp/html/2783.html)

[xargs命令用法](http://man.linuxde.net/xargs)

[Xargs用法详解](https://blog.csdn.net/zhangfn2011/article/details/6776925)

[xargs用法讲解](https://www.cnblogs.com/chengyi818/p/4745382.html)



