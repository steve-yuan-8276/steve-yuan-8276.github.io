---
title: 'linux笔记：计划任务服务'
date: 2018-06-25 17:37:26
categories: linux
tags: [linux, at, crond, anacron, notes]
---

## 写在前面

之前学习第八章的时候，整理过linux计划任务方面的内容，参见[Linux运维学习笔记8-1：系统定时任务](https://www.steve-yuan.com/2018/05/10/week8-1-linuxnotes-cron/)，但当时并不太懂，今天看资料的时候恰好又看到这方面的内容，再来重新整理一下。

所谓计划任务服务，就是通过设定，让linux系统在无人值守的情况下，在指定时间，自动执行某些特定的命令或者服务，从而实现运维自动化。

这里主要整理了三个工具，分别是at、crond、anacron，分别整理如下：

## `at`命令

**应用场景：**指定的时间执行一个指定任务，且只能执行一次。

**命令行格式：**

```
at [参数] [时间]
```

**常用参数：**

* `-v` 显示任务将被执行的时间；

* `-c` 打印任务的内容到标准输出；

* `-f<文件>` 从指定文件读入任务而不是从标准输入读入；

at命令依赖atd服务，因此使用它的先决条件就是atd服务已经启动。

![](https://farm2.staticflickr.com/1773/43013350581_fbe2d8ef8c_o.png)

<!--more-->

如果没有启动，显示dead的话，需要执行以下命令重新启动：

```
[root@local-linux02 ~]# systemctl restart atd
```

### 使用方式一：默认为交互式输入

```
//5pm+3 days 表示三天后的下午5点
[root@localhost ~]# at 5pm+3 days   
at> /bin/ls
at> <EOT>    按下control + d 组合键结束输入
```

### 使用方式二：结合管道符号

```
echo 'systemctl restart nginx' | at 23:30
```

### 其他常用操作

`at -l` 列出当前的任务

![](https://farm2.staticflickr.com/1811/41203514040_ea614758fd_o.png)

`atrm [任务序号]` 删除指定的任务

```
//删除编号为3的定时任务
[root@local-linux02 ~]# atrm 3
//校验是否删除
[root@local-linux02 ~]# at -l
2	Tue Jun 26 23:30:00 2018 a root
```

## `cron` 服务：服务器周期性有规律滴执行特定任务

cron是一个用于运行计划任务如系统备份、更新等的守护进程。它适合在那些 24X7 不间断运行的机器如服务器上运行的计划任务。

应用场景：假设我们有一台服务器，需要定期执行某项操作，比如备份，就适合使用crond服务。

**注意：**crond服务有一个重要前提，就是假设系统会一直开机，所以如果恰好关机了，就会出问题。比如，设定了一个23:30分的备份操作，但是在23:29分断电了，过五分钟又来电了，这个备份操作就不会被执行。

### crond服务的设置方法：

crond服务，实质上是调用vim对定时任务进行编辑，退出保存后，被保存在`/var/spool/cron/[username]`。比如，当前笔者是在用root用户操作，默认设置的就被保存在`/var/spool/cron/root`文件当中。

**常用命令包括：**

* `crontab -e`  创建编辑定时任务；

* `crontab -l`  查看当前的定时任务；

* `crontab -r`  删除指定的定时任务；

* `crontab -u`  以管理员身份修改其他用户创建的定时任务（当前用户必须是root用户）；

#### 部署定时任务工作流：

1. crontab -e 进入编辑状态；

2. 写入定时任务，格式为：`min hour date month week command`;

字段 | 说明
---|---
min |分钟，0-59分之间的任意整数，不允许为空或*号
hour|小时，0-23之间的任意整数
date|日期，1-31之间的任意整数
month|月份，1-12之间的任意整数
week|星期几，0-7之间的任意整数，其中0和7均表示星期天
command|需要执行的命令或脚本，命令需要写绝对路径


#### 示例：

- 新建定时任务

```
[root@local-linux02 ~]# crontab -e

//在打开的编辑页面输入以下指令
//意思是每天23:30 重启nginx
30 23 * * * /usr/sbin/nginx restart
```

- 查看当前定时任务

![](https://farm2.staticflickr.com/1795/42296122784_65f37979fd_o.png)

- 删除定时任务

```
[root@local-linux02 ~]# crontab -r
[root@local-linux02 ~]# crontab -l
no crontab for root
```

## anacron 工具

anacron 不按照小时、分钟来执行定时任务，而是**以天为单位运行定时任务**。它的工作与 cron 稍有不同，它假设机器不会一直开机，因此更适合用在个人电脑上面。

也正因为anacron 是以天为单位的，并且也不是守护进程，所以它并不能完全取代cron，而只能作为cron的补充存在。

### 安装启动annacron

![](https://farm2.staticflickr.com/1809/41206108840_888cc974b4_o.png)

### anacron基本语法


```
anacron [options] [job]
```

常用选项如下：

* `-s`：重要！开始所有的jobs，并根据时间戳判断是否开始相应的工作job；

* `-f`：强制进行，忽略时间戳的对比；

* `-n`：立即进行未进行的任务，忽略延迟等待时间；

* `-u`：仅升级相应的执行时间戳，实际不进行任何工作；

* `job`：/etc/anacrontab文件中定义的各项工作名称；


### anacron任务是如何被执行的？

前文说过anacron是cron的有力补充，它是一个普通进程，而不是linux守护进程，也不是服务。这点很重要！！！

举个例子，我们在使用nginx的时候，会使用`systemctl start nginx` 和 `systemctl enable nginx`,让nginx随开机启动，并常驻在后台，但是在网上搜了一圈，也没有找到怎么启动后台守护进程的命令。那么，它是怎么被执行的呢？

#### 第一步：cron服务每小时读取0anacron脚本

实际上，anacron的任务加载配置文件被写进了cron当中，有一个很神奇的脚本文件，叫做`/etc/cron.hourly/0anacron`，关键的秘密都在这里。

![](https://farm2.staticflickr.com/1774/41205511850_6f472e8ac7_o.png)

与anacron任务运行的相关配置文件包括：

* `/etc/anacrontab`

* `/var/spool/anacron/cron.daily`

* `/var/spool/anacron/cron.monthly`

* `/var/spool/anacron/cron.weekly`

#### 第二步：加载`/etc/anacrontab` 配置文件

在运行`/etc/cron.hourly/0anacron`之后，会执行`/usr/sbin/anacron -s`命令，重新加载配置文件`/etc/anacrontab`的内容，而我们**的定时任务，就是通过编辑该配置文件，保存后即可呗调用执行。**

`/etc/anacrontab` 对于执行的方式做了具体的规定：

![](https://farm2.staticflickr.com/1802/29143625118_3884a4b0c4_o.png)

其中，红框框出来的几个部分很关键

先看定时任务的写法，共分为四个部分：

```
#period in days   delay in minutes   job-identifier   command
1	5	cron.daily		nice run-parts /etc/cron.daily
```

* `period in days`： 任务执行的频率，以天为单位。可以是@daily（每天） 、@weekly（每周）、@monthly（每月）；也可以使用数字：1代表每天、7代表每周、30代表每月，或者直接用N指定以N天为一个周期；

* `delay` ： 这是在执行一个任务前等待的分钟数；

* `job-id` - 这是写在日志文件中任务的任务标示，任务标识，这个与/var/spool/anacron/ 目录下的时间戳文件名相一致，anacron任务通过此标识来获取对应的时间戳判断是否执行该任务；

* `command` - 这是要执行的命令或 shell 脚本；

还有两个配置选项也很重要：

`RANDOM_DELAY=45`：anacron任务添加的随机延迟的最大值，单位为分钟，一个anacron任务的运行的总延迟时间为`RANDOM_DELAY+Base_delay`；

`START_HOURS_RANGE=3-22`：设置anacron任务执行的时间范围，此处表示anacron任务只能在3am-22pm的时间范围内执行；

#### 第三步：执行定时任务

1. anacron 会生成任务的时间戳，保存在 `/var/spool/anacron/cron.daily` 文件中；

2. 在执行的过程中，会读取 `/var/spool/anacron/cron.daily` ，对比当前时间戳和上一次的时间戳，如差异天数为1天或以上，就会执行相应的任务；

3. 任务执行完毕，修改对应的时间戳文件为当前时间，anacron程序结束。

困扰多时的谜题终于解开，心情大好。

## 参考资料

* [anacron命令开机唤醒计划任务](https://www.jianshu.com/p/07cae3325854)

* [Linux计划任务之 anacron](https://www.benderfly.com/291.html)

* [如何使用Anacron在Linux上安排作业](https://www.howtoing.com/cron-vs-anacron-schedule-jobs-using-anacron-on-linux)

