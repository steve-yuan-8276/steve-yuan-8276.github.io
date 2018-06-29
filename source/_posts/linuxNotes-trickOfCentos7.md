---
title: 'linux学习笔记：centos的一些小技巧'
date: 2018-06-05 15:37:25
categories: linux
tags: [linux, python, notes]
---


## 写在前面

整理网上找到的一切centos小技巧，随时补充。

## 安装python3

CentOS7系统自带的Python版本是Python2.7，使用如下方法升级到python3.

**实现步骤：**

### yum 安装

#### 第一步：安装IUS软件源

注：以root用户为例；如果不是root用户，加上sudo

```
yum install -y https://centos7.iuscommunity.org/ius-release.rpm
```

<!--more-->

#### 第二步：安装python3.6

```
//安装python3
yum install -y python36u

//创建软链接
ln -s /bin/python3.6 /bin/python3
```

#### 第三步：安装pip3

```
yum install -y python36u-pip

//创建软链接
ln -s /bin/pip3.6 /bin/pip3

//升级 pip
pip install --upgrade pip

```


## 安装thefuck

### 前提条件：

1. python (3.4+)

2. pip

3. python-dev

### 安装thefuck

```
pip install thefuck
```

#### TroubleShooting

```
//错误提示
error: command 'gcc' failed with exit status 

//解决方法：
yum -y install gcc gcc-c++ kernel-devel

yum -y install python-devel libxslt-devel libffi-devel openssl-devel

pip install cryptography

```

## 参考资料

* [CentOS 7 安装 Python3、pip3](https://ehlxr.me/2017/01/07/CentOS-7-%E5%AE%89%E8%A3%85-Python3%E3%80%81pip3/)

* [CentOS7安装Python3.6](https://www.yuzhi100.com/tutorial/centos/centos-anzhuang-python36)

