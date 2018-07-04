---
title: 'Linux运维学习笔记7-2：监控系统状态命令2.1'
date: 2018-05-03 10:34:40
categories: linux
tags: [linux, tcpdump, wireshark, notes] 
---

### 写在前面

接着上一期：[Linux运维学习笔记7-1：监控系统状态命令（一）](https://www.steve-yuan.com/2018/04/28/week701-howToManageLinux/)，继续整理监控系统状态的命令，包括free、ps、监控IO性能、Linux抓包方面的内容。

**注：**因为这则笔记太长，上传到博客后总是格式错误，所以拆成了两节分别上传，请老师见谅。

### 监控IO性能

#### `iostat` 命令 监视磁盘I/O输入输出情况

`iostat` 与 `sar` 同属于sysstat包。

命令格式：

```
iostat[参数][时间][次数]
```

命令参数|解释
---|---
-C |显示CPU使用情况
-d |显示磁盘使用情况
-k |以 KB 为单位显示
-m |以 M 为单位显示
-N |显示磁盘阵列(LVM) 信息
-n |显示NFS 使用情况
-p[磁盘] |显示磁盘和分区的情况
-t |显示终端和CPU的信息
-x |显示详细信息
-V |显示版本信息

<!--more-->

**常见用法：**

```
iostat -x       //监控磁盘I/O的详细信息
```

![](https://farm1.staticflickr.com/966/27077058157_6565c31b01_o.png)

上图中，重点关注`%util`这一列，即，一秒中有百分之多少的时间用于 I/O 操作，即被io消耗的cpu百分比。

如果 `%util` 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明I/O 队列太长，io响应太慢，则需要进行必要优化。如果avgqu-sz比较大，也表示有大量io在等待。

#### `iotop` 命令

iostat 只能统计到每个设备到读写情况，如果想知道每个进程是如何使用I/O的使用情况，就需要`iotop`。

安装：

```
yum install -y iotop
```

基本操作：

```
iotop    //查看进程I/O使用情况
```

![](https://farm1.staticflickr.com/863/41946192541_4d3a3c0a2d_o.png)

键位|作用
---|---
左右箭头|改变排序方式，默认是按IO排序；
r|改变排序顺序；
o|只显示有IO输出的进程；
p|进程/线程的显示方式的切换；
a|显示累积使用量；
q|退出；

这里出现了一个新的大小TID，网上搜索到[TID和PID的区别](https://codeday.me/bug/20171005/81320.html)，有兴趣看下。

### `free` 命令：查看内容使用情况

`free` 命令显示系统使用和空闲的内存情况，包括物理内存、交互区内存(swap)和内核缓冲区内存。在Linux系统监控的工具中，free命令是最经常使用的命令之一。

命令行格式：`free [参数]`

参数| 含义
---|---
-b 　|以Byte为单位显示内存使用情况；
-k 　|以KB为单位显示内存使用情况；
-m 　|以MB为单位显示内存使用情况；
-g |以GB为单位显示内存使用情况；
-o |不显示缓冲区调节列；
-s <间隔秒数> |持续观察内存使用状况； 
-t 　|显示内存总和列；
-V 　|显示版本信息；
-h |自适应选择合适的单位

#### `free` 能查看哪些信息？

![](https://farm1.staticflickr.com/972/26990814707_bee586145e_o.png)

列名|含义
---|---
total|内存总大小；**total = used + free + buff/cache**
used|真正使用的内存大小；
free|剩余物理内存的大小（还没有被分配，纯剩余）；
shared|共享内存大小
buff/cache|buff是经过CPU处理，即将**写入**磁盘的数据占用的内存大小；cache是从磁盘**读取**数据占用内存的大小；
available|系统可使用内存大小；**available = free + buff/cache（未被系统程序占用的部分）**

**这里需要记忆两个公式：**

* `total = used + free + buff/cache `

* `available = free + buff/cache（未被系统程序占用的部分）`

**注：** 

linux为了更大限度地使用系统资源，会事先把一部分内存分出来预留给应用程序，这部分就叫做`buff/cache`；即便这部分内存没有被真正使用，也已经打上了`buff/cache`的标签。

当然，如果系统需要，也可以把这部分内容拿来使用。

### `ps` 命令：查看系统进程

命令行格式： `ps [参数]`

参数| 含义
---|---
a  |显示所有进程
-a |显示同一终端下的所有程序
-A |显示所有进程
c  |显示进程的真实名称
-N |反向选择
-e |等于“-A”
e  |显示环境变量
f  |显示程序间的关系
-H |显示树状结构
r  |显示当前终端的进程
T  |显示当前终端的所有程序
u  |指定用户的所有进程
-au |显示较详细的资讯
-aux |显示所有包含其他使用者的进程 
-C<命令> |列出指定命令的状况
--lines<行数> |每页显示的行数
--width<字符数> |每页显示的字符数
--help |显示帮助信息
--version |显示版本显示

#### ps命令支持三种使用的语法格式

* UNIX 风格，选项可以组合在一起，并且选项前必须有一个“-”连字符，如 `ps -aux`;

* BSD 风格，选项可以组合在一起，但是选项前不能有“-”连字符,如 `ps aux`;

* GNU 风格的长选项，选项前有两个“-”连字符，如` ps --aux`;

#### `ps `命令能查看什么信息？

##### 不带参数执行ps命令

![](https://farm1.staticflickr.com/961/41816621922_28f45435a8_o.png)

结果默认会显示4列信息,具体如下：

* PID: 运行着的命令(CMD)的进程编号；

* TTY: 命令所运行的位置（终端）；

* TIME: 运行着的该命令所占用的CPU处理时间；

* CMD: 该进程所运行的命令；

##### `-aux` 显示所有包含其他使用者的进程 

由于进程较多，建议结合`less` 进行查看：

```
ps -aux | less
```

![](https://farm1.staticflickr.com/905/26991280507_ec924b592f_o.png)

状态码 | 进程状态
---|---
D|不可中断 uninterruptible sleep (usually IO) 
R|运行 runnable (on run queue) 
S|大写S，中断 sleeping,如果`ctrl+z`暂停地进程
T|停止 traced or stopped 
Z|僵尸进程 a defunct (”zombie”) process 
<|高优先级进程
N|低优先级进程
s|小写s，主进程
L|在内存中被锁了内存分页
l|多线程进程
+|在前台运行的进程

#### 常见用法：

##### 查看某个进程

```
ps aux | grep sshd
```

![](https://farm1.staticflickr.com/828/27990981008_9aa38d0071_o.png)

##### 查看某个进程的数量

注：适用grep命令时，grep本身也算一个进程，因此查询结果减1才是真正的进程数量

```
[root@stevey ~]# ps -aux | grep -c sshd
3

```

### 参考资料：

* [通俗大白话来理解TCP协议的三次握手和四次分手](https://github.com/jawil/blog/issues/14)

* [tcp的三次握手和四次挥手](http://blog.51cto.com/10690709/2113525)

* [Linux查看应用可用内存-free命令详解](https://blog.csdn.net/loongshawn/article/details/51758116)

* [10个重要的Linux ps命令实战](https://linux.cn/article-4743-1.html)

* [每天一个linux命令（41）：ps命令](https://www.cnblogs.com/peida/archive/2012/12/19/2824418.html)

* [netstat 的10个基本用法](https://linux.cn/article-2434-1.html)

* [Linux抓包工具tcpdump详解](http://www.ha97.com/4550.html)

* [Wireshark命令行工具tshark使用小记](https://www.cnblogs.com/liun1994/p/6142505.html)

