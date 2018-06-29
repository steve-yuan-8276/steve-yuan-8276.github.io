---
title: 'mac笔记：关于mac的一些骚操作（二）'
date: 2018-05-10 21:47:33
categories: mac
tags: [mac, bash, notes] 
---

### 写在前面

继续整理有关mac使用的技巧，先更新三条小技巧，以后在陆续更新。

### 下片神器 `you-get`

#### 安装：

##### 安装依赖：

[python 3](https://www.python.org/downloads/)

[FFmpeg](https://www.ffmpeg.org/) 

**注：**`FFmpeg`也可以通过brew安装 `brew install ffmpeg`

##### 安装`you-get`：

- 通过brew安装

```
brew install you-get
```

- 通过pip3 安装：

```
pip3 install you-get
```

#### 升级`you-get`：

- 如果是通过brew 安装的，则运行：

```
brew upgrade you-get
```

- 如果是通过pip3安装的，则运行：

```
pip3 install --upgrade you-get

```

<!--more-->

#### 使用`youget`

##### 第一步：找到需要下载的资源，复制网页地址，查看画质和格式信息：

命令行格式：`you-get -i [url]` or `you-get --info [url]`

```
steve:~ steveyuan$ you-get -i https://www.youtube.com/watch?v=iaUze8A0844
site:                YouTube
title:               papi酱 - 如果这些职业和医生混搭…【papi酱突然更新的放送】
streams:             # Available quality and codecs
    [ DASH ] ____________________________________
    - itag:          248
      container:     webm
      quality:       1920x1080
      size:          92.3 MiB (96795814 bytes)
    # download-with: you-get --itag=248 [URL]

    - itag:          137
      container:     mp4
      quality:       1920x1080
      size:          68.9 MiB (72264226 bytes)
    # download-with: you-get --itag=137 [URL]

    - itag:          247
      container:     webm
      quality:       1280x720
      size:          50.2 MiB (52596250 bytes)
    # download-with: you-get --itag=247 [URL]
     [ DEFAULT ] _________________________________
    - itag:          22
      container:     mp4
      quality:       hd720
      size:          39.6 MiB (41558299 bytes)
    # download-with: you-get --itag=22 [URL]
```

注：带有`default` 字样的是网页当中默认的格式

##### 第二步：下载视频

命令行格式：`you-get [参数] [url]`

注：这里的参数，对应的就是上面info信息的这个部分，也就是 `--itag=22`，默认下载到当前目录下。如下图所示：

![](https://farm1.staticflickr.com/826/41118014595_7e78e1e4a1_o.png)

-o [filePath] 小写-o选项，可以指定下载文件路径(这里要写绝对路径)；`-O [fileName]`,大写O可以为下载文件重命名；

```
you-get --itag=22 -o ~/documents/ -O papi-1.mp4 https://www.youtube.com/watch?v=iaUze8A0844
```

**注：**

* 按照官方文档说法，这个参数可以省略，表示下载default的视频链接；

* 如果下载了一半又改主意不想下载了，按 `ctrl + c` 中止；

#### 分享几个小技巧

##### 搜索并下载视频

有时候，我们并不知道下载资源的具体网址，只知道名字而已。这时候输入 `you-get [keyword]` , 此时you-get 会通过google进行搜索，并下载最相关的视频`most relevant video`。

##### 通过代理线路下载

```
you-get -x [代理ip：端口号] [url]

或者，两个效果是一样的

you-get --http-proxy [代理ip：端口号] [url]
```

##### 直接播放视频链接


```
you-get -p [播放器名] [url]

或者

you-get --player [播放器名] [url]
```

此处也可以指定浏览器打开，免广告

```
you-get -p chromium [url]    //chromium是浏览器名
```

**注：**更多内容参见[官方说明文档](https://github.com/soimort/you-get#supported-sites)


### 在`mac`下抓包

**前提：**获取网络接口的 BSD 设备名称

打开`about this mac` --> `hardware report` --> `network` ,在图示中，Wi-Fi 的 BSD 设备名称为 en0。

![](https://farm1.staticflickr.com/981/42033886471_f047742c33_o.png)

- 第一步：打开teminal,输入以下命令，但要将“BSDname”替换为“系统信息”中的 BSD 设备名称（如 en0、en1 或 ppp0）：

```
sudo tcpdump -i BSDname -s 0 -B 524288 -w ~/Desktop/DumpFile01.pcap             //要求输入管理员密码
```

- 第二步：按 Ctrl+C，停止抓包，抓包数据保存在桌面为`“DumpFile01.pcap”`的文件中。 

- 第三步：查看抓包的内容，请在终端中使用以下命令：

```
tcpdump -s 0 -n -e -x -vvv -r ~/Desktop/DumpFile01.pcap
```

### vim配置

Vim 配置文件在`/etc/vimrc`(全局配置)、 `～/.vimrc`（用户配置），一般直接编辑`～/.vimrc`即可。

```
vim ~/.vimrc      //添加以下内容

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

### iterm2+zsh配置

mac系统默认使用的是bash，可以该用zsh来实现更强大的功能，具体的操作步骤如下：

#### 第一步：安装zsh

- 安装zsh

```
//查看系统自带的shell
steve:~ steveyuan$ cat /etc/shells
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh

//查看zsh的版本
steve:~ steveyuan$ zsh --version
zsh 5.3 (x86_64-apple-darwin17.0)

```

mac自带了zsh，因此就不用重新安装了。

- 安装zsh-completions

```
brew install zsh-completions
```

- 切换默认shell

```
chsh -s /bin/zsh
```

#### 第二步：安装oh-my-zsh

github地址：https://github.com/robbyrussell/oh-my-zsh

使用curl安装

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

使用wget安装（推荐）

```
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

[To be continue]

