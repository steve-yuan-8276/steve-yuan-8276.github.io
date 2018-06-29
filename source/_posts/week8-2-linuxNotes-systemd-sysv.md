---
title: 'linux运维学习笔记：系统进程管理'
date: 2018-05-24 17:34:48
categories: linux
tags: [linux, sysV, notes]
---

## 写在前面

第一阶段复习，发现有很多内容掌握的不扎实，这个sysV、Systemd尤其是如此，所以单独提出来再查找资料，再整理一下。

## 预备知识：先从linux的历史说起

### `init` 守护进程

`sysV`和`Systemd`是神马？这是一个很复杂的问题，需要Linux系统启动说起。

linux系统启动，主要可以分成以下几步：

* 读取 MBR 的信息,启动 Boot Manager（Windows 使用 NTLDR 作为 Boot Manager；Linux 通常使用GRUB 作为 Boot Manager）；

* 加载系统内核,启动 init 进程，这是Linux 的根进程,所有的系统进程都是它的子进程；

* init 进程读取 /etc/inittab 文件中的信息,并进入预设的运行级别,按顺序运行该运行级别对应文件夹下的脚本；

* 根据 /etc/rcS.d/ 文件夹中对应的脚本启动；

* 启动登录管理器,等待用户登录；
        

根据维基百科的解释，`init`是 Unix 和 类Unix 系统中用来产生其它所有进程的程序。它以守护进程的方式存在，其进程号pid为1。Linux系统在开机时加载Linux内核后，便由Linux内核加载init程序，由init程序完成余下的开机过程，比如加载运行级别，加载服务，引导Shell/图形化界面等等。

<!--more-->

#### SysV 

PID 1 是一个非常特殊的进程，任务是把整个操作系统带入可操作的状态。传统的 Unix System V 相兼容的，所以也叫 sysvinit。

在 sysvint中，有好几个允许级别，常见的级别，比如0代表关机，3代表多用户模式，6代表重启配置在 `/etc/inittab` 文件中。当然，这里还需要知道一个文件夹`/etc/init.d/`，存放各种进程的启停脚本。

#### UpStart

linux是从unix系统衍生而来的，早期多用在服务器上。但是随着时代的发展，linux逐渐进入了桌面系统时代，sysV这种启动方式的弊端也就暴露的出来。由于他没有自动检测机制，只能一次性启动所有服务，所以系统系统的速度就很慢。

为了解决这个痛点，2006年，一些大牛决定重新设计init系统，推出了upstart系统，upstart 基于事件驱动的机制，把之前的完全串行的同步启动服务的方式改成了由事件驱动的异步的方式。这样做的好处是，可以并行启动服务，他们之间的依赖关系，由相应的事件通知完成。听起来有点复杂，不过简单说，采用这套机制，系统启动速度更快了。

#### Systemd

时间前进到2010年，又有大牛提出来这个UpStart也是不完美的，systemd 设计的首要目标是要让用户快速的进入可以操作OS的环境，其设计理念就是两条：

* To start less. 按需启动，能不启动就不启动

* And to start more in parallel. 如果要启动，能并行启动就并行启动，即使程序之间有依赖，也并行启动；

目前，systemd 几乎成为了所有主流Linux 发行版的默认配置，包括：Arch Linux、CentOS、CoreOS、Debian、Fedora、Megeia、OpenSUSE、RHEL、SUSE企业版和 Ubuntu。

sysV和systemd的对应指令

任务	|指令	|新指令
---|---|---
使某服务自动启动	|chkconfig –level 3 sshd on|	systemctl enable sshd.service
使某服务不自动启动	|chkconfig –level 3 sshd off|	systemctl disable sshd.service
检查服务状态	|service sshd status|	systemctl status sshd.service （服务详细信息）systemctl is-active sshd.service （仅显示是否 Active)
显示所有已启动的服务	|chkconfig –list|	systemctl list-units –type=service
启动某服务	|service sshd start|systemctl start sshd.service
停止某服务	|service sshd stop|systemctl stop sshd.service
重启某服务	|service sshd restart|systemctl restart sshd.service

好了，下面切入正题。

## 重新认识sysytemd


`systemd`是一种新的linux系统服务管理器，用于取代SysV初始进程。它替换了init系统，能够管理系统的启动过程和一些系统服务，一旦启动起来，就将监管整个系统。

**Systemd 架构图**

![](https://farm1.staticflickr.com/948/42013365781_be673ef7c2_o.png)

这里有两个重要概念：**unit**和**target**，下面分别介绍：

### unit介绍

`Systemd` 可以管理所有系统资源，我们将不同的资源统称为 `unit`，一共分成12种：

unit | 说明
---|---
Service|系统服务
Target|多个 Unit 构成的一个组
Device|硬件设备
Mount|文件系统的挂载点
Automount|自动挂载点
Path|文件或路径
Scope|不是由 Systemd 启动的外部进程
Slice|进程组
Snapshot|Systemd 快照，可以切回某个快照
Socket|进程间通信的 socket
Swap|swap 文件
Timer|定时器

#### unit配置文件

每一个 Unit 都有一个配置文件，`Systemd` 默认目录在 `/etc/systemd/system/`；但其中有很多配置文件都是软链接，指向 `/usr/lib/systemd/system/`，也就是说，**真正的配置文件**保存在 `/usr/lib/systemd/system/`目录下。

**配置文件的格式**

```
[root@stevey system]# cat auditd.service
[Unit]
Description=Security Auditing Service
DefaultDependencies=no
## If auditd.conf has tcp_listen_port enabled, copy this file to /etc/systemd/system/auditd.service and add network-online.target to the next line so it waits for the network to start before launching.

[Service]
Type=forking
PIDFile=/var/run/auditd.pid
ExecStart=/sbin/auditd
## To not use augenrules, copy this file to /etc/systemd/system/auditd.service and comment/delete the next line and uncomment the auditctl line.

[Install]
WantedBy=multi-user.target
```

unit配置文件通常包括几个部分：

`[Unit]` ：区块通常是配置文件的第一个区块，用来定义 Unit 的元数据，以及配置与其他 Unit 的关系，主要字段如下：

名称|说明
---|---
Description|简短描述
Documentation|文档地址
Requires|当前 Unit 依赖的其他 Unit，如果它们没有运行，当前 Unit 会启动失败
Wants|与当前 Unit 配合的其他 Unit，如果它们没有运行，当前 Unit 不会启动失败
BindsTo|与Requires类似，它指定的 Unit 如果退出，会导致当前 Unit 停止运行
Before|如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之后启动
After|如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之前启动
Conflicts|这里指定的 Unit 不能与当前 Unit 同时运行
Condition...|当前 Unit 运行必须满足的条件，否则不会运行
Assert...|当前 Unit 运行必须满足的条件，否则会报启动失败


`[Install]`|通常是配置文件的最后一个区块，用来定义如何启动，以及是否开机启动，主要字段如下：

名称|说明
---|---
WantedBy|它的值是一个或多个 Target，当前 Unit 激活时（enable）符号链接会放入/etc/systemd/system目录下面以 Target 名 + .wants后缀构成的子目录中
RequiredBy|它的值是一个或多个 Target，当前 Unit 激活时，符号链接会放入/etc/systemd/system目录下面以 Target 名 + .required后缀构成的子目录中
Alias|当前 Unit 可用于启动的别名
Also|当前 Unit 激活（enable）时，会被同时激活的其他 Unit


`[Service]`： Service 的配置，只有 Service 类型的 Unit 才有这个区块。它的主要字段如下。

名称|说明
---|---
Type|定义启动时的进程行为。它有以下几种值。
Type=simple|默认值，执行ExecStart指定的命令，启动主进程
Type=forking|以 fork 方式从父进程创建子进程，创建后父进程会立即退出
Type=oneshot|一次性进程，Systemd 会等当前服务退出，再继续往下执行
Type=dbus|当前服务通过D-Bus启动
Type=notify|当前服务启动完毕，会通知Systemd，再继续往下执行
Type=idle|若有其他任务执行完毕，当前服务才会运行
ExecStart|启动当前服务的命令
ExecStartPre|启动当前服务之前执行的命令
ExecStartPost|启动当前服务之后执行的命令
ExecReload|重启当前服务时执行的命令
ExecStop|停止当前服务时执行的命令
ExecStopPost|停止当其服务之后执行的命令
RestartSec|自动重启当前服务间隔的秒数
Restart|定义何种情况 Systemd 会自动重启当前服务，可能的值包括always（总是重启）、on-success、on-failure、on-abnormal、on-abort、on-watchdog
TimeoutSec|定义 Systemd 停止当前服务之前等待的秒数
Environment|指定环境变量


配置文件一共有四种状态：

* `enabled`：已建立启动链接

* `disabled`：没建立启动链接

* `static`：该配置文件没有[Install]部分（无法执行），只能作为其他配置文件的依赖

* `masked`：该配置文件被禁止建立启动链接

#### `unit` 依赖关系

Unit 之间存在依赖关系，如果`unit A` 依赖 `unit B`，在启动`unit A`的时候，会同时启动`unit B`。

查看Unit 的依赖

```
$ systemctl list-dependencies nginx.service
```

如果依赖是 Target 类型，默认不会展开；如果需要查询，需要加上`--all`参数

```
$ systemctl list-dependencies --all nginx.service
```

### `target`介绍

启动级别是一个旧的概念。现在，systemd 引入了一个和启动级别`runlevel`功能相似又不同的概念——目标`target`。

`Target` 就是一个 `Unit`组，包含许多相关的 `Unit` ，类似于"状态点"，启动某个 `Target` 的时候，会让机器启动到到某种状态。

#### Target 与 传统 RunLevel 的对应关系

在`/usr/lib/systemd/system/`目录下，查找` ls -l runlevel*`，可以看到：

![](https://farm1.staticflickr.com/966/42312212441_aa09a27285_o.png)

SysV run level启动级别	|Systemd target 目标	|注释
---|---|---
0| `poweroff.target`|	关机
1|	`rescue.target`|	救援模式
2|`multi-user.target`	|多用户模式
3|`multi-user.target`|	多用户模式
4|`multi-user.target`|	多用户模式，在contos6之前，这是一个预留的模式
5|`graphical.target`|	多用户，图形界面。继承级别3的服务，并启动图形界面服务。
6| `reboot.target`|	重启

好了，基础知识到此为止，下面到应用部分：

## 服务管理相关命令

### `sysV`下如何管理服务

`chkconfig`命令主要用来更新（启动或停止）和查询系统服务的运行级信息，主要跟sysV配合起来作用。具体的要管理服务，大概的操作流程是这样的：

1. 编写服务脚本`vim [servicename]`，保存在`/etc/ini.d/`目录下； 注：该文件必须为755 权限 `chown 755 [shellfilename]`

2. 增加指定的系统服务 `chkconfig --add [servicename]`

3. 设定服务的默认启动等级 `chkconfig --level 35 mysqld on`

#### sysV相关命令


```
chkconfig --list                //列出所有的系统服务

chkconfig --list mysqld         //列出mysqld服务设置情况  

chkconfig --add httpd           //增加httpd服务 

chkconfig --del httpd           //删除httpd服务 
```

还有一种用法：

```
chkconfig [--level][系统服务][参数] 

```

`--level`，指定读系统服务要在哪一个执行等级中开启或关闭，centos6.x之前的版本中，共有以下7种级别。

级别代码|说明
---|---
0|表示关机 
1|单用户模式 
2|无网络连接的多用户命令行模式 
3|有网络连接的多用户命令行模式 
4|不可用 
5|带图形界面的多用户模式 
6|重新启动 

常见参数|说明
---|---
on|启动
off|关闭
reset|重置服务

**注：**`on`和`off`默认只对运行级3，4，5有效；`reset`对所有运行级有效。

示例：

```
chkconfig --level httpd 2345 on //设置httpd在运行级别为2、3、4、5的情况下都是on（开启）的状态 

chkconfig --level 35 mysqld on  //--level 35表示操作只在等级3和5执行，on表示启动，off表示关闭
 
chkconfig mysqld on             //设定mysqld在各等级为on，“各等级”包括2、3、4、5等级 

chkconfig [service] off     //关闭指定服务 

```

### Systemd 如何管理服务

#### `systemctl:` `Systemd` 的主命令，用于管理系统。


```
// 重启系统
systemctl reboot

// 关闭系统，切断电源
systemctl poweroff

// CPU停止工作
systemctl halt

// 暂停系统
systemctl suspend

// 让系统进入冬眠状态
systemctl hibernate

// 让系统进入交互式休眠状态
systemctl hybrid-sleep

// 启动进入救援状态（单用户状态）
systemctl rescue

```

##### 与Unit有关的命令


```
// 列出正在运行的 Unit
$ systemctl list-units

//列出列出所有配置文件
$ systemctl list-unit-files

// 列出所有Unit，包括没有找到配置文件的或者启动失败的
$ systemctl list-units --all

// 列出所有没有运行的 Unit
$ systemctl list-units --all --state=inactive

// 列出所有加载失败的 Unit
$ systemctl list-units --failed

// 列出所有正在运行的、类型为 service 的 Unit
systemctl list-units --type=service

//列出正在运行，状态为active的service
systemctl list-units --type=service

//查看某个服务是否为active
systemctl is-active crond.service

// 立即启动一个服务
systemctl start apache.service

// 立即停止一个服务
systemctl stop apache.service

// 重启一个服务
systemctl restart apache.service

// 杀死一个服务的所有子进程
systemctl kill apache.service

// 重新加载一个服务的配置文件
systemctl reload apache.service

// 重载所有修改过的配置文件
systemctl daemon-reload

// 显示某个 Unit 的所有底层参数
systemctl show httpd.service

// 显示某个 Unit 的指定属性的值
systemctl show -p CPUShares httpd.service

// 设置某个 Unit 的指定属性
systemctl set-property httpd.service CPUShares=500

//检查服务是否开机启动
systemctl is-enabled crond
```

#### 与`target`有关的常用命令


```
// 查看当前系统的所有 Target
systemctl list-unit-files --type=target

// 查看一个 Target 包含的所有 Unit
systemctl list-dependencies multi-user.target

// 查看启动时的默认 Target
systemctl get-default

// 设置启动时的默认 Target(类似于centos6之前设置启动级别)
systemctl set-default multi-user.target

// 切换 Target 时，默认不关闭前一个 Target 启动的进程，而是关闭前一个 Target 里面所有不属于后一个 Target 的进程
systemctl isolate multi-user.target
```


## 参考资料：

* [Linux下chkconfig命令及守护进程](https://my.oschina.net/junn/blog/215825)

* [《chkconfig命令》－linux命令五分钟系列之四](http://roclinux.cn/?p=51)

* [Systemd 入门教程：命令篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)

* [用systemd命令来管理linux系统](https://linux.cn/article-3801-1.html)

* [systemctl 命令完全指南](https://linux.cn/article-5926-1.html)

* [LINUX PID 1 和 SYSTEMD](https://coolshell.cn/articles/17998.html)

* [Ubuntu下使用sysv-rc-conf管理服务](https://blog.csdn.net/gatieme/article/details/45251389)


