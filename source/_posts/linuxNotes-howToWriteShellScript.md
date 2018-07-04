---
title: 'Linux运维学习笔记6-4：怎么写shell脚本？'
date: 2018-04-27 10:47:20
categories: linux
tags: [linux, regularExpression, notes] 
---

### 写在前面

终于学到shell脚本了，但是视频课程竟然没有专门讲……好吧，既然你不讲，我就自己整理吧。

### 什么是shell脚本？

提到shell脚本，先要说到shell：shell是一种应用程序，一个操作界面，用户能够通过shell访问操作系统服务。linux中最常见的shell是bash。bash是交互式的，输入一条命令就能看到执行结果。

shell脚本（shell script），是一种为shell编写的脚本程序。我们可以把需要执行的命令写在脚本当中，系统通过读取脚本执行命令。

示例：

```
#!/bin/sh

## Auto create files

cd ~
mkdir shell_tut
cd shell_tut

for ((i=0; i<10; i++)); do
	touch test_$i.txt
done
```

如上所示，一个标准内容大概分成三个部分:

* 第一部分：`#!/bin/sh`，指定脚本解释器，这里是用/bin/sh做解释器的；专业术语叫`shebang`，是` #` （sharp） 和`！`（bang）两个字符的合起来的简写。 

* 第二部分：`## Auto create files` ，注释，主要是用来解释这个脚本是干嘛的。

* 第三部分：脚本本身，就是一行行的脚本代码。

<!--more-->

#### 关于脚本一些约定俗成的规则：

* 脚本通常都以 .sh 作为后缀名；

* 注释虽然不是必须的，但为了便于后期维护和管理，强烈建议每个脚本都要写注释；

* 脚本建议统一放在`/usr/local/sbin/` ，便于维护和管理

#### 脚本的执行方式

##### 第一种：脚本作为命令行参数调用

```
sh a.sh    // 执行a.sh；这里的a.sh 是命令行的参数
```

前面写成bash也可以，

```
bash a.sh
```

##### 第二种：赋予脚本执行权限 X ，将其变为可执行文件

```
chmod 755 a.sh  //此处也可以写成 chmod +x a.sh 注意+x中间没有空格
a.sh     //这里也可以使用文件的绝对路径
```

#### shell脚本的预备知识：`date` 命令

`date `命令 是在shell脚本当中最常用的命令，用来显示或者设置系统的日期和时间。

命令行格式： `date [参数] [+格式]`

参数 | 含义
---|---
-d|--date=字符串：显示指定字符串所描述的时间，而非当前时间
-f|--file=日期文件：类似--date，从日期文件中按行读入时间描述
-r|--reference=文件：显示文件指定文件的最后修改时间
-R| --rfc-2822：以RFC 2822格式输出日期和时间

格式| 含义
---|---
%y|以两位数字表示年份(00-99)
%Y|以四位数字表示年份
%m|月份(01-12)
%d|按月计的日期(例如：01)
%j|按年计的日期（0-366）
%c|当前locale 的日期和时间 (如：2005年3月3日 星期四 23:05:25)
%b|当前locale 的月名缩写 (如：一，代表一月)
%a|当前locale 的星期名缩写(例如： 日，代表星期日)
%A|当前locale 的星期名全称 (如：星期日)
%H|小时(00-23)
%M|分(00-59)
%S|大写S，表示秒(00-60)
%s|小写s，自UTC 时间 1970-01-01 00:00:00 以来所经过的秒数
%w|小写w，一星期中的第几日(0-6)，0 代表周一
%W|大写W，一年中的第几周，以周一为每星期第一天(00-53)

##### date 示例

###### 显示时间，可以设定显示格式 `date + [格式]`

- 显示时间

```
[root@stevey ~]# date
Fri Apr 27 14:09:18 HKT 2018
```

- `%A` : 星期几 (Sunday..Saturday)

```
[root@stevey ~]# date '+%A'
Friday
```

- `%j` ：一年中的第几天

```
[root@stevey ~]# date '+%j'
117
```

- `%W` ：一年中的第几周


```
[root@stevey ~]# date '+%W'
17
```

- `%`s : 从 1970 年 1 月 1 日 00:00:00 UTC 到目前为止的秒数

```
[root@stevey ~]# date '+%s'
1524809758
```

###### 加减操作 `-d` 参数


```
date +%Y%m%d                   //显示前天年月日
date -d "+1 day" +%Y%m%d       //显示前一天的日期
date -d "-1 day" +%Y%m%d       //显示后一天的日期
date -d "-1 month" +%Y%m%d     //显示上一月的日期
date -d "+1 month" +%Y%m%d     //显示下一月的日期
date -d "-1 year" +%Y%m%d      //显示前一年的日期
date -d "+1 year" +%Y%m%d      //显示下一年的日期
```

一天前的日期：

```
date date -d "-1 day" +%d
```

一小时前：

```
date date -d "-1 hour" +%H
```

一分钟前；

```
date date -d "-1 min" +%M
```

###### 设定时间：`-s` 选项，

**注：**设置当前时间，只有root权限才能设置，其他只能查看

```
date -s 20120523               //设置成20120523，这样会把具体时间设置成空00:00:00
date -s 01:01:01               //设置具体时间，不会对日期做更改
date -s "01:01:01 2012-05-23"  //这样可以设置全部时间
date -s "01:01:01 20120523"    //这样可以设置全部时间
date -s "2012-05-23 01:01:01"  //这样可以设置全部时间
date -s "20120523 01:01:01"    //这样可以设置全部时间
```

### shell脚本变量

变量格式： `var=value` var是变量名，value是变量内容；

**注：**

* 引用变量，需要加上`$`,即 `$var` ；

* 变量名var 如果是环境变量，约定俗成都用大写字母；如果是其他的变量，建议使用驼峰命名法，第一个单词用小写，从第二个单词开始，每个单词的首字母使用大写，例如 `myName`；

* 变量值可以是数字、字符串或者shell命令；如果是shell命令需要使用反引号（`tab` 上面那个键）；

示例:

![](https://farm1.staticflickr.com/975/39925240990_c2e22deb07_o.png)

**注：**引用变量时，要用双引号 "" ，否则 $var 会被当作字面量输出；

#### 数字运算

**注：**数学计算要用方括号 [] 括起来；同时，引用变量需要加 $

示例：

```
cat sum1.sh
#! /bin/bash     //shebang

## get sum of two numbers     //注释：求和

m=3
n=5

sum=$[$m+$n] 

echo 'The sum is $sum.'

```

#### 获取用户输入

**注：**相当于只定义变量，不赋值；value以参数的形式传入。

示例：

这里用到了rea命令，read 命令从键盘读取变量的值，通常用在shell脚本中与用户进行交互的场合。

命令行格式：`read [选项] [参数]` 参数指的就是变量名，可以跟多个变量名；

常用选项| 含义
---|---
-p|“提示信息” ：在等待键盘输入时，输出提示信息，方便用户输入
-t|秒数：指定等待时间，防止read命令一直等待用户输入
-s|输入的数据不显示在监视器上（实际上，数据是显示的，只是read命令将文本颜色设置成与背景相同的颜色）
-n|字符数：指定输入的字符数，只要用户输入指定的字符数，该read命令立即执行完毕

```
#! /bin/bash

## Using 'read' in shell scripts

read -p "please input a number: " x
read -p "Input another number: " y

sum=$[$x+$y]

echo "$x + $y = $sum"
```

#### 预设变量

shell变量没有数据类型的区别，变量中的值都是以字符串的形式保存的，按照使用环境大概有以下几种：

* 环境变量：用于保存操作系统运行时使用的环境参数

* 位置变量：Bash将传递给脚本的参数保存在位置变量中，以便于在脚本中引用这些参数

* 预定义变量：由系统保留和维护的一组特殊的变量，这些变量通常用于保存程序运行状态等

* 自定义变量：由用户自行定义的变量，可用于用户编写的脚本，多个命令间的值传递等


预设变量|含义
---|---
$0|保存当前程序或脚本的名称
$*|保存传递给脚本或进程的所有参数
$$|当前进程给脚本的PID号
$!|后台运行的最后一个进程的PID号
$?|用于返回上一条命令是否成功执行。如果这个变量的值为0，说明上一个命令正确执行；如果这个变量的值为非0，则说明上一个命令执行有错误
$#|用于保存脚本的参数个数

示例：

```
#! /bin/bash

## This script contains $0

echo "$1 $2 $0"    $0指的是脚本的名字

sum=$[$1+$2]

echo "$1 + $2 = $sum"

```

### shell脚本中的逻辑判断 `if` 和 `case`

#### if判断 

命令行格式

```
if 判断语句1；then     //逻辑判断是否符合条件1
    command A        //满足条件则执行command A
alif 判断语句2；then   //逻辑判断是否符合条件B
    command B        //满足条件则执行command B
else
    command C       //否则执行command C
fi
```

** 注: ** 跟JS 中的if判断如出一辙，同样也有 `else`、 `elif`；根据实际需要使用判断；`else`、 `elif` 可以省略。

##### 数字判断

数字判断语句有两种写法：

- 第一种写法：类似` ((var<n))` ，其中a为变量名，n为数字；比如`((a<16))`

逻辑判断中也支持 `&&` 且；  `||` 或

示例：

```
#! /bin/bash

## Using if

read -p "please input your age: " n

if((n<16));then
	echo "hello,kid"
elif ((n>=16)) && ((n<=60));then
	echo "Time to play!"
else
	echo "OK, You need rest."
fi
```

- 第二种写法：使用 `[]`

使用这种写法，逻辑关系需要改变一下写法

逻辑关系| `(())` 写法| `[]` 写法
---|---|---
大于|((m>n))|[m -gt n]
小于|((m<n))|[m -lt n]
小于等于|((m<=n))|[m -le n]
大于等于|((m>=n))|[m -ge n]
等于|((m=n))|[m -eq n]
不等于|((m!=n))|[m -ne n]

##### 字符判断

if 判断也支持判断文档的类型

命令行格式：

```
if [ 参数 filename ];then
    command
fi
```
也可以写成一行

```
if [ 参数 filename ];then command;fi
```

**注：**`[]` 里面要有空格。

这里的参数常见的有：

选项|含义
---|---
-e|exist ，判断文档或目录是否存在；  
-d|directory，判断是否是目录；
-f|file，判断是否是文件；
-r|read，判断是否有read权限；
-w|write，判断是否有write权限；
-x|execute，判断是否有execute权限；

示例

```
if [ -e a.txt ];then echo "This is a file";fi
```

#### case 判断

case 判断可以理解成if 判断的一种高级形式，跟JS当中的`switch `语句很像。

命令行格式：

```
case var in
value1）
    command A
    ;;
value2）
    command B
    ;;
value3）
    command C
    ;;
value4）
    command D
    ;;
*)
    command F
    ;;
esac
```

示例：

```
#! /bin/bash

## measure input number is even or not

read -p "please in number: " n

a=$[$n%2]     // $n%2 表示取余数

case $a in
1)
	echo "This is odd number."
        ;;
0)
	echo "This is a even number."
        ;;
*)
	echo "Ops, Something wrong."
esac
```

### shell 脚本中的函数

格式：

```
function 函数名（）
{
    command1
    command2
}
```

**注：**shell函数要写在最前面，不能写在中间或后面。就是说，函数需要先定义，再调用，否则就会出错。

示例：

```
#! /bin/bash

## In this script, we'll use function

read -p "input a number: " m
read -p "inout another number: " n
function sum()
{
	sum=$(($m+$n))
	echo "sum = $sum"
}

sum
```

### shell 脚本中的循环

跟python其他的高级语言类似，shell脚本也有for循环、while循环，以及break、continue。

#### for 循环

格式：

```
for 变量名 in 循环条件；do
    command
done
```

先来看一个最简单的for循环，列出1-5数字：

```
#! /bin/bash

## for loop practice

for i in `seq 1 5`; do
	echo $i
done
```

再增加一点难度，循环条件也可以是一个命令的执行结果

```
#! /bin/bash

## for loop practice

for file in `ls /root/`; do
	echo $file
done
```

#### while 循环

while 循环实质上就是for的一种变体，只要满足某个条件，循环就会一直执行。

格式：

```
while 循环条件；do
    command
done
```

先看一个例子：

```
#! /bin/bash

## while loop practice

a=10

while [ $a -ge 1 ]; do
	echo "$a"
	a=$[$a-1]
done
```

**注：如果我们想让这个循环一直执行下去，可以用冒号代替循环条件。
**

#### break语句：中断本次循环

适用场景：将判断和循环结合起来，符合某些条件时break中断循环。注：break只会影响本次循环，在有多重循环嵌套的情况，break只会影响本层级的循环，它的上层循环不受影响。

示例：

```
#! /bin/bash

## for loop with break practice

for a in `seq 1 10`
do
	echo "$a"
	if [ $a == 5 ]; then
		break
	fi
done
echo "It is over."
```

#### continue 语句：跳出本次循环继续执行

示例：

```
#! /bin/bash

## for loop with continue practice

for a in `seq 1 10`
do
	if [ $a == 5 ]; then
		continue
	else
		echo "$a"
	fi
done
echo "It is over."
```

在这个循环中，因为加入了continue语句，会直接跳过5，输出结果如下

![](https://farm1.staticflickr.com/828/41749855691_543603fce0_o.png)


#### exit语句：终止循环

跟break不同之处，在于使用exit语句，会直接退出整个shell脚本。

比如，我们把刚才脚本中continue改为 exit，如下所示

```
#! /bin/bash

## for loop with exit practice

for a in `seq 1 10`
do
	if [ $a == 5 ]; then
		continue
	else
		echo "$a"
	fi
done
echo "It is over."
```

执行脚本，结果如下：

```
[root@stevey sbin]# bash while1.sh
1
2
3
4       //执行到5，运行exit语句，直接退出shell脚本，后面的语句都不执行
```

