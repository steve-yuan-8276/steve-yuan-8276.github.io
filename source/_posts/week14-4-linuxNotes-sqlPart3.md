---
title: 'linux学习笔记: mysql的主从配置'
date: 2018-06-27 16:24:40
categories: linux
tags: [linux, mysql, notes]
---

## 写在前面

这节课开始整理mysql主从配置方面的知识。

## mysql主从配置原理

所谓mysql主从配置，就是假设有两台服务器A和B，其中A服务器为master，B服务器为slave，在A服务器master写数据时，会自动同步到B服务器slave，两台服务数据同步，保持一致。

![](https://farm1.staticflickr.com/842/43006240622_959ae6647f_o.jpg)

<!--more-->

两台服务器的通讯过程大致如下：

1. master将更改操作记录到binlog里;

2. slave将master的binlog事件(sql语句)同步到本机上并记录在relaylog里;

3. slave根据relaylog里面的sql语句按顺序执行;


## 主从mysql配置

实验在两台虚拟机上完成，其中：

* master 服务器：172.16.155.132

* slave 服务器：172.16.155.128

安装的mysql版本一致，均为5.6

```
[root@local-linux02 ftp]# mysql --version
mysql  Ver 14.14 Distrib 5.6.36, for linux-glibc2.5 (x86_64) using  EditLine wrapper

```

### 准备工作：

先在master服务器上创建一个数据库 syncMaster

```
[root@local-linux02 ~]# mysql -u root -p
Enter password:

mysql> create database syncMaster;
Query OK, 1 row affected (0.03 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| syncMaster         |
| test               |
+--------------------+
5 rows in set (0.02 sec)
```

### master服务器配置：

- 修改my.cnf配置文件

```
vi /etc/my.cnf

//增加以下配置内容：

//主数据库端id号
server_id = 100         

//开启二进制日志                  
log-bin = mysql-bin    

//需要复制的数据库名，如果复制多个数据库，重复设置这个选项即可                  
binlog-do-db = syncMaster        

//slave服务器将master更新记入到自身自己的二进制日志文件中                 
log-slave-updates                        

//定义每执行多少次事务写入一次(这个参数性能消耗很大，但可减小MySQL崩溃造成的损失) 
sync_binlog = 100                    

//一般用在主主同步中，用来错开自增值, 防止键值冲突
auto_increment_offset = 1           

//一般用在主主同步中，用来错开自增值, 防止键值冲突
auto_increment_increment = 1            

//二进制日志自动删除的天数，默认值为0,表示“没有自动删除”
expire_logs_days = 7                    

//将函数复制到slave  
log_bin_trust_function_creators = 1
```

- 创建允许从服务器同步数据的账户

```
//账户名为syncSlave，密码为123456
mysql> grant replication slave on *.* to 'syncSlave'@'172.16.155.128' identified by '123456';
Query OK, 0 rows affected (0.05 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.02 sec)
```

- 重启mysql服务

![](https://farm1.staticflickr.com/841/43067081551_44be2980b9_o.png)

- 再次进入mysql，查看主服务器状态

![](https://farm1.staticflickr.com/919/43016998022_322f9d46c9_o.png)


### 第二步：slave服务器配置：

修改配置文件

```
vi /etc/my.cnf

//修改如下配置
server_id = 101
log-bin = mysql-bin
log-slave-updates
sync_binlog = 101
innodb_flush_log_at_trx_commit = 0
replicate-do-db = syncMaster    //指定slave要复制哪个库
slave-net-timeout = 60    //网络等待时间
log_bin_trust_function_creators = 1
```

执行同步命令

```
mysql> change master to master_host='172.16.155.132',master_user='syncSlave',master_password='123456',master_log_file='mysql-bin.000005',master_log_pos=120;
Query OK, 0 rows affected, 2 warnings (0.02 sec)

mysql> start slave;
Query OK, 0 rows affected (0.01 sec)
```

查看从服务器状态

```
mysql> show slave status\G;
```

![](https://farm2.staticflickr.com/1805/28192131417_42578275b4_o.png)

### TroubleShooting 连接错误

![](https://farm1.staticflickr.com/835/28192242267_4cb05b506f_o.png)

这里本该有两个yes才对，但是`Slave_IO_Running: No`，显然出错了。

#### 排错过程：

- 查找mysql的err log 位置

```
mysql> show variables like 'log_error';
+---------------+-------------------------------+
| Variable_name | Value                         |
+---------------+-------------------------------+
| log_error     | /data/mysql/local-linux01.err |
+---------------+-------------------------------+
1 row in set (0.02 sec)
```

- 查看错误日志

```
[root@local-linux01 mysql]# tail /data/mysql/local-linux01.err
```

![](https://farm1.staticflickr.com/840/29188484618_2d5ed1c7f5_o.png)

**重点是这句 `The slave I/O thread stops because master and slave have equal MySQL server ids; these ids must be different for replication to work`，意思是说主从服务器用了同样的server ID，这样是不行的。**

- 分别检查一下，果然是一样的。

```
mysql> show variables like 'server_id';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 128   |
+---------------+-------+
1 row in set (0.00 sec)
```

- 重新编辑 `/etc/my.cnf`，发现是上一节课做实验时候的配置，这次忘记删除了，主从服务器分别修改保存，重新启动。

- 主服务器查看主服务器状态

![](https://farm2.staticflickr.com/1788/43062234741_660c699190_o.png)


- slave服务器登陆mysql，结束之前的slave进程,重新授权，再启动slave进程

```
mysql> stop slave;
Query OK, 0 rows affected (0.01 sec)

mysql> change master to master_host='172.16.155.132',master_user='syncSlave',master_password='123456',master_log_file='mysql-bin.000006',master_log_pos=120;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```

- 查看状态，见证奇迹的时刻。

![](https://farm2.staticflickr.com/1828/29190419818_8a38985b54_o.png)



### 主从服务器测试：

让我们在主服务器数据库做一些更改，然后在从服务器上看看效果。

#### 主服务器端操作

```
mysql> use syncMaster;
Database changed

mysql> create table test1(`id` int(4), `name` char(40));
Query OK, 0 rows affected (0.30 sec)

mysql> INSERT INTO testDB.t1 values (1,'abc');
Query OK, 1 row affected (0.01 sec);

mysql> show tables;
+----------------------+
| Tables_in_syncMaster |
+----------------------+
| test1                |
+----------------------+
1 row in set (0.00 sec)

mysql> select * from syncMaster.test1;
+------+------+
| id   | name |
+------+------+
|    1 | abc  |
+------+------+
1 row in set (0.00 sec)
```


#### 从服务器Mysql查询：    

```
mysql> show tables;
+----------------------+
| Tables_in_syncMaster |
+----------------------+
| test1                |
+----------------------+
1 row in set (0.00 sec)
```







