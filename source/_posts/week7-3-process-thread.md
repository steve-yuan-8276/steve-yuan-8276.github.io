---
title: 'Linux运维学习笔记：进程和线程'
date: 2018-05-08 22:05:04
categories: linux
tags: [linux, pid, tid, notes] 
---

### 写在前面

这部分笔记是学习`ps` 命令的拓展知识，`ps`命令的用法在[这里](https://www.steve-yuan.com/2018/05/03/week7-2-howToManageLinuxPart2/)，不再赘述。

### 进程和线程

### CPU工作的基本原理

要理解进程和线程，先要大概了解一下CPU的工作原理：

电脑的核心是CPU，它承担了计算机所有的计算任务，就像一个永不停歇的工厂。但是，CPU又个特点，就是一个核心同时只能执行一个任务，一心不可二用。

即便是对于多核CPU而言，每个核心在单位时间内也只能执行一个任务：

* 总线程数 <= CPU数量：并行运行

* 总线程数 > CPU数量：并发运行

为了优化CPU运行，就有了必须使用并发技术，其中最容易理解的是**“时间片轮转进程调度算法”**，大致原理如下：在操作系统的管理下，所有正在运行的进程轮流使用CPU，每个进程允许占用CPU的时间非常短(比如10毫秒)，虽然单位时间内每个核心有且只有一个进程，但由于进程快速轮换，用户感觉不出来，就好象所有的进程都在不间断地运行一样。

<!--more-->

#### 进程和线程的基本概念

**进程**` process`：是程序在计算机上的一次执行活动，是操作系统分配资源的单位。当你运行一个程序，你就启动了一个进程。

**线程** `Thread`：是进程的一个实体，是CPU调度和分派的基本单位。

#### 线程和进程的关系：

* 一个线程只能属于一个进程，而一个进程可以有多个线程，但至少有一个线程（通常说的主线程）；

* 资源分配给进程，同一进程的所有线程共享该进程的所有资源；

* 线程在执行过程中，需要协作同步。不同进程的线程间要利用消息通信的办法实现同步；

* 处理机分给线程，即真正在处理机上运行的是线程；

* 线程是指进程内的一个执行单元，也是进程内的可调度实体。

两者的区别在于：

* 调度：线程作为调度和分配的基本单位，进程作为拥有资源的基本单位；

* 并发性：不仅进程之间可以并发执行，同一个进程的多个线程之间也可以并发执行；

* 拥有资源：进程是拥有资源的一个独立单位，线程不拥有系统资源，但可以访问隶属于进程的资源；

#### 操作系统的分类

上面了解了cpu工作原理，以及进程和线程的简单知识。需要了解的是，进程和线程如何执行，并不是有它自己决定的，而是由操作系统决定的。

根据进程和线程的特点，有两种解决方案：

* 一是启动多个进程，每个进程虽然只有一个线程，但多个进程可以一块执行多个任务；

* 二是启动一个进程，在一个进程内启动多个线程，这样，多个线程也可以一块执行多个任务；

所以，操作系统要想实现多任务处理，无非三种途径：

1. 多进程模式；

2. 多线程模式；

3. 多进程+多线程模式


#### pid和tid

Linux的线程是“轻量级进程”（Light-Weight Process，即LWP）。在Linux系统上运行一个程序时，操作系统会为这个程序创建一个进程，也就是“主线程”，后续则可以产生出更多的线程。每个进程都有一个PID（Process ID），每个线程也会有一个TID（Thread ID），属于同一进程的线程各自有拥有不同的TID，但它们的PID是相同的，都等于“主线程”的TID。

从本质上来讲，Linux系统下的进程和线程没有区别，只不过同一进程中的线程可以共享某些资源。

示例一：

```
[root@stevey bin]# cd /usr/local/sbin     //进入脚本目录
[root@stevey sbin]# ls
createFile.sh  even.sh  fun1.sh  option1.sh  sum1.sh  while1.sh
df1.sh         for1.sh  if1.sh   read1.sh    var1.sh
[root@stevey sbin]# bash even.sh &       //运行脚本
[1] 16282
[root@stevey sbin]# please in number: bg    //把脚本后台运行
[1]+ bash even.sh &
[root@stevey sbin]# ps -T 16282          //ps查询pid信息
   PID   SPID TTY      STAT   TIME COMMAND
 16282  16282 pts/1    T      0:00 bash even.sh

```

上面的例子实际只运行了一个进程/进程，所以不明显；网上上找了一个示例如下：

```
#include <unistd.h>
#include <omp.h>

int main(void){

        #pragma omp parallel num_threads(4)
        for(;;)
        {
            sleep(1);
        }

        return 0;
}
```

编译并在后台运行这个程序：

```
$ gcc -fopenmp threads.c
$ ./a.out &
[1] 9802
```

进程的PID是9802，用ps -T pid命令查看进程的线程信息：

```
$ ps -T 9802
  PID  SPID TTY      STAT   TIME COMMAND
 9802  9802 pts/1    Sl     0:00 ./a.out
 9802  9803 pts/1    Sl     0:00 ./a.out
 9802  9804 pts/1    Sl     0:00 ./a.out
 9802  9805 pts/1    Sl     0:00 ./a.out
```

其中SPID即为TID。可以看到当前进程的PID是9802，共包含4个线程，其TID依次为：9802，9803，9804和9805，其中PID和SPID相同的线程即为主线程。


### 参考资料

* [进程和线程的区别](https://www.cnblogs.com/lmule/archive/2010/08/18/1802774.html)

* [腾讯面试题04.进程和线程的区别？](https://blog.csdn.net/mxsgoden/article/details/8821936)

* [进程与线程的一个简单解释](http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)

* [进程和线程](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0013868322563729e03f6905ea94f0195528e3647887415000)

* [进程和线程有什么区别？](https://www.zhihu.com/question/21535820)

* [Linux线程模型浅析](http://nanxiao.me/linux-thread-model-analysis/)


