---
title: 'Linux运维学习笔记7-2：监控系统状态命令2.2'
date: 2018-05-08 08:31:05
categories: linux
tags: [linux, tcpdump, wireshark, notes] 
---

### 写在前面

这则笔记太长，上传到博客后总是显示格式错误，所以截成两段分别上传。这是后半部分。

### `netstat `命令： 查看网络情况

Netstat 是一款命令行工具，可用于查看tcp/ip的状态，包括 tcp, udp 以及 unix 套接字，还能列出处于监听状态（即等待接入请求）的套接字。

命令行格式： `netstat [参数]`

常用参数 | 含义
---|---
-a |(all)显示所有选项，默认不显示LISTEN相关；
-t |(tcp)仅显示tcp相关选项；
-u |(udp)仅显示udp相关选项；
-n |拒绝显示别名，能显示数字的全部转化成数字；
-l |仅列出有在 Listen (监听) 的服務状态；
-p |显示建立相关链接的程序名；
-r |显示路由信息，路由表；
-e |显示扩展信息，例如uid等；
-s |按各个协议进行统计；
-c |每隔一个固定时间，执行该netstat命令；

<!--more-->

#### netstat 能查看哪些信息？

##### 打印当前系统启动哪些服务

```
netstat -lnp | head -n 30
```

![](https://farm1.staticflickr.com/978/41817123212_430ecb0817_o.png)

##### 查看网络连接状况

```
netstat -an |head -n 30
```

![](https://farm1.staticflickr.com/980/40961187725_8e899e73ed_o.png)

##### 统计各个状态的线程数

```
[root@stevey sa]# netstat -an | awk '/^tcp/ {++sta[$NF]} END {for(key in sta) print key,"\t",sta[key]}'
LISTEN 	 4
ESTABLISHED 	 1
```

**注：**如果`ESTABLISHED` 数值过高，说明服务器压力很大。

#### 更多netstat 命令示例


##### 列出所有端口:包括监听和未监听的

![](https://farm1.staticflickr.com/975/40156693000_b8708d744a_o.png)

                       
##### 列出所有处于监听状态的 Sockets


```
netstat -l        #只显示监听端口
netstat -lt       #只列出所有监听 tcp 端口
netstat -lu       #只列出所有监听 udp 端口
netstat -lx       #只列出所有监听 UNIX 端口
```

##### 显示每个协议的统计信息


```
netstat -s   显示所有端口的统计信息
netstat -st   显示TCP端口的统计信息
netstat -su   显示UDP端口的统计信息
```

##### 在netstat输出中显示 PID 和进程名称


```
netstat -pt
```

**注：**netstat -p可以与其它开关一起使用，就可以添加“PID/进程名称”到netstat输出中，这样debugging的时候可以很方便的发现特定端口运行的程序。

##### 查找指定进程信息


```
netstat -ap | grep ssh    //找出程序运行的端口；注：并不是所有的进程都能找到，没有权限的会不显示，使用 root 权限查看所有的信息。

netstat -an | grep ':80'    //找出运行在指定端口的进程：
```

##### IP和TCP分析

- 显示网络接口列表


```
netstat -i
```

- 查看连接某服务端口最多的的IP地址：


```
netstat -ntu | grep :80 | awk '{print $5}' | cut -d: -f1 | awk '{++ip[$1]} END {for(i in ip) print ip[i],"\t",i}' | sort -nr
```

- TCP各种状态列表：


```
netstat -nt | grep -e 127.0.0.1 -e 0.0.0.0 -e ::: -v | awk '/^tcp/ {++state[$NF]} END {for(i in state) print i,"\t",state[i]}'
```

### `ss `命令

`ss`是Socket Statistics的缩写，可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat**更快速更高效**。

netstat命令是net-tools工具集中的一员，而ss命令是iproute工具集中的一员，如果无法使用，则需要手动安装：


```
yum install iproute iproute-doc

```

iproute工具集几乎可以替代掉net-tools工具集，具体的替代方案如下：
 
用途	|net-tool（替代）	|iproute2
---|---|---
地址和链路配置	|ifconfig	|ip addr, ip link
路由表	|route	|ip route
邻居	|arp	|ip neigh
VLAN	|vconfig	|ip link
隧道	|iptunnel	|ip tunnel
组播	|ipmaddr	|ip maddr
统计	|netstat	|ss

命令格式:

```
ss [参数] [过滤]
```

#### ss 使用案例

##### 查看当前服务器的网络连接统计

```
[root@stevey sa]# ss -s
Total: 561 (kernel 1020)
TCP:   5 (estab 1, closed 0, orphaned 0, synrecv 0, timewait 0/0), ports 0

Transport Total     IP        IPv6
*	  1020      -         -
RAW	  1         0         1
UDP	  0         0         0
TCP	  5         3         2
INET	  6         3         3
FRAG	  0         0         0
```

##### 查看所有打开的网络端口

```
[root@stevey sa]# ss -l
Netid  State      Recv-Q Send-Q Local Address:Port                 Peer Address:Port
nl     UNCONN     0      0          rtnl:452985400                    *
nl     UNCONN     0      0          rtnl:kernel                       *
nl     UNCONN     0      0          rtnl:452985400                    *
nl     UNCONN     768    0       tcpdiag:kernel                       *
nl     UNCONN     4352   0       tcpdiag:ss/14702                     *
nl     UNCONN     0      0          xfrm:kernel                       *
nl     UNCONN     0      0       selinux:dbus-daemon/539              *
nl     UNCONN     0      0       selinux:kernel                       *
nl     UNCONN     0      0       selinux:systemd/1                    *
```

**注：**使用`-pl`参数的话，则会列出具体的程序名称

##### 列出所有的网络连接


```
[root@stevey sa]# ss -an |grep -i listen    //使用`-a`选项即可列出所有网络连接；`-n` 不尝试解析服务的名字
u_str  LISTEN     0      100    public/flush 19659                 * 0
u_str  LISTEN     0      100    public/showq 19674                 * 0
u_str  LISTEN     0      128    /var/run/dbus/system_bus_socket 15671                 * 0
u_str  LISTEN     0      128    /run/lvm/lvmpolld.socket 12861                 * 0
u_str  LISTEN     0      128    /run/systemd/private 12610                 * 0
u_seq  LISTEN     0      128    /run/udev/control 12916                 * 0
u_str  LISTEN     0      32     /var/run/vmware/guestServicePipe 16504                 * 0
u_str  LISTEN     0      128    /run/lvm/lvmetad.socket 12673                 * 0
```

**更多参数**

* 查看TCP sockets，那么使用-ta选项；

* 查看UDP sockets，那么使用-ua选项；

* 查看RAW sockets，那么使用-wa选项；

* 查看UNIX sockets，那么使用-xa选项；

### linux 抓包工具

#### `tcpdump` 抓取数据

tcpdump是一个用于截取网络分组，并输出分组内容的工具，简单说就是数据包抓包工具。tcpdump凭借强大的功能和灵活的截取策略，使其成为Linux系统下用于网络分析和问题排查的首选工具。

`tcpdump` 需要安装：

```
yum install -y tcpdump
```

命令行格式： `tcpdump [参数]` 

参数|含义
---|---
-A |以ASCII格式打印出所有分组，并将链路层的头最小化。
-c |在收到指定的数量的分组后，tcpdump就会停止。
-C |在将一个原始分组写入文件之前，检查文件当前的大小是否超过了参数file_size 中指定的大小。如果超过了指定大小，则关闭当前文件，然后在打开一个新的文件。参数 file_size 的单位是兆字节（是1,000,000字节，而不是1,048,576字节）。
-d |将匹配信息包的代码以人们能够理解的汇编格式给出。
-dd |将匹配信息包的代码以c语言程序段的格式给出。
-ddd |将匹配信息包的代码以十进制的形式给出。
-D |打印出系统中所有可以用tcpdump截包的网络接口。
-e |在输出行打印出数据链路层的头部信息。
-E |用spi@ipaddr algo:secret解密那些以addr作为地址，并且包含了安全参数索引值spi的IPsec ESP分组。
-f |将外部的Internet地址以数字的形式打印出来。
-F |从指定的文件中读取表达式，忽略命令行中给出的表达式。
-i |指定监听的网络接口。
-l |使标准输出变为缓冲行形式，可以把数据导出到文件。
-L |列出网络接口的已知数据链路。
-m |从文件module中导入SMI MIB模块定义。该参数可以被使用多次，以导入多个MIB模块。
-M |如果tcp报文中存在TCP-MD5选项，则需要用secret作为共享的验证码用于验证TCP-MD5选选项摘要（详情可参考RFC 2385）。
-b |在数据-链路层上选择协议，包括ip、arp、rarp、ipx都是这一层的。
-n |不把网络地址转换成名字。
-nn |不进行端口名称的转换。
-N |不输出主机名中的域名部分。例如，‘nic.ddn.mil‘只输出’nic‘。
-t |在输出的每一行不打印时间戳。
-O |不运行分组分组匹配（packet-matching）代码优化程序。
-P |不将网络接口设置成混杂模式。
-q |快速输出。只输出较少的协议信息。
-r |从指定的文件中读取包(这些包一般通过-w选项产生)。
-S |将tcp的序列号以绝对值形式输出，而不是相对值。
-s |从每个分组中读取最开始的snaplen个字节，而不是默认的68个字节。
-T |将监听到的包直接解释为指定的类型的报文，常见的类型有rpc远程过程调用）和snmp（简单网络管理协议；）。
-t |不在每一行中输出时间戳。
-tt |在每一行中输出非格式化的时间戳。
-ttt |输出本行和前面一行之间的时间差。
-tttt |在每一行中输出由date处理的默认格式的时间戳。
-u |输出未解码的NFS句柄。
-v |输出一个稍微详细的信息，例如在ip包中可以包括ttl和服务类型的信息。
-vv |输出详细的报文信息。
-w |直接将分组写入文件中，而不是不分析并打印出来。

##### tcpdump 正则表达式

tcp还可以配合正则表达式，匹配出符合特定特定条件的内容。在表达式中一般如下几种类型的关键字：

###### 第一种：类型的关键字

* host(缺省类型): 指明一台主机，如：host 210.27.48.2

* net: 指明一个网络地址，如：net 202.0.0.0

* port: 指明端口号，如：port 23

###### 第二种：确定传输方向的关键字

* src: src 210.27.48.2, IP包源地址是210.27.48.2

* dst: dst net 202.0.0.0, 目标网络地址是202.0.0.0

* dst or src(缺省值)

* dst and src

###### 协议关键字：缺省值是监听所有协议的信息包

* fddi

* ip

* arp

* rarp

* tcp

* udp

**注：**Fddi指明是在FDDI(分布式光纤数据接口网络)上的特定的网络协议，实际上它是"ether"的别名，fddi和ether具有类似的源地址和目的地址，所以可以将fddi协议包当作ether的包进行处理和分析。其他的几个关键字就是指明了监听的包的协议内容。如果没有指定任何协议，则tcpdump将会监听所有协议的信息包。

###### 其他关键字

* gateway

* broadcast

* less

* greater

###### 逻辑运算

* 非 : ! or “not” (去掉双引号)

* 且 : && or “and”

* 或 : || or “or”

上述这些关键字组合起来，可以使用正则表达式进行更为精确的查询

##### tcpdump 常见用法

- 指定网卡抓包


```
//-nn表示不进行端口名称的转换, -i表示监听指定的网络接口
tcpdump -nn -i ens33  
```

**注：**重点观察红框部分，看数据流向

![](https://farm1.staticflickr.com/826/41907998552_ebb392482a_o.png)


- 指定监听80端口

```
tcpdump -nn port 80
```

- 监听指定网卡并排除某些端口

```
//排除22端口
tcp -nn -i ens33 not port 22
```

- 指定抓包个数并保存

```
tcpdump -nn -i ens33 -c 100 -w /tmp/a.txt   // -c 选项后面跟抓包个数； -w后面跟保存文件路径

```

**注：**读取数据包，使用 `tcpdump -r [filename]`

- 指定主机抓包


```
//截获主机172.16.14.107与主机172.16.14.27或172.16.14.99的通信  
//注：使用`（）`要配合转义符号’\’
tcpdump host 172.16.14.107 and \ (172.16.14.27or172.16.14.99 \)

//获取主机172.16.14.107除了和主机172.16.14.27之外所有主机通信的ip包：
tcpdump ip host 172.16.14.107 and ! 172.16.14.27

//获取主机172.16.14.107接收或发出的telnet包：
tcpdump tcp port 23 host 172.16.14.107

//读取主机hostname发送的所有数据： 
tcpdump -i eth0 src host hostname

//监视所有送到主机hostname的数据包： 
tcpdump -i eth0 dst host hostname
```

#### `wireshark` 数据分析工具

##### wireshark 安装

```
yum install -y wireshark
```

##### wireshark 使用


- 显示访问http请求的域名以及uri

```
tshark -n -t a -R http.request -T fields -e "frame.time" -e "ip.src" -e "http.host" -e "http.request.method" -e "http.request.uri"

```
- 抓取mysql的查询

```
tshark -n -i eth1 -R 'mysql.query' -T fields -e "ip.src" -e "mysql.query"
//另外一种方法：
tshark -i eth1 port 3307  -d tcp.port==3307,mysql -z "proto,colinfo,mysql.query,mysql.query"

```
- 抓取指定类型的MySQL查询

```
tshark -n -i eth1 -R 'mysql matches "SELECT|INSERT|DELETE|UPDATE"' -T fields -e "ip.src" -e "mysql.query"
```

- 统计http的状态

```
tshark -n -q -z http,stat, -z http,tree
这个命令，直到你ctrl + c 才会显示出结果
```

- tshark 增加时间标签   

```
tshark  -t  ad
tshark  -t  a
```

- 打印http协议流相关信息

```
tshark -s 512 -i eth0 -n -f 'tcp dst port 80' -R 'http.host and http.request.uri' -T fields -e http.host -e http.request.uri -l | tr -d '\t'
　　
　　注：
　　　　-s: 只抓取前512字节；
　　　　-i: 捕获eth0网卡；
　　　　-n: 禁止网络对象名称解析;
　　　　-f: 只捕获协议为tcp,目的端口为80;
　　　　-R: 过滤出http.host和http.request.uri;
　　　　-T,-e: 指的是打印这两个字段;
　　　　-I: 输出到命令行界面; 
```
　　　　
　　　　
- 实时打印当前mysql查询语句

```
tshark -s 512 -i eth0 -n -f 'tcp dst port 3306' -R 'mysql.query' -T fields -e mysql.query
　　　注:-R: 过滤出mysql的查询语句;
```

- 导出smpp协议header和value的例子

```
tshark -r test.cap -R '(smpp.command_id==0x80000004) and (smpp.command_status==0x0)' -e smpp.message_id -e frame.time -T fields -E header=y >test.txt
　　　注:
　　　　-r: 读取本地文件，可以先抓包存下来之后再进行分析;
　　　　-R: smpp...可以在wireshark的过滤表达式里面找到，后面会详细介绍;
　　　　-E: 当-T字段指定时，设置输出选项，header=y意思是头部要打印;
　　　　-e: 当-T字段指定时，设置输出哪些字段;
　　　　 >: 重定向;
```

- 统计http状态

```
tshark -n -q -z http,stat, -z http,tree
　　　
　　　注:
　　　　-q: 只在结束捕获时输出数据，针对于统计类的命令非常有用;
　　　　-z: 各类统计选项，具体的参考文档，后面会介绍，可以使用tshark -z help命令来查看所有支持的字段;
　　　　http,stat: 计算HTTP统计信息，显示的值是HTTP状态代码和HTTP请求方法。
　　　　http,tree: 计算HTTP包分布。 显示的值是HTTP请求模式和HTTP状态代码。
```
　　　　　　 
　　　　　　 
-  抓取500个包提取访问的网址打印出来

```
tshark -s 0 -i eth0 -n -f 'tcp dst port 80' -R 'http.host and http.request.uri' -T fields -e http.host -e http.request.uri -l -c 500
　　　
　　　注: 
　　　　-f: 抓包前过滤；
　　　　-R: 抓包后过滤；
　　　　-l: 在打印结果之前清空缓存;
　　　　-c: 在抓500个包之后结束;

```

- 显示ssl data数据

```
tshark -n -t a -R ssl -T fields -e "ip.src" -e "ssl.app_data"
```

- 读取指定报文,按照ssl过滤显示内容

```
tshark -r temp.cap -R "ssl" -V -T text
```
　　
　　**注:** 
　　　　-T text: 格式化输出，默认就是text;
　　　　-V: 增加包的输出;
　　　　
　　　　//-q 过滤tcp流13，获取data内容
`tshark -r temp.cap -z "follow,tcp,ascii,13"`

- 按照指定格式显示-e

```
tshark -r temp.cap -R ssl -Tfields -e "ip.src" -e tcp.srcport -e ip.dst -e tcp.dstport
```

- 输出数据

```
tshark -r vmx.cap -q -n -t ad -z follow,tcp,ascii,10.1.8.130:56087,10.195.4.41:446 | more
```
　　
　　**注:** -t ad: 输出格式化时间戳;

- 过滤包的时间和rtp.seq

```
tshark  -i eth0 -f "udp port 5004"  -T fields -e frame.time_epoch -e rtp.seq -o rtp.heuristic_rtp:true 1>test.txt
```
　　
　　**注:** -o: 覆盖属性文件设置的一些值;

- 提取各协议数据部分

```
tshark -r H:/httpsession.pcap -q -n -t ad -z follow,tcp,ascii,71.6.167.142:27017,101.201.42.120:59381 | more
```


### 参考资料：

* [通俗大白话来理解TCP协议的三次握手和四次分手](https://github.com/jawil/blog/issues/14)

* [tcp的三次握手和四次挥手](http://blog.51cto.com/10690709/2113525)

* [Linux查看应用可用内存-free命令详解](https://blog.csdn.net/loongshawn/article/details/51758116)

* [10个重要的Linux ps命令实战](https://linux.cn/article-4743-1.html)

* [每天一个linux命令（41）：ps命令](https://www.cnblogs.com/peida/archive/2012/12/19/2824418.html)

* [netstat 的10个基本用法](https://linux.cn/article-2434-1.html)

* [Linux抓包工具tcpdump详解](http://www.ha97.com/4550.html)

* [Wireshark命令行工具tshark使用小记](https://www.cnblogs.com/liun1994/p/6142505.html)


