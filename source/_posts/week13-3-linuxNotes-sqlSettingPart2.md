---
title: 'linux学习笔记13-3：mysql的基本使用（二）'
date: 2018-06-19 22:16:27
categories: linux
tags: [linux, mysql, notes]
---

## 写在前面

继续整理mysql相关知识，主要包括用户管理、mysql常用语句，以及数据库的备份和恢复。

## mysql用户管理

### 添加用户并授权

#### 添加用户

命令行格式：

```
create user [username] identified by 'password';
```

示例：

```
MariaDB [(none)]> create user testUser1 identified by '123456';
Query OK, 0 rows affected (0.04 sec)
```

#### 授权

仅仅添加用户还是不够的，还需要授权，该用户才能对根据授权进行相关操作。

命令行格式：

```
grant privilegesCode on dbName.tableName to username@host identified by "password";
```

`privilegesCode`表示授予的权限类型，常用类型包括：

* `all privileges`：所有权限；

* `select`：读取权限；

* `delete`：删除权限；

* `update`：更新权限；

* `create`：创建权限；

* `drop`：删除数据库、数据表权限；

<!--more-->

`dbName.tableName`表示授予权限的具体库或表，常用的有以下几种选项：

* `.`  授予该数据库服务器所有数据库的权限；

* `dbName.*` 授予dbName数据库所有表的权限；

* `dbName.dbTable` 授予数据库dbName中dbTable表的权限；

`username@host`表示授予的用户以及允许该用户登录的IP地址。主要有一下几种类型：

* `localhost` 只允许该用户在本地登录，不能远程登录；

* `%` 允许在除本机之外的任何一台机器远程登录；

* `192.168.1.107` 表示只允许该用户从指定IP登录，再实践中，如果我们要对远程主机进行配置，应该是本机的外网ip；

#### 刷新权限变更

```
MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.03 sec)
```


### 修改密码

```
//修改密码
update mysql.user set password = password('123456') where user = 'testUser1' and host = '%';

//更新配置
flush privileges;
```


### 删除用户

```
drop user testUser@'%';
```

## 常用sql语句

### 查询语句

第一种：

```
//查询mysql数据库中的user表一共有多少行
select count(*) from mysql.user;
```


第二种：

```
//查询mysql数据库的db表中所有的数据
select * from mysql.db;

//查询单个关键词
select username from mysql.db;

//查询多个关键词，之间用逗号隔开
select username, passwd from mysql.db;

//使用通配符进行查询
select * from mysql.db where host like '192.168.%';
```

### 插入及删除

- 插入行

```
//插入一行，共两列，第一列为1，第二列为abc
MariaDB [(none)]> insert db1.t1 values(1, 'abc');
Query OK, 1 row affected (0.11 sec)

//查看当前表，校验上一步是否成功
MariaDB [(none)]> select * from db1.t1;
+------+------+
| id   | name |
+------+------+
|    1 | abc  |
+------+------+
1 row in set (0.02 sec)
```


- 更改行

```
MariaDB [(none)]> update db1.t1 set name='abcdef' where id=1;
Query OK, 1 row affected (0.14 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [(none)]> select *  from db1.t1;
+------+--------+
| id   | name   |
+------+--------+
|    1 | abcdef |
+------+--------+
1 row in set (0.00 sec)


```

### 清空某个表的数据

```
//清空db1.t1表
MariaDB [(none)]> truncate table db1.t1;
Query OK, 0 rows affected (0.14 sec)

//查看结果
MariaDB [(none)]> select * from db1.t1;
Empty set (0.00 sec)
```


### 删除表

```
//删除db1.t1表
MariaDB [(none)]> drop table db1.t1;
Query OK, 0 rows affected (0.06 sec)

//校验结果，db1.t1不存在说明已删除
MariaDB [(none)]> select * from db1.t1;
ERROR 1146 (42S02): Table 'db1.t1' doesn't exist
```

### 删除数据库

```
drop database db1；
```

![](https://farm2.staticflickr.com/1752/42866202382_915e758dfe_o.png)

## 数据库的备份和恢复


### mysqldump命令：备份

- 备份：**mysqldump有三种形式：**

```
//备份所有数据库：
mysqldump -ujack -p --all-databases > /tmp/all.sql

//备份指定数据库：
mysqldump -ujack -p test > /tmp/test.sql

//备份指定的表。可以包括多个表，以空格分隔
//备份books数据库中的orders和users表
mysqldump -ujack -p books orders users > /tmp/key.sql

//备份表结构（不备份数据本身）：
mysqldump -ujack -p --no-data books > /tmp/books.sql
```

- 恢复命令 

```
mysql -u root -p'123456' mysql < /tmp/mysql.sql;
```

### 实验测试

- 文件准备，先创建一个数据库（db2），其中有表t1，t1中包含两行数据

```

MariaDB [(none)]> create database db2;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> use db2; create table t1(`id` int(4), `name` varchar(40));
Database changed
Query OK, 0 rows affected (0.03 sec)

MariaDB [db2]> insert db2.t1 values(1, 'abc');insert db2.t1 values(2, 'def');
Query OK, 1 row affected (0.01 sec)

Query OK, 1 row affected (0.01 sec)

MariaDB [db2]> select * from db2.t1;
+------+------+
| id   | name |
+------+------+
|    1 | abc  |
|    2 | def  |
+------+------+
2 rows in set (0.00 sec)

```

- 备份数据库


```
mysqldump -u root -p'123456' db2 > /tmp/mysql.sql;
```

![](https://farm2.staticflickr.com/1766/42014038975_87f3d732b4_o.png)

- 删除数据库db2

```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| db2                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> drop database db2;
Query OK, 1 row affected (0.03 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)
```

- 恢复 

```
//直接回复显示错误
[root@stevey ~]# mysql -u root -p'holypoter5500' db2 < /tmp/mysql.sql
ERROR 1049 (42000): Unknown database 'db2'
```

```
mysql -u root -p'123456' db2 < /tmp/mysql.sql;
```

- 校验结果

```
[root@stevey ~]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 26
Server version: 10.2.15-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> select * from db1.t1;
ERROR 1146 (42S02): Table 'db1.t1' doesn't exist
MariaDB [(none)]> select * from db2.t1;
+------+------+
| id   | name |
+------+------+
|    1 | abc  |
|    2 | def  |
+------+------+
2 rows in set (0.00 sec)
```

## 参考资料

* [MySQL 学习笔记](http://idleworks.io/notes/mysql.html)

* [SQL 教程](http://www.runoob.com/sql/sql-tutorial.html)

* [SQL——什么是事务？事务的特性有哪些？]( http://blog.csdn.net/yenange/article/details/7556094)

* [mysqldump和binlog备份恢复某个库/表](https://blog.csdn.net/lilongsy/article/details/74726002)

* [mysql字符集调整总结](http://xjsunjie.blog.51cto.com/999372/1355013)


