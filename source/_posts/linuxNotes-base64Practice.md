---
title: 'linux学习笔记：从一则招聘启示学base64编码'
date: 2018-05-22 10:39:02
categories: linux
tags: [linux, base64, notes]
---

## 写在前面

今天，在微信群里看到了一则有趣的招聘启示：

![](https://farm1.staticflickr.com/948/27398471567_5f65289517_o.png)

重点是红框括起来的部分，`5pyJIHJtIC1yZiAvIOaVsOaNruaBouWkjee7j+mqjA==`这是什么鬼呢？当然，很快就有网友认出来这是base64编码，并给出了正确答案。

那么，base64又是什么呢？网上搜了一下，还挺有意思的，于是就整理出了这篇学习笔记。

<!--more-->

## `base64` 是神马东东？

base64，简单的说，就是使用64个字符来对任意数据进行编码的一种方法，本质上是一种将二进制数据转成文本数据的方案。

二进制文件包含很多无法显示和打印的字符，在进行网络信息传递时，这些二进制资源往往需要转换为Base64编码进行传输，以提高传输效率。

要理解`base64`编码原理，先看这张表：

![](https://farm1.staticflickr.com/980/41548088474_9ee076bf6c_o.jpg)

先将其转换成二进制形式，然后每连续6比特（2的6次方=64）计算其十进制值，根据该值在上面的索引表中找到对应的字符，最终得到一个文本字符串。

以文本”M”转换为Base64的过程为例，”M”先转化为二进位制形式，转换后的ASCII不足以填充够24位二进制字符串，全部用”0”补充；之后将每3个字节（24位）转换为4个字符，得到的索引值即分别为”T”,”Q”,”=”,”=”,最终的Base64编码结果为”TQ==”。

![](https://farm1.staticflickr.com/902/28396180948_dae69fa569_o.jpg)

这里的等号就是为了凑字数，在后面补上“=”字符，使编码后的字符数为4的倍数。这里的等号在传输的可以被省略，解码时如果发现Base64编码字符串长度不能被4整除，则先补充 “=” 字符，再解码即可。

再比如，hello转化为base64

![](https://farm1.staticflickr.com/960/41548220424_7c54c25dc1_o.jpg)

## `base64`解码方法


### 方法一：mac命令行转码

mac命令行自带转码工具

![](https://farm1.staticflickr.com/882/40462674230_bafb932a5b_o.png)

- base64编码：

```
steve:steveBlog steveyuan$ echo 'hello' | base64
aGVsbG8K

```

- base64解码：


```
steve:steveBlog steveyuan$ echo 'aGVsbG8K' | base64 -D
hello

```

### 方法二：利用python 自带的base64转码

- base64编码：

```
steve:steveBlog steveyuan$ python -m base64 <<< hello
aGVsbG8K
```
- base64解码：

```
steve:steveBlog steveyuan$ python -m base64 -d <<< aGVsbG8K
hello
```

### 方法三：利用openssl对base64进行编码和解码


- base64编码：

```
steve:steveBlog steveyuan$ openssl enc -base64 <<< hello
aGVsbG8K
```

- base64解码：

```
steve:steveBlog steveyuan$ openssl enc -base64 -d <<< aGVsbG8K
hello
```

### 方法四：使用coreutils包装中的base64程序

- 安装coreutils包(centos7系统下）

```
yum install -y coreutils
```

- base64编码：

```
[root@steve ~]# echo 'hello' | base64
aGVsbG8K
```

- base64解码：

```
[root@steve ~]# echo aGVsbG8K | base64 --decode
hello
```

那么，说了这么多，招聘启事当中那句话究竟是什么意思呢？用刚刚学到的方法看下：

```
[root@steve ~]# echo 5pyJIHJtIC1yZiAvIOaVsOaNruaBouWkjee7j+mqjA== | base64 --decode
有 rm -rf / 数据恢复经验
```

不过，话说回来`rm -rf /`之后，真的有办法进行数据恢复嘛？这又是一个新的问题了。

## 参考资料：

* [Base64编码原理与应用](http://blog.xiayf.cn/2016/01/24/base64-encoding/)

* [Base64编码简介](https://www.cnblogs.com/antineutrino/p/3756106.html)

* [Base64编码规则](https://geocld.github.io/2016/05/27/base64/)

* [为什么要使用base64编码，有哪些情景需求？](https://www.zhihu.com/question/36306744)

* [让你完全理解base64是怎么回事](https://segmentfault.com/a/1190000004533485)

* [mac下base64命令](https://blog.csdn.net/fjh658/article/details/8875840)

* [我怎样才能从命令行解码base64字符串？](http://bbs.bugcode.cn/t/7855)



