---
title: 'linux学习笔记12-5：lnmp环境的搭建和配置（五）'
date: 2018-06-10 22:58:32
categories: linux
tags: [linux, LNMP, notes]
---

## 写在前面

继续整理lnmp环境配置的笔记,这则笔记关注的重点是ssl相关知识。

## Nginx的负载均衡

负载均衡，是很多大型网站必备的服务器资源分配策略，从字面上的意思来理解，就是N台服务器分担负载，合理使用资源，获得更好的用户体验。

比如，如果你在运营一个类似京东、淘宝、12306这样的大型网站，假设只有一台服务器，当用户访问量过大，服务器处理不了时就会宕机，用户访问就会很慢甚至卡死。为了避免这种情况，很多大型网站都有很多台服务器，同时来提供服务。

当然，这些服务不可能是完全一致的，有的配置高，有的配置稍微差一点；如果做到既能够充分利用服务器的资源，不浪费；又不会让服务器压力过大呢？这就是均衡负载做的工作。

### nginx均衡负载的实现方法

Nginx负载均衡是通过upstream模块来实现的，内置实现了三种负载策略：

#### 方法一：轮循（默认） 

Nginx根据请求次数，将每个请求均匀分配到每台服务器

示例：

同一个应用有3个实例分别运行在srv1-srv3。当没有特别指定负载均衡方法时， 默认为round-robin/轮询。所有请求被代理到服务器集群myapp1， 然后nginx实现HTTP负载均衡来分发请求。

```
http {
    upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp1;
        }
    }
}
```

<!--more-->

#### 方法二：最少连接 

将请求分配给连接数最少的服务器。Nginx会统计哪些服务器的连接数最少。

示例：

```
upstream myapp1 {
    least_conn;
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}

```

#### 方法三：IP Hash 

利用URI HASH绑定处理请求的服务器。具体而言，第一次请求时，根据该客户端的IP算出一个HASH值，将请求分配到集群中的某一台服务器上。后面该客户端的所有请求，都将通过HASH算法，找到之前处理这台客户端请求的服务器，然后将请求交给它来处理。

```
upstream myapp1 {
    ip_hash;
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}
```

### 基于权重的负载均衡

上述的三种默认策略，并没有制定服务器的权重，这意味着所有列出的服务器被认为是完全平等的。但实际上并非如此，有的机器使用时间久，配置低；有的则是新机器，内存大、配置过，两者的处理能力显然是不同的，所以可以通过使用服务器权重来影响nginx的负载均衡算法。

举例来说：

下列配置中，配置最好的服务器101被赋予了最高权重，被分配的任务最多；其次是100；最后是102；

```
upstream tomcats {
    server 192.168.0.100:8080 weight=2;  // 每6次访问有2次分配到该服务器
    server 192.168.0.101:8080 weight=3;  // 这台机器配置最好，每6次访问有3次分配到该服务器
    server 192.168.0.102:8080 weight=1;  // 每6次访问有1次分配到该服务器
}
```

### 配置中的参见参数

参数|说明
---|---
max_fails |默认为1。某台Server允许请求失败的次数，超过最大次数后，在fail_timeout时间内，新的请求将不会分配给这台机器。如果设置为0，Nginx会将这台Server置为永久无效状态
fail_timeout| 默认为10秒，某台Server请求失败次数达到max_fails后，在fail_timeout期间内，nginx会认为这台Server暂时不可用，不会将请求分配给它
backup|备份机，所有服务器挂了之后才会生效
down|标识某一台server不可用
max_conns|限制分配给某台Server处理的最大连接数量，超过这个数量，将不会分配新的连接给它。默认为0，表示不限制
resolve |将server指令配置的域名，指定域名解析服务器。需要在http模块下配置resolver指令，指定域名解析服务

## nginx配置ssl

### 从http 到 https

#### http的工作原理

HTTP协议，是互联网上应用最广泛的一种网络传输协议，默认使用80端口，主要用来在计算机网络之间进行通信。

**HTTP请求/响应的步骤：**

1. 客户端连接到Web服务器，默认端口为80，建立一个TCP套接字连接；

2. 通过TCP套接字，发送HTTP请求；

3. 服务器接受请求并返回HTTP响应；

4. Web服务器主动关闭TCP套接字，释放TCP连接；客户端被动关闭TCP套接字，释放TCP连接；

5. 客户端浏览器解析HTML内容；

**HTTP协议的缺点：**

由于HTTP协议采用明文传输，所以天生有两大弊端：

* 隐私泄露：因为全都是明文传输；

* 安全隐患：页面劫持、中间人攻击；

为了克服上述弊端，HTTPS应运而生。

#### HTTPS的工作原理

![](https://farm5.staticflickr.com/4765/39421693144_da0a14a079_o.jpg)

* 客户使用HTTPS的URL访问Web服务器，要求与Web服务器建立SSL连接；
　
* Web服务器收到客户端请求后，会将网站的证书信息（证书中包含公钥）传送一份给客户端；
　
* 客户端的浏览器与Web服务器开始协商SSL/TLS连接的安全等级，也就是信息加密的等级；

* 客户端的浏览器根据双方同意的安全等级，建立会话密钥，然后利用网站的公钥将会话密钥加密，并传送给网站；

* Web服务器利用自己的私钥解密出会话密钥；

* Web服务器利用会话密钥加密与客户端之间的通信；

这里涉及到一些加密的概念：

- 对称加密：

形象的解释，就是加密和解密用的是同一把钥匙，采用相同的秘钥进行加密和解密。

![](https://farm2.staticflickr.com/1760/41830975685_74a283ef3a_o.png)

- 非对称加密

与对称加密算法不同的是，非对称加密算法使用的加密密钥和解密密钥是不同的。

![](https://farm2.staticflickr.com/1739/41831013065_1c103f46b9_o.png)

- RSA加密

RSA公钥密码算法是公钥密码算法中的一种，RSA中的加密和解密都可以使用公钥或者私钥，但是用公钥加密的密文只能使用私钥解密，用私钥加密的密文智能使用公钥解密。

![](https://farm2.staticflickr.com/1737/28857243478_db11281ec7_o.png)

#### SSL证书

上述过程看起来已经比较安全了，但还是有可能出现中间人攻击。举例来说，如有有人伪造了服务器的公钥与客户端进行握手，那后面的过程进行看似很“安全”，其实已经失去意义了。

为了进一步提升安全性，就催生了由第三方权威机构颁发的SSL证书。

SSL证书大概是这样的：

![](https://farm2.staticflickr.com/1739/41830859375_c70c7e3a2c_o.png)

操作系统在出厂时会内置这个机构的机构信息和公钥。在实践过程中，A网站得到这张证书后，会在与用户通信的过程中将证书发送给客户端，客户端会检测证书的颁发机构。

如果是受信任的证书机构，应用程序会使用预置的 CA证书的公钥去解密最后的指纹内容和指纹算法，之后与网站发来的证书指纹内容进行比对，如果一致，就可以放心进行下一步的通信过程。

### nginx 配置ssl


#### 第0步：安装openssl

```
//检测是否安装openssl
rpm -qf `which openssl`

//如没有，执行以下命令安装
yum install -y openssl
```

#### 第一步：生成秘钥key

```
//秘钥放在配置文件夹下
cd /usr/local/nginx/conf/

//生成秘钥
openssl rsa -in server.key -out server.key
```

#### 第二步：创建服务器证书的申请文件server.csr

```
openssl req -new -key server.key -out server.csr
```

#### 第三步：创建CA证书

```
penssl req -new -x509 -key server.key -out ca.crt -days 3650
```

#### 第四步：创建自当前日期起有效期为期十年的服务器证书server.crt

```
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey server.key -CAcreateserial -out server.crt
```

**至此，一共生成5个文件，其中,server.crt和server.key就是你的nginx需要的证书文件.**

#### 第五步：配置nginx

```
//修改nginx配置文件

//将ssl_certificate改为server.crt的路径,将ssl_certificate_key改为server.key的路径.

server {

    listen       443;
        server_name  localhost;
        ssl                  on;
        ssl_certificate     /root/Lee/keys/server.crt;#配置证书位置
        ssl_certificate_key  /root/Lee/keys/server.key;#配置秘钥位置
        #ssl_client_certificate ca.crt;#双向认证
        #ssl_verify_client on; #双向认证
 
        ssl_session_timeout  5m;
        ssl_protocols  SSLv2 SSLv3 TLSv1;
        ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
        ssl_prefer_server_ciphers   on;
```

#### 第六步：重新加载配置

```
nginx -t

nginx -s reload
```

#### 第七步：测试成果




## 参考资料

* [nginx负载均衡配置](https://www.jianshu.com/p/ac8956f79206)

* [Nginx负载均衡配置解读](https://blog.csdn.net/xyang81/article/details/51702900)

* [使用nginx实现HTTP负载均衡](https://skyao.gitbooks.io/learning-nginx/content/documentation/HTTP_load_balancer.html)

* [浅析 HTTPS 与 SSL 原理](https://cloud.tencent.com/developer/article/1004382)

* [浅谈HTTPS（SSL/TLS）原理](https://www.jianshu.com/p/41f7ae43e37b)

* [SSL/TLS原理详解](https://segmentfault.com/a/1190000002554673)

* [Nginx 配置 SSL 证书 + 搭建 HTTPS 网站教程](https://www.cnblogs.com/chjbbs/p/5748369.html)

* [如何为nginx配置https(免费证书)](https://www.jianshu.com/p/9523d888cf77)

* [Nginx 配置 SSL 证书 + 搭建 HTTPS 网站](https://segmentfault.com/a/1190000007948986)


