---
title: 'linux笔记：DNS基础及其查询'
date: 2018-06-14 10:45:41
categories: linux
tags: [linux, mysql, notes]
---

## 写在前面

查漏补缺，补充学习前一阶段课程涉及到的一些有用的DNS基础知识。

## 什么是DNS

我们所熟知的互联网通讯是基于TCP/IP协议的，而TCP/IP是基于ip地址的，也就是说，计算机在网络上通讯依赖的都是ip地址。但是，ip地址本身并不方便记忆，我们通常访问网址也不是通过ip（比如20.181.111.188），而是域名（比如www.baidu.com）。因此，就催生了一套域名转化机制，负责把域名转化成ip地址，这就是DNS。

DNS( Domain Name System)是“域名系统”的英文缩写，具体负责将主机名和域名转换为IP地址。它的工作原理通过归递查询，逐级返回访问对象的真正地址。

### 域名层级格式如下：

```
主机名.次级域名.顶级域名.根域名
```
比如www.baidu.com,真正的域名是www.baidu.com.root，其中，

标识|层级
---|---
root|根域名
com|顶级域名
baidu|次级域名
www|主机名

<!--more-->

**注：**

* 各层级域名之间，用 `.` 隔开；

* 因为大家的跟域名都一样，所以在实践中，`.root` 通常被省略了。

### 域名查询原理：

1. 从"根域名服务器"查到"顶级域名服务器"的NS记录和A记录（IP地址）；

2. 从"顶级域名服务器"查到"次级域名服务器"的NS记录和A记录（IP地址）；

3. 从"次级域名服务器"查出"主机名"的IP地址；

目前，全世界有13组根域名服务器，具体可以参见[wiki介绍](https://zh.wikipedia.org/wiki/%E6%A0%B9%E7%B6%B2%E5%9F%9F%E5%90%8D%E7%A8%B1%E4%BC%BA%E6%9C%8D%E5%99%A8)。关于域名解析的详细解释，可以查看这个网页：https://www.verisign.com/zh_CN/website-presence/online/how-dns-works/index.xhtml

可以看到，这些服务器都不在国内，如果每一次访问都要这样查询，那速度肯定比蜗牛还慢。所以，实际是运行中是这样的：

1. `浏览器缓存`　　当用户通过浏览器访问某域名时，浏览器首先会在自己的缓存中查找是否有该域名对应的IP地址（若曾经访问过该域名且没有清空缓存便存在）；　　

2. `系统缓存`　　当浏览器缓存中无域名对应IP则会自动检查用户计算机系统Hosts文件DNS缓存是否有该域名对应IP；　

3. `路由器缓存`　　当浏览器及系统缓存中均无域名对应IP则进入路由器缓存中检查，以上三步均为客服端的DNS缓存；　　

4. `ISP（互联网服务提供商）DNS缓存`　　当在用户客服端查找不到域名对应IP地址，则将进入ISP DNS缓存中进行查询。比如你用的是电信的网络，则会进入电信的DNS缓存服务器中进行查找；　　

5. `根域名服务器`　　当以上均未完成，则进入根服务器进行查询。全球仅有13台根域名服务器，1个主根域名服务器，其余12为辅根域名服务器。根域名收到请求后会查看区域文件记录，若无则将其管辖范围内顶级域名（如.com）服务器IP告诉本地DNS服务器；　　

6. `顶级域名服务器`　　顶级域名服务器收到请求后查看区域文件记录，若无则将其管辖范围内主域名服务器的IP地址告诉本地DNS服务器；　　

7. `主域名服务器`　　主域名服务器接受到请求后查询自己的缓存，如果没有则进入下一级域名服务器进行查找，并重复该步骤直至找到正确纪录；　　

8. `保存结果至缓存`　　本地域名服务器把返回的结果保存到缓存，以备下一次使用，同时将该结果反馈给客户端，客户端通过这个IP地址与web服务器建立链接。

### DNS的记录类型

域名与IP之间的对应关系，称为"记录"`record`。根据使用场景，"记录"可以分成不同的类型`type`，常见的DNS记录类型如下：

* `A：`地址记录（Address），返回域名指向的IP地址；

* `CNAME：`规范名称记录（Canonical Name），返回另一个域名，即当前查询的域名是另一个域名的跳转；

* `NS：`域名服务器记录（Name Server），返回保存下一级域名信息的服务器地址。该记录只能设置为域名，不能设置为IP地址；

* `MX：`邮件记录（Mail eXchange），返回接收电子邮件的服务器地址；

* `PTR：`逆向查询记录（Pointer Record），只用于从IP地址查询域名；

## DNS查询工具

linux下，常用的DNS查询工具有dig 、nslookup，用法如下：

### dig命令

#### 安装

- centos安装

```
yum install bind-utils
```

**注：**mac本身支持dig，不需额外安装。

#### 语法

命令行格式：

```
dig @[DnsServer] [options] [WebSiteName] [querytype]
```

- dnsserver, 指的是dns的地址；

>如果指定dnsserver，那么dig会首先通过默认的上连DNS服务器去查询对应的IP地址，然后再以设置的dnsserver为上连DNS服务器；

>如果未指定dnsserver，那么dig就会依次使用/etc/resolv.conf里的地址作为上连DNS服务器；

- WebSiteName，指的是需要查询的域名；

- querytype,查询的类型，写出 -t [A/AAAA/PTR/MX/ANY]等，默认查询的是A记录，也就是ip地址。

- options，指的是各种选项；

#### DNS 命令输出格式

![](https://farm2.staticflickr.com/1758/42740366832_9f7feb9a61_o.png)

**说明：**

* `status: `NOERROR 表示查询没有什么错误，Query time 表示查询完成时间；

* `SERVER:` 10.202.72.116#53(10.202.72.116)，表示本地 DNS 服务器地址和端口号；

* `QUESTION SECTION` 表示需要查询的内容，这里需要查询域名的 A 记录；

* `ANSWER SECTION` 表示查询结果，返回 A 记录的 IP 地址。600 表示本次查询缓存时间，在 600 秒本地 DNS 服务器可以直接从缓存返回结果；

* `AUTHORITY SECTION` 表示从那台 DNS 服务器获取到具体的 A 记录信息，由它来维护查询域名的信息;

* `ADDITIONAL SECTION` 表示 NS 服务器对应的 IP 地址，这些 IP 地址对应的服务器安装了 BIND 软件;

#### options 常见用法：

- 不加任何插件，显示ipv4环境下13个根服务器的信息

```
[root@stevey test]# dig

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>>
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23085
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;.				IN	NS

;; ANSWER SECTION:
.			112652	IN	NS	a.root-servers.net.
.			112652	IN	NS	b.root-servers.net.
.			112652	IN	NS	c.root-servers.net.
.			112652	IN	NS	d.root-servers.net.
.			112652	IN	NS	e.root-servers.net.
.			112652	IN	NS	f.root-servers.net.
.			112652	IN	NS	g.root-servers.net.
.			112652	IN	NS	h.root-servers.net.
.			112652	IN	NS	i.root-servers.net.
.			112652	IN	NS	j.root-servers.net.
.			112652	IN	NS	k.root-servers.net.
.			112652	IN	NS	l.root-servers.net.
.			112652	IN	NS	m.root-servers.net.

;; Query time: 64 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Thu Jun 14 15:02:20 HKT 2018
;; MSG SIZE  rcvd: 239
```

- `-f选项`：从指定文本中逐行读取内容进行查询

```
[root@stevey test]# cat a.txt
www.baidu.com
www.163.com
[root@stevey test]# dig -f a.txt @8.8.8.8

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> www.baidu.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5241
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.baidu.com.			IN	A

;; ANSWER SECTION:
www.baidu.com.		1176	IN	CNAME	www.a.shifen.com.
www.a.shifen.com.	123	IN	A	220.181.111.188
www.a.shifen.com.	123	IN	A	220.181.112.244

;; Query time: 15 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Thu Jun 14 14:36:46 HKT 2018
;; MSG SIZE  rcvd: 90

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> www.163.com
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63874
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.163.com.			IN	A

;; ANSWER SECTION:
www.163.com.		242	IN	CNAME	www.163.com.lxdns.com.
www.163.com.lxdns.com.	439	IN	CNAME	163.xdwscache.ourglb0.com.
163.xdwscache.ourglb0.com. 22	IN	A	103.254.188.230
163.xdwscache.ourglb0.com. 22	IN	A	106.120.178.41

;; Query time: 18 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Thu Jun 14 14:36:46 HKT 2018
;; MSG SIZE  rcvd: 129
```

- `-t选项` 指定查询的记录类型

查询A记录

![](https://farm1.staticflickr.com/887/42789873201_cf10ec89eb_o.png)

查询CNAME记录

![](https://farm2.staticflickr.com/1743/42740059522_b891f5115f_o.png)

- `-4/6选项` 指定传输协议，-4指的是ipv4；-6指的是ipv6

```
[root@stevey test]# dig @8.8.8.8 www.google.com -4

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> @8.8.8.8 www.google.com -4
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 65095
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.google.com.			IN	A

;; ANSWER SECTION:
www.google.com.		63	IN	A	31.13.66.1
```

- `-x选项` 逆向查询，意思是查询IP地址对应的域名

```
[root@stevey test]# dig -x 193.0.14.129

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> -x 193.0.14.129
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55475
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;129.14.0.193.in-addr.arpa.	IN	PTR

;; ANSWER SECTION:
129.14.0.193.in-addr.arpa. 1	IN	PTR	k.root-servers.net.

;; Query time: 28 msec
;; SERVER: 129.29.29.29#53(129.29.29.29)
;; WHEN: Thu Jun 14 14:54:29 HKT 2018
;; MSG SIZE  rcvd: 75


;; Query time: 13 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Thu Jun 14 14:51:17 HKT 2018
;; MSG SIZE  rcvd: 48

```

- `+trace 选项` 从根域查询一直跟踪直到查询到最终结果，并将整个过程信息输出出来；

```
[root@stevey test]# dig @8.8.8.8 +trace www.baidu.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> @8.8.8.8 +trace www.baidu.com
; (1 server found)
;; global options: +cmd
.			99778	IN	NS	e.root-servers.net.
.			99778	IN	NS	h.root-servers.net.
.			99778	IN	NS	l.root-servers.net.
.			99778	IN	NS	i.root-servers.net.
.			99778	IN	NS	a.root-servers.net.
.			99778	IN	NS	d.root-servers.net.
.			99778	IN	NS	c.root-servers.net.
.			99778	IN	NS	b.root-servers.net.
.			99778	IN	NS	j.root-servers.net.
.			99778	IN	NS	k.root-servers.net.
.			99778	IN	NS	g.root-servers.net.
.			99778	IN	NS	m.root-servers.net.
.			99778	IN	NS	f.root-servers.net.
.			99778	IN	RRSIG	NS 8 0 518400 20180624050000 20180611040000 39570 . qrPNK9BW+EcNETVHvYGuf3cIP7PmfWb6zfikmgYORzMwsn5urzYu6ZPR x/w5jACtOe9ROfqsj1+rxdHr22nWGMz2ZL1ZzfM+ptu1jOK0cunVPwUR Wd4aFk3Dfe9D5uckdNf8x2NuGfpmDa5CXlMiEGv7OF42MIjR5NRZqarZ B3Gx8cWef+PyouWL7rkpCvBHmywisdOE2QSUK8rkBMNyStI5t6Q5yUTj gHujdswu0iYVZfSrKN6p38taWwoIA2Ljf8N2rXsE5T6e+3lK0tYATL0w 1EOpkSVZ90UsLh7/HPE/l6wO91zA5X1uF33LHseLb5U2M4ScNHOQpx0V /OTqaw==
;; Received 525 bytes from 8.8.8.8#53(8.8.8.8) in 263 ms

com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			86400	IN	DS	30909 8 2 E2D3C916F6DEEAC73294E8268FB5885044A833FC5459588F4A9184CF C41A5766
com.			86400	IN	RRSIG	DS 8 1 86400 20180627050000 20180614040000 39570 . Tg+3ggtwhbBFoGx7mrHTZfHSoxhh6kRbtTX6YXnRZkIV25s15IPLIVLq pUEUVNgmuOmccmuN5vhWqJTgQpHbKA6f4X2ABFSbigBL3zS8MioiVMvv FNlxuATGfqYxqdize3x/fNXiQAy5J+RetXFDoHFATh+6v+Pl/FSyoKs2 pNm3WSU12PD+5wqgmMeoAcbAi1MT7oPcWoGmHS8KEI2WJp+2QjeKwnzl OqVzPdoqvMm/UDRgxcrzww58oO7jPGLRVHwyEgS6oSN8oxTear6AC3Io Oo8/PkQBBStX2fTY1jCKbY7byq/WA5v0QTQuqIWP4mrfSDKAdPGPtVS2 /pF9pg==
;; Received 1173 bytes from 192.58.128.30#53(j.root-servers.net) in 386 ms

baidu.com.		172800	IN	NS	dns.baidu.com.
baidu.com.		172800	IN	NS	ns2.baidu.com.
baidu.com.		172800	IN	NS	ns3.baidu.com.
baidu.com.		172800	IN	NS	ns4.baidu.com.
baidu.com.		172800	IN	NS	ns7.baidu.com.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN NSEC3 1 1 0 - CK0Q1GIN43N1ARRC9OSM6QPQR81H5M9A NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 86400 IN RRSIG NSEC3 8 2 86400 20180621044630 20180614033630 36707 com. VCZSbjvAmAGA/Oa2GZecQbcvXGrO4qRqA3o1vhDfJJtopOSROtQrGnDE RnesT31RcG0FUanP2C632ACP0T2uywP5o7UJE4JNOrw0wZm4TJsfmXHb 1rksLWqZOuNi8SnOLIeEos5vL66xgpNb1OJtdhfEu4TlVEgnzNdu6Rqq IBk=
HPVV2B5N85O7HJJRB7690IB5UVF9O9UA.com. 86400 IN NSEC3 1 1 0 - HPVVP23QUO0FP9R0A04URSICJPESKO9J NS DS RRSIG
HPVV2B5N85O7HJJRB7690IB5UVF9O9UA.com. 86400 IN RRSIG NSEC3 8 2 86400 20180619043222 20180612032222 36707 com. e7PaDpbSrn/ckI2xSM3CfjUofaBfLGRHb0i9qfiW5JfPEYs04sF2TY27 iILXwNC+galMIJvTZNVh26xEGMWAobA+ehImRXk0f/x5yU1xgQDDN5nY BOa1pKA08/M4hxrosGfiQw4vYJBnWYkpqpgySVHKW63mzomWErEf1SLx dW0=
;; Received 697 bytes from 192.35.51.30#53(f.gtld-servers.net) in 324 ms

www.baidu.com.		1200	IN	CNAME	www.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns5.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns4.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns3.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns1.a.shifen.com.
a.shifen.com.		1200	IN	NS	ns2.a.shifen.com.
;; Received 239 bytes from 180.76.76.92#53(ns7.baidu.com) in 16 ms

```

- `+short 选项` 仅输出CNAME信息和A记录

```
[root@stevey test]# dig @8.8.8.8 +short www.baidu.com
www.a.shifen.com.
220.181.112.244
220.181.111.188
```

### nslookup命令

跟dig一样，nslookup也是在bind-utils包里的，安装dig的时候，买一送一。

#### nslookup 有两种工作模式：交互模式、非交互模式

##### 交互模式

进入方式：

1. 直接输入nslookup命令，不加任何参数，则直接进入交互模式；

2. 输入 nslookup - [域名服务器主机名或ip地址]

退出方式：输入`exit`

**用法：**

```
[root@stevey test]# nslookup
//输入查询的域名
> baidu.com      
Server:		129.29.29.29
Address:	129.29.29.29#53

Non-authoritative answer:
Name:	baidu.com
Address: 123.125.115.110
Name:	baidu.com
Address: 220.181.57.216  

//指定查询类型
> set type=ns
> baidu.com
Server:		129.29.29.29
Address:	129.29.29.29#53

Non-authoritative answer:
baidu.com	nameserver = ns3.baidu.com.
baidu.com	nameserver = ns4.baidu.com.
baidu.com	nameserver = dns.baidu.com.
baidu.com	nameserver = ns7.baidu.com.
baidu.com	nameserver = ns2.baidu.com.


//set all 列出nslookup工具的常用选项的当前设置值

> set all
Default server: 129.29.29.29
Address: 129.29.29.29#53
Default server: 8.8.8.8
Address: 8.8.8.8#53

Set options:
  novc			nodebug		nod2
  search		recurse
  timeout = 0		retry = 3	port = 53
  querytype = A       	class = IN
  srchlist =
```

##### 非交互模式

直接输入 `nslookup [查询的IP或主机名]`

```
[root@stevey test]# nslookup www.baidu.com
Server:		129.29.29.29
Address:	129.29.29.29#53

Non-authoritative answer:
www.baidu.com	canonical name = www.a.shifen.com.
Name:	www.a.shifen.com
Address: 220.181.112.244
Name:	www.a.shifen.com
Address: 220.181.111.188

```

### `host` 命令

- 同样查询域名ip信息，默认返回A记录和MX记录。

```
[root@stevey test]# host baidu.com
baidu.com has address 220.181.57.216
baidu.com has address 123.125.115.110
baidu.com mail is handled by 20 jpmx.baidu.com.
baidu.com mail is handled by 15 mx.n.shifen.com.
baidu.com mail is handled by 10 mx.maillb.baidu.com.
baidu.com mail is handled by 20 mx50.baidu.com.
baidu.com mail is handled by 20 mx1.baidu.com.
```

- `host -a [域名]`：返回详细信息，与`域名]`：返回详`类似

```
[root@stevey test]# host -a www.baidu.com
Trying "www.baidu.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55109
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.baidu.com.			IN	ANY

;; ANSWER SECTION:
www.baidu.com.		879	IN	CNAME	www.a.shifen.com.

Received 58 bytes from 129.29.29.29#53 in 13 ms
```

- `host -c [域名]` 查询权威域名服务器信息

``` 
[root@stevey test]# host -C bjucd.com
Nameserver 118.145.6.5:
	bjucd.com has SOA record ns7.dns.net.cn. admin.dns.net.cn. 56 900 600 86400 3600
Nameserver 61.135.129.2:
	bjucd.com has SOA record ns.dns.net.cn. admin.dns.net.cn. 56 900 60
```

## 参考资料

* [DNS根服务器、根服务器、全球13台根域名服务器、详细介绍](https://www.cloudxns.net/Support/detail/id/1463.html)

* [wiki：Root name server](https://en.wikipedia.org/wiki/Root_name_server)

* [《dig挖出DNS的秘密》-linux命令五分钟系列之三十四](http://roclinux.cn/?p=2449)

* [Linux命令：使用dig命令解析域名](https://blog.csdn.net/reyleon/article/details/12976889)

* [通过dig命令理解DNS](https://www.jianshu.com/p/71f61652ec23)

* [域名解析之dig,host,nslookup命令](http://luodw.cc/2015/12/27/dns03/)

* [抽象理解DNS](https://www.jianshu.com/p/62a9f68a2573)

