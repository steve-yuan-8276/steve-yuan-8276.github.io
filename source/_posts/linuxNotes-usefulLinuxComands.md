---
title: 'linux学习笔记：有用的linux命令'
date: 2018-06-25 14:34:26
categories: linux
tags: [linux, notes]
---

## 写在前面

这着笔记，整理一些网上搜集到有用的linux笔记。

## sosreport命令

sosreport命令用于收集系统构架及配置信息，并打包输出为诊断文档。当我们系统中出现问题，自己无法搞定的时候，可以使用这个命令搜集全面的系统诊断信息。

### 安装sosreport

```
[root@local-linux02 ~]# yum install -y sos
```

### 使用

```
//生成报告  期间会有几次提示，直接enter即可
[root@local-linux02 sosreport]# sosreport

//复制报告到指定文件夹
[root@local-linux02 sosreport]# mkdir -p /home/sosreport && cp /var/tmp/sosreport-local-linux02-20180625142856.tar.xz /home/sosreport/

//解压报告
[root@local-linux02 sosreport]# tar -Jxvf sosreport-local-linux02-20180625142856.tar.xz

//重命名报告
[root@local-linux02 sosreport]# mv sosreport-local-linux02-20180625142856 sosreport
```

<!--more-->

- 查看报告

![](https://farm2.staticflickr.com/1766/42995285191_8272fa6769_o.png)

```
[root@local-linux02 sosreport]# cat uname
Linux local-linux02 3.10.0-862.3.2.el7.x86_64 #1 SMP Mon May 21 23:36:36 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
[root@local-linux02 sosreport]# cat uptime
 14:29:04 up 23:09,  1 user,  load average: 0.22, 0.08, 0.06
```

## diff命令

用来比较两个文件的异同，常见用法有两种：

### 用法一： diff --brief 判断两个文件是否相同

- 文件准备a.txt、b.txt

```
[root@local-linux02 test]# cat a.txt
www.baidu.com
www.163.com
www.qq.com
www.yaho.com
www.google.com
[root@local-linux02 test]# cat b.txt
www.baidu.com
www.163.com
www.qq.com
www.yahoo.com
www.google.com
cafevf
3e2edwefew
```

查看是否相同

```
[root@local-linux02 test]# diff --brief a.txt b.txt
Files a.txt and b.txt differ
```

### 用法二： diff -c 判断文件哪里不同

![](https://farm1.staticflickr.com/891/42946073752_a78a971b35_o.png)

## `stat` 命令 和 `touch` 命令

**注意：这里有一个非常重要的知识点，就是linux文件系统的三个时间。**


`mtime` 文件内容的修改时间；

`ctime` 文件权限或属性的更改时间；

`atime` 文件的读取时间；

### `stat` 命令用来查看文件时间信息。

命令行格式：

```
stat [fileName]
```

示例：

```
[root@local-linux02 ~]# stat anaconda-ks.cfg
  File: ‘anaconda-ks.cfg’
  Size: 1421      	Blocks: 8          IO Block: 4096   regular file
Device: 803h/2051d	Inode: 25165890    Links: 1
Access: (0600/-rw-------)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2018-06-25 14:28:57.699307060 +0800
Modify: 2018-04-03 19:01:25.048986365 +0800
Change: 2018-04-03 19:01:25.048986365 +0800
 Birth: -
```

### `touch` 命令 用来新建文件或设置文件的时间。

命令行格式：

```
touch [参数] [filename]
```
 
参数| 说明
---|---
-a | 仅更改“读取时间” atime
-m | 仅更改“修改时间” mtime
-d | 同时更改atime和mtime


示例：先用`ls -l` 查看文件的mtime，之后修改文件，最后在通过`touch -d`指定文件的mtime和atime

```
//通过ls -l查看文档的mtime
[root@local-linux02 test]# ls -l a.txt
-rw-r--r-- 1 root root 65 Jun 25 15:06 a.txt

//修改文档内容
[root@local-linux02 test]# echo 'this is test' >> a.txt
[root@local-linux02 test]# ls -l a.txt
-rw-r--r-- 1 root root 78 Jun 25 15:23 a.txt

//touch -d设置mtime和atime
[root@local-linux02 test]# touch -d '2018-6-25 15:06' a.txt
[root@local-linux02 test]# ls -l a.txt
-rw-r--r-- 1 root root 78 Jun 25 15:06 a.txt
```

## dd 命令：通过数据块的大小和格式来生成文件

命令行格式：

```
dd [选项]
```

常见选项包括：

* if =输入文件（或设备名称）； 

* of =输出文件（或设备名称）； 

* bs = bytes 同时设置读/写缓冲区的字节数（等于设置ibs和obs）； 

* count=blocks 只拷贝输入的blocks块； 

常见用法：

### 应用场景一：生成swap交换空间

```
//创建一个大小为256M的文件：
dd if=/dev/zero of=/swapfile bs=1024 count=262144

//把这个文件变成swap文件：
mkswap /swapfile

//启用这个swap文件：
swapon /swapfile

//编辑/etc/fstab文件，使在每次开机时自动加载swap文件：
/swapfile    swap    swap    default   0 0
```

**说明：**`/dev/zero` 是linux系统中一个很神奇的文件，它本身不占用系统存储空间，却可以生出任意大小的存储空间。

### 应用场景二： 制作光盘镜像

- 第一步：将U盘插到电脑上，然后打开终端，输入命令`sudo fdisk -l`或`sudo parted -l`命令查看U盘的设备号

- 第二步：执行以下命令制作光盘镜像

```
sudo dd if=Downloads/ubuntu-14.10-desktop-amd64.iso of=/dev/sdb 
```

## `grep` 命令：与管道连用，用来过滤搜索结果

三种使用场景：

- `-n` 显示搜索信息的行号

```
[root@local-linux02 ~]# cat /etc/passwd | grep -n root
1:root:x:0:0:root:/root:/bin/bash
10:operator:x:11:0:operator:/root:/sbin/nologin
```

- '-v' 反选，即不包括搜索关键词的行

```
[root@local-linux02 ~]# cat /etc/passwd | grep -v nologin
root:x:0:0:root:/root:/bin/bash
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
user1:x:1000:1000::/home/user1:/bin/bash
user2:x:1001:1001::/home/user2:/bin/bash
```

- '-i' 忽略大小写

```
[root@local-linux02 test]# cat ~/test/a.txt | grep -i qq
www.qq.com
www.QQ.com

[root@local-linux02 test]#cat ~/test/a.txt
www.baidu.com
www.163.com
www.qq.com
www.yaho.com
www.google.com
this is test
www.QQ.com
```


