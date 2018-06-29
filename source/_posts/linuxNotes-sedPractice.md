---
title: 'Linux运维学习笔记：sed命令练习题目'
date: 2018-05-02 18:10:09
categories: linux
tags: [linux, regularExpression, notes] 
---

### 写在前面

这则笔记接着上期 [sed 笔记相关知识](https://www.steve-yuan.com/2018/04/20/week6-2-basisOfRegEx-part2/)，主要是一些练习题目。

### 打印某行到某行之间的内容

新建a.txt，内容如下，打印`[abcfd] `到 `[rty]` 

![](https://farm1.staticflickr.com/829/41845676901_a261b86c54_o.png)

**解题思路：**

主要考察`sed -n `， -n 选项表示打印指定行。

方法一：

```
sed -n '4,8p' a.txt
```

方法二：

```
sed -n '/\[abcfd\]/,/\[rty\]/p' a.txt
```

<!--more-->

### sed转换大小写

**解题思路：**

sed中，使用\u表示大写，\l表示小写

```
sed 's/\b[a-z]/\u&/g' filename    // 把每个单词的第一个小写字母变大写：

sed 's/[a-z]/\u&/g' filename    // 所有小写变大写

sed 's/[A-Z]/\l&/g' filename    //大写变小写
```

### sed在某一行最后添加一个数字

替换文档内容： `sed 'm,ns/keyword/替换文字/g' [filename]`

```
sed '/root/s/^/abc /g' /etc/passwd  //在/etc/passwd文件中在含有root的行的前面添加abc字符串，并在后面跟一个空格

sed '/^mail\>/,/^ftp\>/s/$/ abc/' /etc/passwd   //在以第一个mail开头的行到以一个ftp开头的行的后面添加abc，并在abc前面加一个空格
```

### 删除某行到最后一行 

**WARNING：（这个拓展完全没看懂）**定义一个标签a，匹配c，然后N把下一行加到模式空间里，匹配最后一行时，才退出标签循环，然后命令d，把这个模式空间里的内容全部清除。

```
[root@test200 ~]# cat test
a
b
c
d
e
f
[root@test200 ~]# sed '/c/{p;:a;N;$!ba;d}' test  
a
b
c
```

### 打印1到100行含某个字符串的行 

**解题思路：**

本题还是考察 sed -n 的用法。

```
sed  -n '1,100{/abc/p}'  1.txt
```
除了这种方法，借助管道符，还有两种方法也可以实现：

方法一：

```
sed -n '1,100p' 1.txt|sed -n '/abc/'p
```

方法二：

```
sed -n '1,100p' /etc/passwd | grep root
```

### sed打印匹配行与行号的方法

a.txt 文件内容

![](https://farm1.staticflickr.com/954/26989858887_dd847936d8_o.png)

- 打印匹配行的方法

```
[root@stevey ~]# sed -n '/name/p' c.txt
My name is steve.
```

- 打印行号的方法

```
[root@stevey ~]# sed -n -e '/name/=' c.txt
1
```

- 打印匹配行与行号的方法

```	
[root@stevey ~]# sed -n -e '/name/p' -e '/name/=' c.txt
My name is steve.
1
```




