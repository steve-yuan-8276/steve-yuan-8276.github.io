---
title: 'linux笔记：在腾讯云主机配置lnmp环境'
date: 2018-07-04 22:03:06
categories: linux
tags: [linux, notes]
---

## 写在前面

最近几天忙着做公司的半年工作报告，忙的人仰马翻，今天抓紧时间赶紧把之前两天的课程补上。

这则笔记开始整理linux集群的知识。

## 什么是集群？

所谓linux集群，就是指在生产环境中，往往不是一台服务器，而是多台服务器组成一个集群来运行程序，既可以避免单点故障，还能够提升服务器的承载能力。

### 集群类型划分

#### 负载均衡集群

就是指将一台服务器作为任务分发器，负责把用户的请求分发给后端多台不同的服务器处理，实现负载均衡的开源软件有LVS、keepalived、haproxy、nginx，商业软件有F5、Netscaler等。

#### 高可用集群（重点）

即“HA"集群，也常称作“双机热备”，用于关键业务。核心原理都是通过心跳线连接两台服务器，正常情况下一台用于提供服务，另外一台作为冗余，当提供服务的机器宕机，冗余将接替继续提供服务。实现高可用的开源软件有：heartbeat、keepalived。

**高可用集群是这则笔记讲的重点，后面用keepalived这个进行配置。**

<!--more-->

### `Keepalived`工作原理

keepalived通过VRRP`Virtual Router Redundancy Protocol`，虚拟路由冗余协议来实现高可用。VRBP是实现路由高可用的一种通信协议，在这个协议里会将多台功能相同的路由器组成一个小组，这个小组里会有1个master角色和N（N>=1）个backup角色。工作时，master会通过组播的形式向各个backup发送VRRP协议的数据包，当backup收不到master发来的VRRP数据包时，就会认为master宕机了。此时就需要根据各个backup的优先级来决定谁成为新的mater。

Keepalived要有三个模块，分别是core、check和vrrp，他们的作用如下：

1. core模块为keepalived的核心，负责主进程的启动、维护以及全局配置文件的加载和解析；

2. check模块负责健康检查；

3. vrrp模块是来实现VRRP协议的；

### keepalived配置高可用集群

**重要概念：VIP**

开始配置之前，先要了解一个概念VIP，这里的VIP跟我们平常理解的不太一样，而是“Virtual IP",即“虚拟IP"。这个IP是由Keepalived给服务器配置上的，服务器靠这个VIP对外提供服务，当master机器宕机，VIP被分配到backup上。在实际应用中，服务器的切换用户是感知不到的，所以称之为Virtual IP。

#### 准备工作

准备两台虚拟机，其中

* `master` 172.16.155.132

* `backup` 172.16.155.132

两台机器分别安装上keepalived;

```
yum install -y keepalived 
```

master和backup关闭防火墙和selinux

```
//临时关闭selinux
setenforce 0

//关闭iptables
systemctl stop iptables.service
```

#### master机器配置

##### 第一步：编辑配置文件

keepalive的配置文件为`/etc/keepalived/keepalived.conf `，编辑此文件；

```
//首先清空配置文件内容
[root@local-linux02 ~]# > /etc/keepalived/keepalived.conf

//编辑配置文件
[root@local-linux02 ~]# vi /etc/keepalived/keepalived.conf
//加入以下内容

global_defs {                   //全局定义参数
   notification_email {
     aming@aminglinux.com          //定义接收告警的人
   }
   notification_email_from root@aminglinux.com  //定义发邮件地址（实际上没用）
   smtp_server 127.0.0.1  //定义发邮件地址，若为127.0.0.1则使用本机自带邮件服务器发送
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_script chk_nginx {      //chk_nginx为自定义名字，后面还会用到它
    script "/usr/local/sbin/check_ng.sh"  //自定义脚本，该脚本为监控nginx服务的脚本
    interval 3   //每隔3S执行一次该脚本
}
vrrp_instance VI_1 {
    state MASTER         //角色为master
    interface ens33        //针对哪个网卡监听VIP
    virtual_router_id 51
    priority 100            //权重为100，master要比backup大
    advert_int 1   
    authentication {                        
        auth_type PASS
        auth_pass aminglinux>com       //定义密码，这个密码自定义
    }
    virtual_ipaddress {
        172.16.155.132    //定义VIP
    }
    track_script {
        chk_nginx     //定义监控脚本，这里和上面vrr_script后面的字符串保持一致
    }
}
```

##### 第二步：编辑监控脚本

```
[root@local-linux02 ~]# vi /usr/local/sbin/check_ng.sh  

//脚本内容如下：
#!/bin/bash
#时间变量，用于记录日志
d=`date --date today +%Y%m%d_%H:%M:%S`
#计算nginx进程数量
n=`ps -C nginx --no-heading|wc -l`
#如果进程为0，则启动nginx，并且再次检测nginx进程数量，
#如果还为0，说明nginx无法启动，此时需要关闭keepalived
if [ $n -eq "0" ]; then
        /etc/init.d/nginx start
        n2=`ps -C nginx --no-heading|wc -l`
        if [ $n2 -eq "0"  ]; then
                echo "$d nginx down,keepalived will stop" >> /var/log/check_ng.log
                systemctl stop keepalived
        fi
fi
```

##### 第三步：启动keepalived

- 更改文件权限

```
chmod 755 /usr/local/sbin/check_ng.sh 
```

- 启动keepalived

```
systemctl start keepalived
``` 

#### backup 服务器配置

##### 准备工作：yum 安装nginx

```
//安装epel-release 源
yum install -y epel-release 

//安装nginx
yum install -y nginx
```

**说明：**之所以要这样大费周章，是因为源码包安装和yum安装的版本是不同的，方便观察实验结果，在实际使用当中并没有影响。

##### 第一步：编辑配置文件

```
//首先清空配置文件内容
[root@local-linux01 ~]# > /etc/keepalived/keepalived.conf

//编辑配置文件
[root@local-linux01 ~]# vi /etc/keepalived/keepalived.conf
//加入以下内容
global_defs {
   notification_email {
     aming@aminglinux.com
   }
   notification_email_from root@aminglinux.com
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_script chk_nginx {
    script "/usr/local/sbin/check_ng.sh"  //检测脚本
    interval 3
}
vrrp_instance VI_1 {
    state BACKUP      //这个需要改，说明是从的状态
    interface ens33
    virtual_router_id 51
    priority 90      //这个权重比master少
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass aminglinux>com
    }
    virtual_ipaddress {
        172.16.155.132   //跟master保持一致
    }
    track_script {
        chk_nginx
    }
}
```

##### 第二步：编辑监控脚本

```
[root@local-linux01 ~]# vi /usr/local/sbin/check_ng.sh

//添加如下内容：

#时间变量，用于记录日志
d=`date --date today +%Y%m%d_%H:%M:%S`
#计算nginx进程数量
n=`ps -C nginx --no-heading|wc -l`
#如果进程为0，则启动nginx，并且再次检测nginx进程数量，
#如果还为0，说明nginx无法启动，此时需要关闭keepalived
if [ $n -eq "0" ]; then
        systemctl start nginx
        n2=`ps -C nginx --no-heading|wc -l`
        if [ $n2 -eq "0"  ]; then
                echo "$d nginx down,keepalived will stop" >> /var/log/check_ng.log
                systemctl stop keepalived
        fi
fi
```

##### 第三步：启动keepalived

- 更改文件权限

```
chmod 755 /usr/local/sbin/check_ng.sh 
```

- 启动keepalived

```
systemctl start keepalived
``` 

至此，配置部分就完成了，测试效果时，我们只需要在master服务器上开启iptables，然后讲相应的数据包都丢掉，就能够看出效果了。

```
systemctl restart iptables

iptables -I OUTPUT -p vrrp -j DROP 
```

