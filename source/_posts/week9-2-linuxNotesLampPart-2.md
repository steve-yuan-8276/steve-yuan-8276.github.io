---
title: 'Linux运维学习笔记9-2：apache安装'
date: 2018-05-15 14:33:03
categories: linux
tags: [linux, apache, mysql, php,notes] 
---


### 写在前面

这则笔记继续整理lamp环境安装，主要是apache安装。

### apache 安装

Apache是世界使用排名第一的Web服务器软件。它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的Web服务器端软件之一。

**工作流：**

1. 下载rpm包

2. 安装依赖包 `apr` 、`apr-util`

3. 编译安装`httpd`


#### 第一步：下载并解压安装包

**安装包地址：**

2.4源码包：http://mirrors.cnnic.cn/apache/httpd/httpd-2.4.33.tar.gz 

apr安装包: http://mirrors.cnnic.cn/apache/apr/apr-1.6.3.tar.gz

apr-util安装包: http://mirrors.cnnic.cn/apache/apr/apr-util-1.6.1.tar.bz2 

<!--more-->


#### 第二步：安装依赖包：

- 安装apr

```
//解压apr源码包
tar -zvxf apr-1.6.3.tar.gz

//进入tar安装包
cd apr-1.6.3

//配置编辑参数
./configure --prefix=/usr/local/apr

//编译安装
make && make install

//校验是否安装成功
echo $?      //如果输出0，说明安装成功；否则，执行不成功

```

- 安装`expat-devel`  注：`expat-devel`是apr-util的依赖包

```
yum install -y expat-devel
```

- 安装apr-util

```
//解压源码包
tar -jvxf apr-util-1.6.1.tar.bz2

//进入安装包目录
cd apr-util-1.6.1

//配置编辑参数
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr

//编译安装
make && make install

//校验是否安装成功
echo $?      //如果输出0，说明安装成功；否则，执行不成功
```


#### 第三步：安装httpd

- 解压源码包

```
tar -zvxf httpd-2.4.33.tar.gz
```

- 进入安装文件夹

```
cd httpd-2.4.33
```

- 配置编译参数  注：`\`是转义字符，编译参数比较多的时候，可以分行写，用转义符号`\`分行。

```
[root@stevey httpd-2.4.33]# ./configure \
> --prefix=/usr/local/apache2.4 \
> --with-apr=/usr/local/apr \
> --with-apr-util=/usr/local/apr-util \
> --enable-so \
> --enable-mods-shared=most
```

**注：**trebleShooting

- 前一步结束之后，遇到错误提示：

![](https://farm1.staticflickr.com/832/42122689281_981cc9372b_o.png)

- 安装`pcre-devel`

```
yum install -y pcre-devel
```

- 重新编译安装

```
make && make install
```

**TroubleShooting**

这里曾经遇到错误提示如下：

![](https://farm1.staticflickr.com/972/40315985890_a961fd5666_o.png)

**[更新] ** 重新下载码包，操作了一遍，好了。所以，应该是前面参数写错了。

安装完成后，看看都加载了哪些模块

![](https://farm1.staticflickr.com/949/40524184190_51733223c3_o.png)

#### 第四步：配置启动httpd


```
//将链接apachectl 复制到系统启动目录下并命名为httpd
cp /usr/local/apache2.4/bin/apachectl /etc/init.d/httpd

//修改文件权限
chmod 755 /etc/init.d/httpd

//编辑启动脚本,更改监听端口（这部分内容不太清楚，后面再补充）
vim /etc/init.d/httpd
```

- 配置开机启动

```
//进入系统服务目录 /etc/init.d/
cd /etc/init.d/

//将mysqld 服务加入系统服务列表
chkconfig --add httpd      

//设置开机启动
chkconfig httpd on

//启动mysqld
service httpd start

//最后，再吸取之前的教训，做一个软链接解决环境变量配置错误的问题
ln -s /usr/local/apache2.4/bin/apachectl   /usr/bin

```


### 参考资料：

[Linux 下apache 服务器安装与配置教程(配置文件虚拟主机多站点详解)](http://www.osetc.com/archives/20369.html)

[在Linux下Apache配置完全教程](http://laolang.xtmm.cn/?p=14086)


