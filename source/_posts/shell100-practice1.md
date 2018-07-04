---
title: 'linux学习笔记：shell练习第1题'
date: 2018-05-03 17:54:37
categories: linux
tags: [linux, shell, notes] 
---

### 第1题

按照年-月-日 `XXXX-XX-XX` 格式每天生成一个日志文件，记录每天的磁盘使用情况。

#### 相关知识点

##### `date` 命令：显示系统时间

`date`命令用来显示或者设置系统的日期和时间，是shell脚本当中最常用的命令，

命令行格式： 

```
date [参数] [+格式]
```

详细的用法，参见[这里](https://www.steve-yuan.com/2018/04/27/linuxNotes-howToWriteShellScript/)。

常见格式 | 含义
---|---
%y	|以两位数字表示年份(00-99)
%Y	|以四位数字表示年份
%m	|月份(01-12)
%d	|按月计的日期(例如：01)

<!--more-->

##### `df` 命令：查看磁盘使用情况

命令行格式

```
 df [参数]
```

这个命令比较简单，常见的参数有 -k（KB）、 -m（MB）、 h（自适应选择存储单位），以及i（查看inode使用情况）；

更多的用法介绍在[这里](https://www.steve-yuan.com/2018/04/04/week3-4-basisOfDisksetting/)

#### shell脚本

```
#! /bin/bash   

## In this script, we'll create disk-usage log

date=`date +%Y-%m-%d`     //定义日期格式
logfile=$date.log         
df -h > $logfile
```


