---
title: 树莓派笔记：ssh连接、安装源更新、安装vim
date: 2018-05-15 22:36:45
categories: linux
tags: [linux, raspberryPi, notes] 
---

### 写在前面

为了不让买了一年的树莓派继续落灰，同时也检验一下自己的linux学习成果，决定来折腾折腾树莓派，先从最简单的开始系统配置开始。

### 替换国内源

树莓派使用`apt-get`来管理安装包，默认的安装源镜像在国外，因此要做的第一件事情，就是切换到国内的安装源。

#### 目前国内比较常用的源地址：

- 阿里云镜像源：

```
deb http://mirrors.aliyun.com/raspbian/raspbian/ jessie main non-free contrib rpi
deb-src http://mirrors.aliyun.com/raspbian/raspbian/ jessie main non-free contrib rpi
```

- 中科大镜像源：

```
deb https://mirrors.ustc.edu.cn/archive.raspberrypi.org/ jessie main ui
```

- 清华镜像源：

```
deb https://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ jessie main ui
```

安装源的配置文件为`/etc/apt/sources.list` ，是更新前最好先备份一下；万一更新失败，还可以恢复

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak   #备份为 sources.list.bak
```

下面，正式开始更新源：

#### 第一步：编辑 `/etc/apt/sources.list` 文件

```
sudo nano /etc/apt/sources.list    //nano是raspberryPi默认的文本编辑器

//进入编辑界面，注释掉原有内容，粘贴如下内容：

deb http://mirrors.aliyun.com/raspbian/raspbian/ jessie main non-free contrib rpi
deb-src http://mirrors.aliyun.com/raspbian/raspbian/ jessie main non-free contrib rpi

```

#### 第二步： 更新软件列表

```
sudo apt-get update
```

#### 第三步：更新已安装软件

注：这一步会持续很长很长很长很长时间，Be patient！

```
sudo apt-get upgrade -y

```

<!--more-->

### 安装使用vim

raspberryPi的默认文本编辑器是nano，使用起来各种不习惯，所以想办法替换为vim。

#### 安装vim

##### 第一步：删除系统自带的vim-common

```
sudo apt-get remove vim-common
```

##### 第二步：更新源

```
sudo apt-get update
```

##### 第三步：安装vim

```
sudo apt-get install vim
```

安装完毕，执行以下命令查看vim版本

```
vim --version
```

#### 配置vim

vim配置文件保存在`/etc/vim/vimrc`或者`～/.vimrc` ，编辑这个配置文件：

```
sudo vim ～/.vimrc

加入以下内容：

//显示行号
set nu

//设置在命令行界面最下面显示当前模式等
set showmode

//显示标尺，在右下角显示光标位置
set ruler

//自动对齐
set autoindent

//语法高亮
syntax on

//设置缩进
set tabstop=4
set shiftwidth=4
set expandtab
set smarttab

//高亮查找匹配
set hlsearch

//用浅色高亮当前行
autocmd InsertEnter * se cul

//启动时隐去援助提示
set shortmess=atI

```

按`esc`键，再输入`:wq` 保存退出；

更新配置；

```
source ～/.vimrc
```

#### 将vim设为默认文本编辑器

输入命令：

```
update-alternatives --config editor

```
选择`/usr/vim/vim.basic`


### 参考资料：

* [树莓派 Raspberry Pi 更换阿里云更新源方法](http://bbs.shumeipaiba.com/thread-5-1-1.html)

* [树莓派—raspbian软件源（全）](https://www.jianshu.com/p/67b9e6ebf8a0)

* [树莓派 Raspberry Pi 更换阿里云更新源方法](https://www.jianshu.com/p/abb1d797d1f3)

* [raspberrypi安装vim并配置](https://blog.csdn.net/bestBT/article/details/71307800)


