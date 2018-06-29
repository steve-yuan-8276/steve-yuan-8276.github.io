---
title: 'Linux运维学习笔记7-5：网络和防火墙（三）'
date: 2018-05-08 10:36:04
categories: linux
tags: [linux, firewalld, notes] 
---

### 写在前面

这一则笔记继续整理linux网络防火墙的知识，主要是firewalld的用法。

注：iptables 规则备份和恢复上一则笔记已经整理了，具体[参见这里](https://www.steve-yuan.com/2018/05/08/week7-4-howToMnageLinuxPart4/)。

### firewalld 工具

`firewalld` 是`centos 7` 下默认的防火墙前端管理工具，跟`iptables` 相比，它有两个显著区别：

* `firewalld` 使用区域和服务而不是链式规则；

* `firewalld` 动态管理规则集，允许更新规则而不破坏现有会话和连接；

#### `firewalld`的 安装及基本操作

##### 启动服务

```
sudo systemctl start firewalld     //启动firewalld 服务
sudo systemctl enable firewalld    //设置firewalld 开机启动
```

<!--more-->

##### 禁用服务

```
sudo systemctl stop firewalld       //停止firewalld服务
sudo systemctl disable firewalld    //禁止firewalld开机启动
```

##### 检查防火墙状态

```
sudo firewall-cmd --state   //running 表示正在运行；not running 表示没有运行
```

##### 查看 FirewallD 守护进程的状态

```
sudo systemctl status firewalld
```

##### 重新加载 FirewallD 配置

```
sudo firewall-cmd --reload
```
#### firewalld的9个zone

firewalld为了简化网络管理的上手难度，让不懂5表、5链的菜鸟也能够简单配置防火墙规则，提出了一个zone和service两个新概念：

* `zone`：简单来说，`zone` 就是`firewalld`预先准备了几套防火墙策略集合（策略模板），用户可以根据生产场景的不同而选择合适的策略集合，从而实现防火墙策略之间的快速切换；

* `servce`：简单来说，就是针对不同端口设置的`iptables`规则，每个`zone`都是用了不同的`service`；

**firewalld中常用的区域名称及策略规则**

区域	|默认规则策略
---|---
trusted	|允许所有的数据包
home	|拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、mdns、ipp-client、amba-client与dhcpv6-client服务相关，则允许流量
internal	|等同于home区域
work	|拒绝流入的流量，除非与流出的流量数相关；而如果流量与ssh、ipp-client与dhcpv6-client服务相关，则允许流量
public	|拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、dhcpv6-client服务相关，则允许流量
external	|拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量
dmz	|拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量
block	|拒绝流入的流量，除非与流出的流量相关
drop	|拒绝流入的流量，除非与流出的流量相关

**注：**`firewalld` 的缺省区域是 `public`。

#### firewalld 命令行格式及常用参数

命令行格式：

```
firewall-cmd [长参数]
```

**注：**用好tab键自动补齐功能。

**firewall-cmd命令中使用的参数以及作用**

参数	|作用
---|---
--get-default-zone	|查询默认的区域名称
--set-default-zone=<区域名称>	|设置默认的区域，使其永久生效
--get-zones	|显示可用的区域
--get-services	|显示预先定义的服务
--get-active-zones	|显示当前正在使用的区域与网卡名称
--add-source=	|将源自此IP或子网的流量导向指定的区域
--remove-source=	|不再将源自此IP或子网的流量导向某个指定区域
--add-interface=<网卡名称>	|将源自该网卡的所有流量都导向某个指定区域
--change-interface=<网卡名称>	|将某个网卡与区域进行关联
--list-all	|显示当前区域的网卡配置参数、资源、端口以及服务等信息
--list-all-zones	|显示所有区域的网卡配置参数、资源、端口以及服务等信息
--add-service=<服务名>	|设置默认区域允许该服务的流量
--add-port=<端口号/协议>	|设置默认区域允许该端口的流量
--remove-service=<服务名>	|设置默认区域不再允许该服务的流量
--remove-port=<端口号/协议>	|设置默认区域不再允许该端口的流量
--reload	|让“永久生效”的配置规则立即生效，并覆盖当前的配置规则
--panic-on	|开启应急状况模式
--panic-off|关闭应急状况模式

##### 关于zone的常用操作

- 查看指定网卡所在的zone

```
firewall-cmd --get-zone-of-interface=ens33
```

- 查看所有网卡所在zone

```
firewall-cmd --get-active-zones
```

- 设定默认的zone 为work

```
firewall-cmd --set-default-zone=work
```

- 给指定网卡设置zone

```
firewall-cmd --zone=public --add-interface=lo
```

- 针对指定网卡更改zone

```
firewall-cmd --zone=dmz --change-interface=lo
```

- 针对网卡删除zone

```
firewall-cmd --zone=dmz --remove-interface=lo
```

##### 关于service的常用操作

###### 查看service

- 列出所有的servie

```
firewall-cmd --get-service

```

- 查看当前zone有哪些service

```
firewll-cmd --list-services
```

- 查看指定zone下有哪些service

```
firewll-cmd --zone=public --list-services
```

###### 修改service

service的规则都是由配置文件定义的，具体保存在：

* 配置文件模版在 /usr/lib/firewalld/service/目录下；

* 真正生效的**配置文件**在/etc/firewalld/services/目录下（默认为空）；


指定zone添加一个service

```
firewall-cmd --zone=public --add-service=http    //临时改动，重启后会失效

firewall-cmd --zone=public --add-service=http --permanent   //--permanent 表示永久改动，会写入对应zone的配置文件
```

一旦改动了zone的配置文件，会在 `/etc/firewalld/zones` 目录下生成以`.xml`为后缀的配置文件。因此，我们也可以通过vim修改这个配置文件来达到修改zone规则的目的。

修改完成后，执行以下命令重新加载生效：

```
firewall-cmd --reload    //重新加载

```

**更多的示例：**

- 允许 ssh 服务通过

```
firewall-cmd --enable service=ssh

```

- 禁止 ssh 服务通过

```
firewall-cmd --disable service=ssh
```

- 临时允许 samba 服务通过 600 秒

```
firewall-cmd --enable service=samba --timeout=600
```

- 打开 443/tcp 端口在内部区域（internal）:

```
firewall-cmd --zone=internal --add-port=443/tcp 
firewall-cmd – reload
```

- 端口转发

```
firewall-cmd --zone=external --add-masquerade 
firewall-cmd --zone=external --add-forward-port=port=22:proto=tcp:toport=3777
```

### 参考资料

* [Iptables与Firewalld防火墙](https://www.linuxprobe.com/chapter-08.html)

* [使用 firewalld 构建 Linux 动态防火墙](https://www.ibm.com/developerworks/cn/linux/1507_caojh/index.html)

* [CentOS 7下Firewalld防火墙的简明教程](https://www.vmvps.com/brief-firewalld-tutorials-on-centos-7.html)

* [firewalld入门手册](https://www.linuxprobe.com/centos-firewalld.html)


