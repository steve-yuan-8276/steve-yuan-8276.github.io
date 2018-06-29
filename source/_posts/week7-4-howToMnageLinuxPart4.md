---
title: 'Linux运维学习笔记7-4：网络与防火墙（二）'
date: 2018-05-08 10:32:48
categories: linux
tags: [linux,  notes] 
---

### 写在前面

接着上一期：Linux运维学习笔记7-3：网络和防火墙（一），继续整理关于`iptables` 和 `netfilter`的内容。

### 编写脚本定义iptables 规则

**需求：**只针对filter表，预设策略Input链DROP、其他两个链ACCEPT；然后针对192.168.188.0/24网段开通22端口，对所有网段开通80端口和21端口。

**脚本实现：**

```
# /bin/bash

ipt='/etc/sbin/iptables'    //调用iptables，这里要用绝对命令
$ipt -F       //删除所有规则
$ipt -P INPUT DROP    //默认对filter表定义预设策略，所有input流量都丢弃
$ipt -P OUTPUT ACCEPT    //定义预设策略，所有output流量都放行
$ipt -P FORWARD ACCEPT   //表定义预设策略，所有forward流量都放行
$ipt -I state --state RELATED,ESTABLISHED -j ACCEPT   //相关数据包放行
$ipt -A INPUT -s 192.168.188.0/24 -p tcp --dport 22 -j ACCEPT
$ipt -A INPUT -p tcp --dport 80 -j ACCEPT
$ipt -A INPUT -p tcp --dport 21 -j ACCEPT

```

<!--more-->

### `nat`表应用

什么nat表？

nat表是iptables当中的一张表，作用是网络地址转换（Network Address Translation）。

如果第一个数据包允许经行NAT或Masquerade,那么其它数据包都会被做相同的动作，也就是其他数据包不会被一个一个地NAT（属于一个流的包只会经过这个表一次）任何时候都不要在这个表任何一条链经行过滤。

包括三个动作：

#### `SNAT`: 

全称Source Network Address Translation，源地址转换，通常被叫做源映谢，指的是改变数据包的源地址，使局域网能访问公网。

**典型应用：**多台电脑公用一个路由器上网，路由器给每个电脑都分配了一个内网ip地址。当电脑访问外部网络的时候，路由器将数据包的报头中的源地址替换成路由器的ip。外网服务记下来的地址，并不是电脑的内网ip地址，而是路由器ip地址。这样就可以实现多台电脑公用一个路由器上网的目的。


#### `DNAT`：

全称`Destination Network Address Translation`,目的地址转换，通常被叫做目的映谢，指的是改变数据包的目的地址使包能重路由到某台机器，使公网能够访问局域网的服务器。

典型应用：比如在内网架设了一个服务器用来建立网站，内网配置内网ip，前段防火墙配置公网ip。当互联网上的用户来访问这个网站的时候，用户客户端发出的数据包包头目的地ip写的是防火墙公网ip，防火墙会将目的地ip改写为内网服务器ip，然后再把这个数据包发给内网服务器。这样以来，就实现了网络访问内网服务器的目的。

#### `MASQUERADE`:

地址伪装，算是snat中的一种特例，作用是从服务器的网卡上，自动获取当前ip地址来做NAT。

具体而言，不管现在eth0的出口获得哪个动态ip，MASQUERADE都会自动读取eth0现在的ip地址然后做SNAT出去，这样就实现了很好的动态SNAT地址转换。


Nat表包含3条链：

* `PREROUTING`链 :数据包到达防火墙时改变包的目的地址。

* `OUTPUT`链：改变本地产生数据包的目标地址。

*` POSTROUTING`链:在数据包离开防火墙时改变数据包的源地址。

路由器转发的功能其实是由Linux的iptables实现，而iptables又是通过nat表实现的。

修改net表开启路由器转发：

```
echo '1' > /proc/sys/net/ipv4/ip-forward   //打开路由器转发功能

iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE   //设置ip转发操作， -o 后跟设备名，表示出口网卡；MASQUERADE表示伪装。

```

### iptables的启用、停止及保存备份

#### iptables启用

centos7 默认的防火墙管理软件是`firewalld`；centos6.x之前的版本用的是`iptables`。想要在centos7上使用`iptables`，首先要禁用`firewalld`。

实现步骤如下：

```
systemctl stop firewalld    //关闭firealld

systemctl disable firewalld   //禁止firewalld开机启动

yum install -y iptables-services   //安装iptables

systemctl enable iptables   //设置iptables开机启动

systemctl start iptables    //开启iptables
```

#### iptables关闭

```
service iptables stop    //停止防火墙，一旦重新设定规则，防火墙还会重新启动
```

#### iptables 保存

通过命令行设置的规则都保存在内存当中，一旦关机、重启，之前设置的规则就会失效；因此，设定好规则之后需要执行以下操作保存：

```
service iptables save
```

设置好的规则会被保存到`/etc/sysconfig/iptables` 当中。


#### service 备份及恢复

```
iptables-save > myipt.rule   //备份， > 表示输出重定向

iptables-restore < myipt.rule   //恢复，中间的 < 表示输入重定向
```


### 扩展知识

#### iptables应用在一个网段

需求：对于 61.4.176.0-61.4.191.255这个网段的input流量包丢弃

```
iptables -I INPUT -m iprange --src-range 61.4.176.0-61.4.191.255 -j DROP   // -m iprange --src-range 这个参数针对一个网段
```

另一种实现方式：

```
iptables -I INPUT -s 61.4.176.0/61.4.191.255 -j DROP
```

#### 使用 iptables 防止 DDos 攻击

第一种方法：

```
iptables -A FORWARD -p tcp --syn -m limit --limit 1/s --limit-burst 5 -j ACCEPT       //添加规则，每秒中最多允许5个新连接

iptables -A FORWARD -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s -j ACCEPT   //添加规则，防止各种端口扫描

iptables -A FORWARD -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT     //添加规则，防止Ping洪水攻击（Ping of Death）

service iptables save 
```
 
 
第二种方法：直接编辑iptables配置文件


```
vim /etc/sysconfig/iptables

//加入下面几行
#anti syn，ddos
-A FORWARD -p tcp --syn -m limit --limit 1/s --limit-burst 5 -j ACCEPT
-A FORWARD -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s -j ACCEPT
-A FORWARD -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT

//保存退出后重启防火墙
service iptables restart
```

#### `iptables `的常用规则

- 阻挡自定义的恶意 IP 地址的访问

```
iptables -A INPUT -s xxx.xxx.xxx.xxx -j DROP
```

- 防 Ping

```
iptables -A INPUT -p icmp -j DROP
```

- 禁止 ICMP PING

```
iptables -A INPUT -p icmp -m icmp --icmp-type echo-request -j DROP

```

- 防止同步包洪水（Sync Flood）

```
iptables -A FORWARD -p tcp --syn -m limit --limit 1/s -j ACCEPT
```

- 防止端口扫描

```
iptables -A FORWARD -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s -j ACCEPT
```

- 防止 Ping 洪水攻击（Ping of Death）注：限制的速度根据自身情况调整

```
iptables -A INPUT -p tcp -m state --state NEW -m limit --limit 100/second --limit-burst 300 -j ACCEPT

iptables -A INPUT -p tcp -m state --state NEW -j DROP
```

- 防止 NMAP FIN/URG/PSH

```
iptables -A INPUT -i eth0 -p tcp --tcp-flags ALL FIN,URG,PSH -j DROP
```

- 丢弃异常的 XMAS 数据包 (异常的 XMAS 数据包攻击的后果: 可能导致某些系统崩溃)

```
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP

iptables -A INPUT -p tcp --tcp-flags ALL FIN,PSH,URG -j DROP

iptables -A INPUT -p tcp --tcp-flags ALL SYN,RST,ACK,FIN,URG -j DROP
```

- 丢弃 NULL 空数据包

```
iptables -A INPUT -i eth0 -p tcp --tcp-flags ALL NONE -j DROP
```

- 丢弃 Fragments 碎片数据包 (碎片数据包攻击的后果: 可能导致正常数据包丢失)

```
iptables -A INPUT -f -j DROP

```

- SYN/RST

```
iptables -A INPUT -i eth0 -p tcp --tcp-flags SYN,RST SYN,RST -j DROP
```

- SYN/FIN -- Scan(possibly)

```
iptables -A INPUT -i eth0 -p tcp --tcp-flags SYN,FIN SYN,FIN -j DROP
```

- 限制对内部封包的发送速度

![](https://farm1.staticflickr.com/967/27123175687_6b5edac5a5_o.png)

- 限制建立联机的转发

```
iptables -A FORWARD -f -m limit --limit 100/s --limit-burst 100 -j ACCEPT
```

- 丢弃无效数据包

```
iptables -A INPUT -m state --state INVALID -j DROP
iptables -A FORWARD -m state --state INVALID -j DROP
iptables -A OUTPUT -m state --state INVALID -j DROP

```

- 阻挡欺诈 IP 地址的访问 (以下为 RFC1918 类型和 IANA 预留地址，多为 LAN 或者多播地址，这些是不可能作为公网地址源的)

```
iptables -A INPUT -s 10.0.0.0/8 -j DROP
iptables -A INPUT -s 169.254.0.0/16 -j DROP
iptables -A INPUT -s 172.16.0.0/12 -j DROP
iptables -A INPUT -s 127.0.0.0/8 -j DROP
iptables -A INPUT -s 224.0.0.0/4 -j DROP
iptables -A INPUT -d 224.0.0.0/4 -j DROP
iptables -A INPUT -s 240.0.0.0/5 -j DROP
iptables -A INPUT -d 240.0.0.0/5 -j DROP
iptables -A INPUT -s 0.0.0.0/8 -j DROP
iptables -A INPUT -d 0.0.0.0/8 -j DROP
iptables -A INPUT -d 239.255.255.0/24 -j DROP
iptables -A INPUT -d 255.255.255.255 -j DROP

```


### 参考资料

* [使用 IPTABLES 防火墙阻挡常见攻击](https://www.yaoblog.info/?p=7656)

* [使用 iptables 防止 DDos 攻击](https://www.davex.pw/2017/10/01/how-to-prevent-ddos-by-iptables/)


