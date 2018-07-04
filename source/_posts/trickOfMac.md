---
title: 'mac笔记：关于mac的一些骚操作（一）'
date: 2018-04-09 13:19:26
categories: mac
tags: [mac, bash, notes] 
---

### 写在前面

这则笔记主要整理一些在网上看到的关于mac的使用技巧。

### `Mac OS` 添加 `alias`

#### 解决思路：

修改系统配置文件`/etc/profile` 或者 `~/.bash_profile`，这两个文件的区别在于：

* `/etc/profile` 适用于所有用户的全局配置脚本

* `~/.bash_profile` 适用于当前用户的启动配置文件

所以，修改`~/.bash_profile`比较安全点。

<!--more-->

#### 实现步骤：

##### 修改`~/.bash_profile`

```
vim ~/.bash_profile   //使用vim编辑

alias ll='ls -l --color=auto'    //添加这一行，使得` ll `命令生效

alias subl="'/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl'"   // 使用 subl 打开sublime text

alias ping='ping -c 4'     //ping 命令默认执行4次

```

之后，按esc， 输入 :wq 保存退出；

##### 执行以下命令，配置文件生效

```
source ~/.bath_profile 
```

### 终端安装 mac-vim

#### 安装brew

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

```

#### 安装VIM

```
brew install vim --with-lua --with-override-system-vi

```

#### 安装 GUI 版 mac vim

```
brew install macvim --with-lua --with-override-system-vim
```

安装完成后重启终端即可。

#### 更新mac vim

```
brew upgrade macvim
```

输入 `mvim` 可以从终端启动 GUI 版的 Vim。


### 终端自动补全

#### 基本思路

修改`～/.inputrc`配置文件（如果家目录下没有就新建一个）；配置完成后，按tab键，就能够自动补全。

#### 实现步骤

- 第一步：

```
vim ~/.inputrc    打开编辑文件，加入一下内容

set completion-ignore-case on
set show-all-if-ambiguous on
TAB: menu-complete
```

- 第二步：完成后，按 `esc` ，输入 `:wq` 保存退出。

- 第三步：重启shell


### mac自带的sed和linux表现不一致

mac 和 linux中都有sed命令，但是因为mac系统用的是原生的bsd系列，而一般的linux系统用的是gnu系统，所以两者在某些命令中还是有区别的。

在linux中，

```
sed "$line a\\(多一个\，用来防止转义) $value" $filename
```

在mac当中，

```
sed "$line a\       //在 \ 后要加一个空格

>$value             //需要添加新一行的内容）

>" $filename        //文件名
```

如果想要像linux那样，在mac中使用sed，就需要安装 `gnu-sed` ，具体如下：


```
brew install gnu-sed --with-default-names   //安装gnu-sed
vim ~/.zshrc    //编辑环境变量文件，加入下面这一行
export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH"
3.source ~/.zshrc   //或者新开窗口，让设置生效
```

**注：**如果希望直接更改源文件，在使用sed的时候加上 `-i` 选项。

### mac 录屏生成gif动图

#### 第一步：录制mov格式视频

- 打开mac自带 QuickTime Player

![](https://farm1.staticflickr.com/909/41145334074_51240e092f_o.png)

- 点击 `QuickTime Player -> file -> New Moive Recording`

- 点击录制按钮 > 选择录制区域 (选择模拟器) > 开始录制

- 录制完成 > 点击mac右上角停止按钮结束录制 > 保存格式为 mov

- 调整视频尺寸，如果是放在公众号里面的话，480P清晰度就足够了；不然尺寸太大，转成gif依然放不到公众号里面。如下图所示：

![](https://farm1.staticflickr.com/953/40963919175_e6b5e2d735_o.png)


#### 第二步：将mov视频转成gif

在APP Store 就可以下载GIFBrewery，免费的功能就够了。

![](https://farm1.staticflickr.com/956/27993879978_90c7258c4e_o.png)

安装后运行，GIFBrewery -> File  -> Open 那个mov视频

点击界面Create GIF -> Save成gif 文件即可。


### 命令行打开vscode

编辑 `～/.bash_profile`

```
vim ~/.bash_profile

加入以下内容：

code () { VSCODE_CWD="$PWD" open -n -b "com.microsoft.VSCode" --args $* ;}
```

保存退出并执行以下命令：

```
source ~/.bash_profile
```


