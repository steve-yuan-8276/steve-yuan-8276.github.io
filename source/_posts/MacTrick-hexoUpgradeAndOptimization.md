---
title: 'mac技巧：ssh代理及hexo升级同步'
date: 2018-06-29 11:10:30
categories: hexo
tags: [linux, hexo, github, notes]
---

## 写在前面

我的博客是采用的是`hexo + github` 方案，昨天在更新的时候，莫名其妙滴就卡在`hexo d`这一步了，怎么也过不去。原本五分钟应该搞定的事情，足足四十分钟也没成功，崩溃至极。

今天早上又冷静分析了一下，造成更新不成功，最大的可能无非两点：

1. 网络原因

2. 系统配置

于是，今天主要做两个修改，算是对个人博客的一次更新优化吧。

<!--more-->

## hexo 博客repo优化

hexo更新博客的流程大概是：

1. hexo g 将本地md文件生成静态网页；

2. gulp 优化静态网页；

3. hexo d 将优化后的静态网页推送到git.io

由于上述过程都是在本地电脑进行的，因此原始的md文件，也就保存在本地机器上，如果需要更换电脑的话，就需要自行拷贝文件，重新配置环境，这样以来就很不方便。

网上查了资料，有人说可以通过分支branch，master用来存放生成后的静态网页，source用来存储源文件。听起来确实是个很好的办法，但是还是那句话，本人是菜鸟，又怕搞不好把博客挂掉了，所以迟迟没有动手。

今天又在网上找资料，查到一个很好的方法，而且使用简单，连我这样的菜鸟也能很容易配置成功。

该方法借助`hexo-deployer-git`插件完成，GitHub地址：https://github.com/hexojs/hexo-deployer-git

实现步骤：

### 第一步：安装插件

```
npm install hexo-deployer-git --save
```

### 第二步：登陆新建branch分支

最便捷的方法是登陆GitHub网站，进入github.io的repo，系统默认有一个master的分支，再新建一个source就OK，当然此处的名字随你喜欢，取什么都行。

也可以在本地命令行操作，先cd进入`github.io`目录

```
//新建一个名为source的branch
git branch source

//进入该分支
git checkout source

//上述两步还可以合并为一行
git checkout -b source
```


### 第三步：修改hexo配置文件

注意：修改的是hexo文件夹根目录下的`_config.yml`，只需修改其中deploy那部分，示例代码如下：


```
deploy:
  - type: git
    //这个repo地址是github.io的默认地址，如果你已经配置过了，这里是不需要更改的。
    repo: git@github.com:<username>/<username>.github.io.git     
    branch: master
  - type: git
    //跟上面的repo地址保持一致
    repo: git@github.com:<username>/<username>.github.io.git
    //这里写的是需要保持的branch分支名
    branch: source
    //这个是需要同步的文件夹路径，/表示从hexo博客根目录下的所有文件都同步
    extend_dirs: /
    ignore_hidden: false
    ignore_pattern:
        public: .
```

基本上就是这样，设置完成保存，就可以了。这个方法最大的好处，还是不会增加任何的额外步骤，仍然是老三样，非常简单。

## 配置iterm2走代理 

我的mac翻墙使用的是surge，终端用的是iterm2+zsh。虽然浏览器可以打开github，但是在iterm2终端却ping不通，查了一下，原来浏览器走的是http代理，但是ping用的是ICMP协议，两者并不一样。

以下是按照网上找到的设置方案，让iterm2走代理。

设置步骤：

### 第一：查看`surge-->通用-->代理`

![](https://farm2.staticflickr.com/1830/43032668192_e0751c3ce9_o.png)

### 第二：修改zsh配置文件

```
vi ~/.zshrc

//添加两行

alias goproxy='export http_proxy=http://127.0.0.1:6152 https_proxy=http://127.0.0.1:6152'

alias disproxy='unset http_proxy https_proxy'

//保存退出后执行以下命令生效
source ~/.zshrc
```

### 第三步：使用

可以看到上面其实就是设置了两个alias命令，使用的时候，输入goproxy 启用代理；输入disproxy退出代理。







