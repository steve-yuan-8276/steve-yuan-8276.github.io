---
title: 'Linux运维学习笔记8-1：系统定时任务'
date: 2018-05-10 10:57:12
categories: linux
tags: [linux, crontab, systemd, notes] 
---

### 写在前面

这则笔记整理linux定时任务管理的内容，涉及的命令主要包括crontab、anacron 等。

注：系统管理的部分单独剥离出来了，重新整理了一个笔记，[在这里](https://www.steve-yuan.com/2018/05/24/week8-2-linuxNotes-systemd-sysv/#more)。

### Linux的定时任务管理 `crontab`

`crontab` 是linux系统中内置的一个服务，用来管理定时任务。比如，我们写了一个脚本，需要定时执行，就要用到`crontab`

#### crontab 服务启用及停止

- 安装 Crontab 服务    **注：**centos 7默认已经安装

```
yum install -y crontabs
```

- 启用服务        注：默认此服务是没有启动的

```
systemctl start crond
```

- 停止服务

```
systemctl stop crond
```

- 查看服务状态

```
systemctl status crond
```

- 加入开机启动

```
systemctl enable crond
```

- 取消开机启动

```
systemctl disable crond
```

- 备份 crontab 文件（以root用户为例）

```
cp /var/spool/cron/root /var/spool/cron/root.backup
```

<!--more-->

#### crontab 配置文件

注；crontab的配置文件在`/etc/crontab`，既然是定时任务，下面这部分就是演示如何表示时间的，很有用。

```
[root@stevey ~]# cat /etc/crontab      //查看crontab配置文件
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)    //表示分数（0-59）
# |  .------------- hour (0 - 23)      //表示24小时制（0-24）
# |  |  .---------- day of month (1 - 31)    //表示天（1-31）
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...   //表示月（1-12），也可以用英文简写也表示
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat   //表示星期几，1-6分别对应星期一至星期六；星期天可以写为0或者7
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

```

在表达时间的字段中，还可以使用以下特殊符号：

* `*`星号：代表所任意值，如果在month字段使用，就是每个月都执行；

* `,`逗号：用逗号隔开的值，指定一个列表范围，例如，“1,2,5,7,8,9”；

* `-`短横线：整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”；

* `/`斜线：指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次，对应的时间就是0、2、4、6、8、10、12、14、16、18、20、22；斜线可以和星号一起使用，如果在表示分钟的字段出现*/10，表示每十分钟执行一次。

##### 跟crontab相关的其他配置文件

* `/var/spool/cron/` 所有用户 crontab 任务文件存放的目录,文件名即用户名

* `/etc/cron.deny` 该文件中所列的用户不允许使用 crontab 命令

* `/etc/cron.allow` 该文件中所列的用户允许使用 crontab 命令

* `/etc/cron.d/` 该目录用来存放任何要执行的crontab文件或脚本

#### `crontab`使用

命令行格式：

```
crontab [options] file
crontab [options]
crontab -n [hostname]
```

常用选项|含义
---|---
-u <user>  |define user 设定某个用户的crontab服务；如果不指定用户，则编辑当前用户的crontab文件
-e         |edit user's crontab 进入编辑状态；
-r         |delete user's crontab 删除某个用户的 crontab 文件；
-l         |列出任务计划

`crontab -e` 进入编辑状态，实际相当于调用vim对定时任务进行编辑，退出保存后，被保存在`/var/spool/cron/[username]`。比如，当前笔者是在用root用户操作，默认设置的就被保存在`/var/spool/cron/root`文件当中。

修改编辑器，可以通过自定义环境变量的方式：

```
vim ~/.bashrc        //编辑当前当前用户的配置文件；也可以编辑 /etc/bashrc来更改全局用户变量

export EDITOR=vim    //添加这一行，意思是指定vim作为默认编辑器；之后保存退出

source ~/.bashrc     //重新加载配置文件

```

**注：**

1. 涉及路径时，需要使用绝对路径。

2. crontab 任务都是保存了之后立即执行的，但是有的时候却无法执行，将命令单独拿出来却可以使用，这个时候需要检查一下 crontab 文件的环境变量是否正常。


##### 使用实例

- 每1分钟执行一次Command

```
* * * * * Command
```

- 每小时的第3和第15分钟执行

```
3,15 * * * * Command
```

- 在上午8点到11点的第3和第15分钟执行

```
3,15 8-11 * * * Command

```

- 每隔两天的上午8点到11点的第3和第15分钟执行

```
3,15 8-11 */2  *  * myCommand
```

- 每周一上午8点到11点的第3和第15分钟执行

```
3,15 8-11 * * 1 myCommand
```

- 每晚的21:30重启smb

```
30 21 * * * /etc/init.d/smb restart
```

- 每月1、10、22日的4 : 45重启smb

```
45 4 1,10,22 * * /etc/init.d/smb restart
```

- 每周六、周日的1 : 10重启smb

```
10 1 * * 6,0 /etc/init.d/smb restart
```

- 每天18 : 00至23 : 00之间每隔30分钟重启smb

```
0,30 18-23 * * * /etc/init.d/smb restart
```

- 每星期六的晚上11 : 00 pm重启smb

```
0 23 * * 6 /etc/init.d/smb restart
```

- 每一小时重启smb

```
* */1 * * * /etc/init.d/smb restart
```

- 晚上11点到早上7点之间，每隔一小时重启smb

```
0 23-7 * * * /etc/init.d/smb restart
```


#### anacron 安装：

- 检查是否安装：

```
[root@stevey system]# rpm -q anccron
package anccron is not installed
```

- yum安装：

```
yum install -y anacron
```

#### `anacron` 工具

`anacron` 跟 `cron` ，也是管理定时任务的命令行工具，但它又一个最大的不同，就是**假设机器不会一直开机。**

假设你有一个计划任务（比如备份脚本）要使用 cron 在每天半夜运行，但那时你的笔记本电脑已经关机，备份脚本就不会被运行。`anacron` 就是来解决这个痛点的。

**anacron与crontab 对比**

crontab	|anacron
---|---
适合服务器	|适合桌面/笔记本电脑
它是守护进程	|它不是守护进程
关机时不会执行计划任务	|如果计划任务到期，机器是关机的，那么它会在机器下次开机后执行计划任务
可以让你以分钟级运行计划任务	|只能让你以天为基础来运行计划任务
普通用户和 root 用户都可以使用	|只有 root 用户可以使用（使用特定的配置启动普通任务）
#### anacron 配置文件

![](https://farm1.staticflickr.com/961/40222449090_96b9c1bd27_o.png)

anacron的配置文件是 /etc/anacrontab，而它的很多内容则是在/var/spool/anacron里面保存。

执行计划由四部分组成：

* `period in days`：执行周期，表示任务多少天执行一次。

* `delay in minutes`：延迟时间（分钟）

* `job-identifier`：任务的描述，用在 anacron 的消息中，并作为作业时间戳文件的名称，只能包括非空白的字符（除斜线外）。

* `command`：实际运行的命令。这里的 run-parts 是一个运行指定目录中所有程序与脚本的命令，可以通过 man run-parts 来查看它的说明。

当anacron下达anacron -s cron.daily时，它会有如下的步骤：

1. 由/etc/anacrontab分析到cron.daily这项工作名称的天数为一天;

2. 由/var/spool/anacron/cron.daily取出最近一次运行anacron的时间戳;

3. 把取出的时间戳与当前的时间戳相比较，如果差异超过了一天，那么就准备进行命令;

4. 若准备进行命令，根据/etc/anacrontab的配置，将延迟65分钟;

5. 延迟时间后，开始运行后续命令，也就是run-parts /etc/cron.daily这串命令;

6. 运行完毕后，anacron程序结束。

#### anacron 用法

命令行格式：

```
anacron [参数] [作业名] [参数]

```

常用参数	|说明
---|---
-d	  |anacron命令把信息输出到标准输出设备和系统日志中，同时作业的输出也以E-mail的形式发出
-f	  |强制执行作业，忽略时间戳
-h	  |显示在线帮助信息
-n	  |这个参数隐含了-s参数，马上开始执行作业，忽略/etc/anacrontal文件中的延迟
-q	  |禁止向标准输出发送信息，只能和-d参数配合使用
-s	  |在前一个作业完成以前anacron不会开始新的作业
-t<文件名>	  |使用指定的配置文件，忽略默认的/etc/anacron
-u	  |更新作业的时间戳，但不执行作业
-V	  |显示版本信息




### 参考资料

* [crontab 定时任务](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/crontab.html)

* [Linux任务管理工具之Crontab](https://www.ctolib.com/topics-83224.html)

* [linux定时运行命令脚本——crontab](https://blog.csdn.net/ithomer/article/details/6817019)

* [cron 与 anacron：如何在 Linux 中计划任务](https://linux.cn/article-8590-1.html)

* [anacron命令](https://blog.csdn.net/zyc_love_study/article/details/78893619)

* [Linux守护进程(init.d和xinetd)](https://www.cnblogs.com/itech/archive/2010/12/27/1914846.html)


