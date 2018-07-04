---
title: 'mac笔记：有趣的命令行工具'
date: 2018-05-11 17:43:00
categories: mac
tags: [mac, bash, notes] 
---


### 写在前面

整理一些有趣的命令行工具，主要来源都是github

### htop 工具： top 替代

git地址：https://github.com/hishamhm/htop 

#### 安装：

```
brew install htop
```

#### 使用：

终端输入 htop：

![](https://farm1.staticflickr.com/972/42101390811_af946583a2_o.png)

<!--more-->

快捷键操作| 说明
---|---
h, ? F1  |查看htop使用说明
S，F2 |htop 设定
/，F3|搜索进程
\，F4|增量进程过滤器
T，F5|显示树形结构
<, >，F6|选择排序方式[
F7|可减少nice值可以提高对应进程的优先级
F8|可增加nice值，降低对应进程的优先级
K，F9|可对进程传递信号
q，F10|结束htop
u|只显示一个给定的用户的过程
U|取消标记所有的进程
H|显示或隐藏用户线程
K|显示或隐藏内核线程
F|跟踪进程
P|按CPU 使用排序
M|按内存使用排序
T|按Time+ 使用排序
l|显示进程打开的文件
I|倒转排序顺序
s|选择某进程，按s:用strace追踪进程的系统调用

### `gifgen` ：gif文件生成工具

git地址：https://github.com/lukechilds/gifgen

#### 安装：

```
brew install lukechilds/tap/gifgen

```

#### 使用：

命令行格式：

```
gifgen [options] [input]

```

常用参数|说明
---|---
-o   |Output file [input.gif]
-f   |Frames per second [10]
-s   |Optimize for static background
-v   |Display verbose output from ffmpeg


**示例：**

```
//压缩视频文件为gif
gifgen video.mp4

//-o选项指定输出文件名称
gifgen -o demo.gif SCM_1457.mp4

//优化北京帧率
gifgen -sf 15 screencap.mov

```

### thefuck 自动更正命令行错误

git地址：https://github.com/nvbn/thefuck

#### 安装：

```
brew install thefuck
```

修改环境变量

```
vim ~/.bashrc    //编辑bash配置文件，加入以下内容：

eval $(thefuck --alias)    //之后保存退出

source ~/.bashrc    重新载入配置文件
```

#### 使用：

直接`fuck`就OK了，语法简单到令人发指。

示例：

```
steve:steveBlog steveyuan$ ls a
ls: a: No such file or directory
steve:steveBlog steveyuan$ fuck
ls [enter/↑/↓/ctrl+c]
_config.yml		node_modules		scaffolds
a.txt			package-lock.json	source
db.json			package.json		themes
gulpfile.js		public

```



