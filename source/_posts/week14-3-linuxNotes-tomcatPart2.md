---
title: 'linux学习笔记: tomcat的基础知识(二)'
date: 2018-06-27 09:10:05
categories: linux
tags: [linux, tomcat, notes]
---

## 写在前面

这节课继续整理tomcat配置相关的知识。

## tomcat的目录结构

### tomcat的目录结构说明

```
[root@local-linux02 conf]# tree -L 1 /usr/local/tomcat
/usr/local/tomcat    //tomcat目录
├── bin   //执行脚本目录
├── conf    //配置文件
├── lib   //运行时必要的库文件（JARS）
├── LICENSE
├── logs   //执行日志文件
├── NOTICE
├── RELEASE-NOTES
├── RUNNING.txt
├── temp   //临时文件存放目录
├── webapps   //主要Web发布目录（存放JSP,SERVLET等）
└── work   //工作目录，Tomcat将翻译JSP文件到的Java文件和class文件放在这里
```

#### bin目录主要文件包括：

* `catalina.sh` 用于启动和关闭tomcat服务器；

* `configtest.sh` 用于检查配置文件；

* `startup.sh` 启动Tomcat脚本；

* `shutdown.sh` 关闭Tomcat脚本；

#### conf目录主要的配置文件：

* `server.xml` Tomcat 的全局配置文件；

* `web.xml` 为不同的Tomcat配置的web应用设置缺省值的文件；

* `tomcat-users.xml` Tomcat用户认证的配置文件；

<!--more-->

#### logs目录主要文件包括：

![](https://farm1.staticflickr.com/923/42135003635_2045033be5_o.png)

* `localhost_access_log.2018-06-26.txt` 访问日志

* `localhost.2018-06-26.log` 错误和其它日志

* `manager.2018-06-26.log` 管理日志

* `catalina.2018-06-26.log` Tomcat启动或关闭日志文件


### Tomcat 配置文件

tomcat配置文件目录为 `/usr/local/tomcat/conf/`, 主要包括以下文件：

* `server.xml`  omcat的主配置文件，包含`Service, Connector, Engine, Realm, Valve, Hosts`主组件的相关配置信息；

* `web.xml`  每个webapp只有“部署”后才能被访问，它的部署方式通常由web.xml进行定义，其存放位置为WEB-INF/目录中；

* `tomcat-user.xml`  用户认证的账号和密码文件，在Tomcat中添加/删除用户，为用户指定角色等将通过编辑此文件实现；

* `catalina.policy`  java相关的安全策略配置文件，在系统资源级别上提供访问控制的能力；当使用-security选项启动tomcat时，用于为tomcat设置安全策略；

* `catalina.properties`  Java属性的定义文件，用于设定类加载器路径，以及一些与JVM调优相关参数，Tomcat在启动时会事先读取此文件的相关设置；

* `logging.properties`  Tomcat日志记录器相关的配置信息，可以用来定义日志记录的组件级别以及日志文件的存在位置等；

* `context.xml` 每个web专用的配置文件，包括所有host的默认配置信息，其存放位置为WEB-INF/目录中； 

### tomcat的核心组件

#### `Server` 组件

元素是整个配置文件的根元素,在所有元素最顶层，代表整个Tomcat容器，类似于<html>标签

* `className`: 用于实现此Server容器的完全限定类的名称，默认为`org.apache.catalina.core.StandardServer`；

* `port` 接收shutdown指令的端口，默认仅允许通过本机访问，默认为8005；

* `shutdown` 指定向端口发送的命令字符串；

#### `Service` 组件

Service主要用于关联一个引擎和与此引擎相关的连接器，每个连接器通过一个特定的端口和协议接收入站请求交将其转发至关联的引擎进行处理。

#### `Connector`组件

主要功能接收连接请求，创建Request和Response对象用于和请求端交换数据；然后分配线程让Engine来处理这个请求，并把产生的Request和Response对象传给Engine。

进入tomcat的请求可分为两类：

第一类：standalone，请求来自于客户端浏览器；

第二类：由其它的web server反代，来自前端的反代服务器；

* `nginx –> http connector –> tomcat`

* `httpd(proxy_http_module) –> http connector –> tomcat`

* `httpd(proxy_ajp_module) –> ajp connector –> tomcat`

相关属性：

```
port=”8080″
protocol=”HTTP/1.1″
connectionTimeout=”20000″
address：监听的IP地址；默认为本机所有可用地址；
maxThreads：最大并发连接数，默认为200；
enableLookups：是否启用DNS查询功能；
acceptCount：等待队列的最大长度
```

#### `Engine`组件

servlet引擎，其内部可以一个或多个host组件来定义站点； 通常需要通过`defaultHost`来定义默认的虚拟主机；

#### `Host`组件

Engine的子容器。Engine组件中可以内嵌1个或多个Host组件，每个Host组件代表Engine中的一个虚拟主机。Host组件至少有一个，且其中一个的name必须与Engine组件的defaultHost属性相匹配。

* `name` 指定主机名；

* `appBase` 应用程序基本目录，即存放应用程序的目录；

* `unpackWARs` 如果为true，则tomcat会自动将WAR文件解压，否则不解压，直接从WAR文件中运行应用程序；

### `tomcat` 常用配置

#### 配置Tomcat监听80端口

##### 第一步：修改server.xml配置文件

```
[root@local-linux02 conf]# vi /usr/local/tomcat/conf/server.xml

//Connector port="8080",将端口号改为80，之后保存退出。
```

##### 第二步：重启tomcat

```
//关闭tomcat
[root@local-linux02 conf]# /usr/local/tomcat/bin/shutdown.sh
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk1.8
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar

//再次启动tomcat 
[root@local-linux02 conf]# /usr/local/tomcat/bin/startup.sh
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/jdk1.8
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
Tomcat started.
```
  
##### 校验成果

- 检查端口情况

```
[root@local-linux02 conf]# netstat -lntp | grep java
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      42011/java
tcp6       0      0 :::8009                 :::*                    LISTEN      42011/java
tcp6       0      0 :::80                   :::*                    LISTEN      42011/java
```

- 浏览器测试

![](https://farm2.staticflickr.com/1806/28168971887_3cb7e60d39_o.png)#### 配置Tomcat虚拟主机

同样还是修改配置文件server.xml，其中`<host></host>`字段就是关于虚拟主机的配置，包括以下选项：

`name`定义域名；

`appBase`定义应用的目录，Java的应用通常是一个war的压缩包，你只需要将war的压缩包放到appBase目录下面即可；

`docBase`，这个参数用来定义网站的文件存放路径，默认为`appBase/ROOT`；

**应用示例：利用zrlog部署一个java的应用**

##### 第一步：修改配置文件

```
vi /usr/local/tomcat/conf/server.xml

//修改host字段

<Host name="www.java-test.com" appBase=""
    unpackWARs= "true" autoDeploy="true"
    xmlValidation="false" xmlNamespaceAware="false">
    <Context path="" docBase="/data/wwwroot/java-test.com/" debug="0" reloadable="true" crossContext="true"/>
</Host>
```


##### 第二步：下载zrlog

```
//下载zrlog
[root@local-linux02 src]# wget http://dl.zrlog.com/release/zrlog-1.7.1-baaecb9-release.war  

//移动至webapps 
mv zrlog-1.7.1-baaecb9-release.war /usr/local/tomcat/webapps/

mv /usr/local/tomcat/webapps/zrlog-1.7.1-baaecb9-release /usr/local/tomcat/webapps/zrlog

```

##### 第三步：数据库配置

- 登陆数据库

```
[root@local-linux02 webapps]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 10.2.15-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

创建数据库

```
MariaDB [(none)]> create database zrlog;
Query OK, 1 row affected (0.02 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| db2                |
| information_schema |
| mysql              |
| performance_schema |
| zrlog              |
+--------------------+
5 rows in set (0.08 sec)
```

- 授权用户

```
MariaDB [(none)]> grant all on zrlog.* to 'zrlog'@127.0.0.1 identified by 'steve-123';
Query OK, 0 rows affected (0.05 sec)

MariaDB [(none)]> exit
Bye
```

- 测试是否数据库是否能正常登陆

![](https://farm1.staticflickr.com/927/43044220491_70991797dd_o.png)

##### 第四步：浏览器访问 ip:8080/zrlog ，运行安装程序

- 依次填写服务器地址、数据库名、用户名、密码、端口、系统邮箱

![](https://farm2.staticflickr.com/1767/43044258241_21a9952d0b_o.png)


- 下一步，随便填了，反正是实验环境

![](https://farm1.staticflickr.com/847/41233959100_3abe796e45_o.png)

- 完成

![](https://farm2.staticflickr.com/1810/29172852238_85724d1443_o.png)

- 效果预览

![](https://farm2.staticflickr.com/1783/42142092225_54c31c3782_o.png)

##### 第五步：去掉浏览器访问时输入的目录名

![](https://farm1.staticflickr.com/845/28176355517_4985de7688_o.png)

##### 第六步：重启tomcat

```
[root@local-linux02 webapps]# /usr/local/tomcat/bin/shutdown.sh
[root@local-linux02 webapps]# /usr/local/tomcat/bin/startup.sh 

```

##### 浏览器测试

- 修改本地host文件

![](https://farm2.staticflickr.com/1788/41234146920_6000aaa669_o.png)

- 浏览器测试

![](https://farm1.staticflickr.com/843/29173225698_007bf6578a_o.png)
### Tomcat日志

![](https://farm2.staticflickr.com/1767/29166127608_492f31ac1f_o.png)

其中，

* `catalina`开头的日志为Tomcat的综合日志，它记录Tomcat服务相关信息，也会记录错误日志。`catalina.2018-xx-xx.log`和`catalina.out`内容相同，前者会每天生成一个新的日志；

* `host-manager`和`manager`为管理相关的日志，其中host-manager为虚拟主机的管理日志；

* `localhost`和`localhost_access`为虚拟主机相关日志，其中带access字样的日志为访问日志，不带access字样的为默认虚拟主机的错误日志。

日志文件同样是在/usr/local/tomcat/conf/server.xml 中虚拟主机配置的位置<value>字段，其中：
* `prefix`定义访问日志的前缀;

* `suffix`定义日志的后缀;

* `pattern`定义日志格式；

示例：

```
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
         prefix="123.cn_access" suffix=".log"
         pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```

### tomcat 连接 mysql

tomcat连接mysql是通过自带的JDBC驱动实现的，具体步骤如下：

#### 第一步：配置mysql，创建数据库、表、用户

```
mysql -u root -p

create database java_test;

use java_test;

grant all on java_test.*'java@ip' identified by 'passwd';
```

#### 第二步：修改tomcat配置文件

```
vi content.xml

<Resource name="jdbc/mytest"
    auth="Container"
    type="javax.sql.DataSource"
    maxActive="100" maxIdle="30" maxWait="10000"
    username="java" password="aminglinux"
    driverClassName="com.mysql.jdbc.Driver"
    url="jdbc:mysql://127.0.0.1:3306/java_test">
</Resource>
```

#### 第三步：修改 web.xml配置文件

``` web.xml配置文件

vi /usr/local/tomcat/webapps/ROOT/WEB-INF/web.xml

<resource-ref>
   <description>DB Connection</description>
   <res-ref-name>jdbc/mytest</res-ref-name>
   <res-type>javax.sql.DataSource</res-type>
   <res-auth>Container</res-auth>
</r
```
#### 第四步：编辑测试内容

```
vi /usr/local/tomcat/webapps/ROOT/t.jsp


<%@page import="java.sql.*"%>
<%@page import="javax.sql.DataSource"%>
<%@page import="javax.naming.*"%>
<%
Context ctx = new InitialContext();
DataSource ds = (DataSource) ctx
.lookup("java:comp/env/jdbc/mytest");
Connection conn = ds.getConnection();
Statement state = conn.createStatement();
String sql = "select * from aminglinux";
ResultSet rs = state.executeQuery(sql);
while (rs.next()) {
out.println(rs.getString("id") +"<tr>");
out.println(rs.getString("name") +"<tr><br>");
}
rs.close();
state.close();
conn.close();
%>
```

#### 第五步：重启tomcat

```
/usr/local/tomcat/bin/shutdown.sh /usr/local/tomcat/bin/startup.sh
``` 

## 扩展阅读-  [邱李的tomcat文档](https://www.linuser.com/forum.php?mod=forumdisplay&fid=37) 

-  [JAR、WAR包区别](http://blog.csdn.net/lishehe/article/details/41607725) 

-  [tomcat常见配置汇总](http://blog.sina.com.cn/s/blog_4ab26bdd0100gwpk.html)

-  [resin安装](http://fangniuwa.blog.51cto.com/10209030/1763488/)
  - [tomcat 单机多实例](http://www.ttlsa.com/tomcat/config-multi-tomcat-instance/)- [tomcat的jvm设置和连接数设置](http://www.cnblogs.com/bluestorm/archive/2013/04/23/3037392.html)- [jmx监控tomcat](http://blog.csdn.net/l1028386804/article/details/51547408)- [tomcat内存溢出](https://blog.csdn.net/ye1992/article/details/9344807)

## 参考资料：

- [Tomcat 配置文件详解](https://www.jianshu.com/p/602f051918d9)


