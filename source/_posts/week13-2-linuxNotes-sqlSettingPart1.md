---
title: 'linux学习笔记13-2：mysql的基本使用（一）'
date: 2018-06-13 11:15:26
categories: linux
tags: [linux, mysql, notes]
---

## 写在前面

这则笔记开始整理mysql的基本配置和使用知识。之前测试环境安装的是mariaDB，不过mriaDB跟mysql兼容，下面的例子也用mariaDB来做。

## 初次登陆及基本设置

### 修改环境变量

测试环境使用的是yum安装mariaDB，其安装目录为`/usr/bin/mysql`。如果不清楚，可以使用find命令查一下：

```
[root@stevey ~]# find / -name 'mysql'
/etc/logrotate.d/mysql
/etc/rc.d/init.d/mysql
/etc/selinux/targeted/active/modules/100/mysql
/var/lib/mysql
/var/lib/mysql/mysql
/usr/bin/mysql
/usr/lib64/mysql
/usr/share/mysql
/usr/local/src/php-7.2.6/travis/ext/mysql
[root@stevey ~]# which mysql
/usr/bin/mysql
```

如果采用源码包安装的mysql，一般安装目录在`/usr/local/mysql/` ，因此在执行命令时需采用绝对路径，非常不方便。解决方法有两种：

- 修改全局环境变量

```
vim /etc/profile

//增加一行：
export PATH=$PATH:/usr/local/mysql/bin/

//保存退出后，执行以下命令生效
source /etc/profile
```

- 设置软链接

```
ln -s /usr/local/mysql/bin/mysql /usr/bin/
```

<!--more-->

### 初次登陆

```
mysql [-u username] [-h host] [-p password] [database]
```

其中，

* `-u选项`，指定用户名；

* `-h选项`，指定主机IP地址或socket

* `-p选项`，指定密码；**注意p和密码之间没有空格**

* `database`，指定登陆哪个数据库（假设有多个数据库的话）

由于是本地虚拟机安装mysql，初次登陆也设置密码，因此只需要输入如下命令即可。

```
mysql -u root
```

#### 设置mysql root 密码(三种方式)

- 方式一：mysqladmin命令

```
mysqladmin –u root password 123456  //密码设定为123456
```

- 方法二：set password修改口令：

```
[root@stevey test]# mysql
MariaDB [(none)]> set password for root@localhost = password('PASSWORD');
MariaDB [(none)]> flush privileges;    //执行刷新命令后生效
```

- 方法三： UPDATE 语句来设置密码：

```
MariaDB [(none)]> update mysql.user set password = password('PASSWORD') where user='root';
MariaDB [(none)]> flush privileges;    //执行刷新命令后生效
```

#### 用户授权

##### 本机操作

```
//all 表示所有权限（读、写、执行）；
//`*.* ` 前一个星号表示所有的数据库，后一个表示所有的表；
//username 表示用户名；
//identified by 后面跟的是密码；
grant all on *.* to username identified by 'abcdef';
```

##### 远程操作

```
//hostip是远程主机的ip
grant all on db1.* to 'username'@'hostIP' identified by 'abcdef';
```

#### 查看当前用户表

```
MariaDB [(none)]> select User,Host,Password from mysql.user;
+------+-----------+-------------------------------------------+
| User | Host      | Password                                  |
+------+-----------+-------------------------------------------+
| root | localhost | *2E31C5EB30A0D665E861AB5277A2508FB7E7C16A |
| root | 127.0.0.1 | *2E31C5EB30A0D665E861AB5277A2508FB7E7C16A |
| root | ::1       | *2E31C5EB30A0D665E861AB5277A2508FB7E7C16A |
+------+-----------+-------------------------------------------+
3 rows in set (0.13 sec)
```

#### 重置root密码

- 修改配置文件

```
vim /etc/my.cnf

//在[mysql]下一行增加如下内容：
skip-grant
```

- 重启mysql

```
/etc/init.d/mysqld restart
```

- 进入密码表更改密码

```
MariaDB [(none)]> use mysql;  //用户名密码存在user表里，而user表存在mysql这个库里，进入mysql，记得加分号

MariaDB [(none)]> select * from user;  //查看user表

MariaDB [(none)]> select password from user where user='root' ; //查询语句查询密码表。加密的字符串是password这个函数生成
```

- 修改配置文件

```
vim /etc/my.cnf

//删除如下内容行或
skip-grant

```

- 重新登陆mysql

```
mysql -u root
```

### mysql常用命令

- 查询当前库

```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.03 sec)

```

- 切换库

```
use mysql;
```

![](https://farm2.staticflickr.com/1722/28023907667_2908e380c3_o.png)

- 查询库的表

```
MariaDB [mysql]> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| column_stats              |
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| gtid_slave_pos            |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| host                      |
| index_stats               |
| innodb_index_stats        |
| innodb_table_stats        |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| roles_mapping             |
| servers                   |
| slow_log                  |
| table_stats               |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
30 rows in set (0.00 sec)
```

- 查看表里的字段

```
desc user;
```

![](https://farm2.staticflickr.com/1731/28023957007_ffc9b6e5e4_o.png)

- 查看建表语句

```
show create table user\G;
```
 
![](https://farm2.staticflickr.com/1745/28023976457_46d97b64ea_o.png) 

- 查看当前用户

```
MariaDB [mysql]> select user();
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)
```

- 查看当前使用的数据库

```
MariaDB [mysql]> select database();
+------------+
| database() |
+------------+
| mysql      |
+------------+
1 row in set (0.00 sec)
```
 
- 创建库

```
MariaDB [mysql]> create database db1;   //创建名为db1的库
Query OK, 1 row affected (0.00 sec)
```
 
- 创建表

```
//sql语句以分号结尾，下面实际上是将两个语句放在一行书写
//id 和 name 使用的是反引号，tab键上面的
//int(4)表示这个表仅包括整数，括号里的4表示最大为4位数
//char(40)表示固定长度（40位）的字符数；如果是可变长度的字符数，用varchar（n）表示
MariaDB [db1]> use db1; create table t1(`id` int(4), `name` char(40));
Database changed
Query OK, 0 rows affected (0.07 sec)
```
 
- 查看当前数据库版本

```
MariaDB [db1]> select version();
+-----------------+
| version()       |
+-----------------+
| 10.2.15-MariaDB |
+-----------------+
1 row in set (0.04 sec)
```
 
- 查看数据库状态

```
show status;
```
 
![](https://farm2.staticflickr.com/1761/42174799614_1f0e3cc236_o.png)


- 查看各参数

```
//查看参数
show variables; 

//查看指定的参数
show variables like 'max_connect%';

//示例
MariaDB [db1]> show variables like 'max_connect%';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| max_connect_errors | 100   |
| max_connections    | 151   |
+--------------------+-------+
2 rows in set (0.00 sec)
```
 
- 修改参数

```
//设定参数
MariaDB [db1]> set global max_connect_errors=1000;    
Query OK, 0 rows affected (0.05 sec)

//查看参数设定的结果
MariaDB [db1]> show variables like 'max_connect%';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| max_connect_errors | 1000  |
| max_connections    | 151   |
+--------------------+-------+
2 rows in set (0.00 sec)
```

 
- 查看数据库队列

`show processlist` 是显示用户正在运行的线程，需要注意的是，除了 root 用户能看到所有正在运行的线程外，其他用户都只能看到自己正在运行的线程，看不到其它用户正在运行的线程。除非单独个这个用户赋予了PROCESS 权限。

`show processlist` 默认只显示前100个字符，也就是你看到的语句可能是截断了的，要看全部信息，需要使用 `show full processlist`。

```  
MariaDB [db1]> show processlist;
+----+-------------+-----------+------+---------+------+--------------------------+------------------+----------+
| Id | User        | Host      | db   | Command | Time | State                    | Info             | Progress |
+----+-------------+-----------+------+---------+------+--------------------------+------------------+----------+
|  1 | system user |           | NULL | Daemon  | NULL | InnoDB purge worker      | NULL             |    0.000 |
|  2 | system user |           | NULL | Daemon  | NULL | InnoDB purge worker      | NULL             |    0.000 |
|  3 | system user |           | NULL | Daemon  | NULL | InnoDB purge worker      | NULL             |    0.000 |
|  4 | system user |           | NULL | Daemon  | NULL | InnoDB purge coordinator | NULL             |    0.000 |
|  5 | system user |           | NULL | Daemon  | NULL | InnoDB shutdown handler  | NULL             |    0.000 |
| 15 | root        | localhost | db1  | Query   |    0 | init                     | show processlist |    0.000 |
+----+-------------+-----------+------+---------+------+--------------------------+------------------+----------+
6 rows in set (0.00 sec)

MariaDB [db1]> show full processlist;
+----+-------------+-----------+------+---------+------+--------------------------+-----------------------+----------+
| Id | User        | Host      | db   | Command | Time | State                    | Info                  | Progress |
+----+-------------+-----------+------+---------+------+--------------------------+-----------------------+----------+
|  1 | system user |           | NULL | Daemon  | NULL | InnoDB purge worker      | NULL                  |    0.000 |
|  2 | system user |           | NULL | Daemon  | NULL | InnoDB purge worker      | NULL                  |    0.000 |
|  3 | system user |           | NULL | Daemon  | NULL | InnoDB purge worker      | NULL                  |    0.000 |
|  4 | system user |           | NULL | Daemon  | NULL | InnoDB purge coordinator | NULL                  |    0.000 |
|  5 | system user |           | NULL | Daemon  | NULL | InnoDB shutdown handler  | NULL                  |    0.000 |
| 15 | root        | localhost | db1  | Query   |    0 | init                     | show full processlist |    0.000 |
+----+-------------+-----------+------+---------+------+--------------------------+-----------------------+----------+
6 rows in set (0.00 sec)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          
```

**查询结果的说明：**

* `Id`: 就是这个线程的唯一标识，show processlist 显示的信息时来自information_schema.processlist 表，所以这个Id就是这个表的主键。当我们发现这个线程有问题的时候，可以通过 kill 命令，加上这个Id值将这个线程杀掉。

* `User`: 就是指启动这个线程的用户。

* `Host`: 记录了发送请求的客户端的 IP 和 端口号。通过这些信息在排查问题的时候，我们可以定位到是哪个客户端的哪个进程发送的请求。

* `DB`: 当前执行的命令是在哪一个数据库上。如果没有指定数据库，则该值为 NULL 。

* `Command`: 是指此刻该线程正在执行的命令，这个选项需要和state结合起来看。

* `Time`: 表示该线程处于当前状态的时间。

* `State`: 线程的状态，和 Command 对应，下面单独解释。


**mysql列出的状态`State`主要有以下几种：**

* `Checking table`：正在检查数据表；

* `Closing tables`：正在将表中修改的数据刷新到磁盘中，同时正在关闭已经用完的表。这是一个很快的操作，如果不是这样的话，就应该确认磁盘空间是否已经满了或者磁盘是否正处于重负中；

* `Connect Out`：复制从服务器正在连接主服务器；

* `Copying to tmp table on disk`：由于临时结果集大于tmp_table_size，正在将临时表从内存存储转为磁盘存储以此节省内存。

* `Creating tmp table`：正在创建临时表以存放部分查询结果；

* `deleting from main table`：服务器正在执行多表删除中的第一部分，刚删除第一个表；

* `deleting from reference tables`：服务器正在执行多表删除中的第二部分，正在删除其他表的记录；

* `Flushing tables`；正在执行FLUSH TABLES，等待其他线程关闭数据表；

* `Killed`；发送了一个kill请求给某线程，那么这个线程将会检查kill标志位，同时会放弃下一个kill请求。MySQL会在每次的主循环中检查kill标志位，不过有些情况下该线程可能会过一小段才能死掉。如果该线程程被其他线程锁住了，那么kill请求会在锁释放时马上生效；

* `Locked`；被其他查询锁住了。

* `Sending data`：正在处理SELECT查询的记录，同时正在把结果发送给客户端。

* `Sorting for group`：正在为GROUP BY做排序。

* `Sorting for order`：正在为ORDER BY做排序。

* `Opening tables`：这个过程应该会很快，除非受到其他因素的干扰。例如，在执ALTER TABLE或LOCK TABLE语句行完以前，数据表无法被其他线程打开。正尝试打开一个表。

* `Removing duplicates`：正在执行一个SELECT DISTINCT方式的查询，但是MySQL无法在前一个阶段优化掉那些重复的记录。因此，MySQL需要再次去掉重复的记录，然后再把结果发送给客户端。

* `Reopen table`：获得了对一个表的锁，但是必须在表结构修改之后才能获得这个锁。已经释放锁，关闭数据表，正尝试重新打开数据表。

* `Repair by sorting`：修复指令正在排序以创建索引。

* `Repair with keycache`：修复指令正在利用索引缓存一个一个地创建新索引。它会比Repair by sorting慢些。

* `Searching rows for update`：正在讲符合条件的记录找出来以备更新。它必须在UPDATE要修改相关的记录之前就完成了。

* `Sleeping`：正在等待客户端发送新请求.

* `System lock`：正在等待取得一个外部的系统锁。如果当前没有运行多个mysqld服务器同时请求同一个表，那么可以通过增加--skip-external-locking参数来禁止外部系统锁。

* `Upgrading lock`：INSERT DELAYED正在尝试取得一个锁表以插入新记录。

* `Updating`：正在搜索匹配的记录，并且修改它们。

* `User Lock`：正在等待GET_LOCK()。

* `Waiting for tables`：该线程得到通知，数据表结构已经被修改了，需要重新打开数据表以取得新的结构。然后，为了能的重新打开数据表，必须等到所有其他线程关闭这个表。以下几种情况下会产生这个通知：FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE,或OPTIMIZE TABLE。

* `waiting for handler insert`：INSERT DELAYED已经处理完了所有待处理的插入操作，正在等待新的请求。




