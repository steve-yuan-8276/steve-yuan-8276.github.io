---
title: 'linux学习笔记：如何安装使用ftp'
date: 2018-06-12 16:25:27
categories: linux
tags: [linux, ftp, notes]
---

## 写在前面

这则笔记主要整理如何在linux上安装使用ftp。

## 第一步：安装ftp

```
//安装vsftpd
yum install -y vsftpd

//启动服务
systemctl start vsftpd.service

//校验进程是否启动
ps -aux | grep vsftpd

```

如下图所示，说明已启动

![](https://farm1.staticflickr.com/875/40938717090_0438c2032c_o.png)

<!--more-->


## 第二步：修改 vsftpd 配置

vcftpd 的配置文件地址为/etc/vsftpd/vsftpd.conf

- 如果不知道配置文件在那里，使用一下命令查询

```
find / -name 'vsftpd.conf
```

![](https://farm1.staticflickr.com/888/27896822517_4fe15639cd_o.png)

- 查看已有的配置

```
// grep -v参数表示取反，也就是显示不加#的行
[root@steve ~]# cat /etc/vsftpd/vsftpd.conf | grep -v '#'
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
chroot_local_user=YES
listen=NO
listen_ipv6=YES

pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES

```

- 修改配置文件

```
vi /etc/vsftpd/vsftpd.conf

// 基本配置
# 开启匿名登录
anonymous_enable=YES
# 允许使用本地帐户进行FTP用户登录验证
local_enable=YES
# 允许写
write_enable=YES
# 设置本地用户默认文件掩码022
local_umask=022
# 允许匿名上传
anon_upload_enable=YES
# 允许匿名创建新目录
anon_mkdir_write_enable=YES
# 同时开放其它权限
anon_other_write_enable=YES
# 可以发送消息当访问某个目录时
dirmessage_enable=YES
# 开启上传下载记录
xferlog_enable=YES
# 数据链通过20端口建立
connect_from_port_20=YES

# 允许其它用户上传匿名文件
#chown_uploads=YES
# 所有用户
#chown_username=whoever
# 日志保存到
#xferlog_file=/var/log/xferlog
# 日志标准输出
xferlog_std_format=YES
# 空闲会话时间
#idle_session_timeout=600
# 数据连接超时时间
#data_connection_timeout=120
# 隔离的安全用户
#nopriv_user=ftpsecure
# 开启异步数据线程
#async_abor_enable=YES
# 开启ASCII协议上传
ascii_upload_enable=YES
# 开启ASCII协议下载
ascii_download_enable=YES
# 开启邮箱验证
#deny_email_enable=YES
# 拒绝的邮箱列表
#banned_email_file=/etc/vsftpd/banned_emails

# 是否允许直接获取子目录信息
#ls_recurse_enable=YES
# 监听IPv4
listen=NO
# 监听IPv6和监听IPv4
listen_ipv6=YES

# 虚拟用户启用pam认证
pam_service_name=vsftpd
# 用户组管理
userlist_enable=YES
# 访问控制
tcp_wrappers=YES

# 允使用被动模式
pasv_enable=YES
# 指定使用被动模式时打开端口的最小值
pasv_min_port=10060
# 指定使用被动模式时打开端口的最大值。
pasv_max_port=10090
# 用户宽带限制200kps
#local_max_rate=200000
# 登录后欢迎内容
ftpd_banner=Welcome to My FTP service.

# ---------开启虚拟用户组参数--------
# 开启虚拟用户
guest_enable=YES
# 主虚拟用户名vsftpd，等下会建立
guest_username=vsftpd
# 虚拟用户配置（可以对每一个虚拟用户进行单独的权限配置）
user_config_dir=/etc/vsftpd/vconf

# 启用限定用户在其主目录下
chroot_local_user=YES
# 开启用户列表chroot管理
chroot_list_enable=YES
# chroot管理的用户列表（一行一用户,虚拟用户都要添加进去）
# 当设置用户只能在登录目录时，chroot管理的用户为不受限制，否则相反
chroot_list_file=/etc/vsftpd/chroot_list
# 允许chroot管理用户进行写操作
allow_writeable_chroot=YES

# ---------虚拟用户高级参数（请选择一组）--------
# 虚拟用户和本地用户有相同的权限
virtual_use_local_privs=YES

# 虚拟用户和匿名用户有相同的权限，默认是NO
virtual_use_local_privs=NO

# 虚拟用户具有写权限（上传、下载、删除、重命名）
virtual_use_local_privs=YES
write_enable=YES

# 虚拟用户不能浏览目录，只能上传文件，无其他权限
virtual_use_local_privs=NO
write_enable=YES
anon_world_readable_only=YES
anon_upload_enable=YES

# 虚拟用户只能下载文件，无其他权限
virtual_use_local_privs=NO
write_enable=YES
anon_world_readable_only=NO
anon_upload_enable=NO

# 虚拟用户只能上传和下载文件，无其他权限
virtual_use_local_privs=NO
write_enable=YES
anon_world_readable_only=NO
anon_upload_enable=YES

# 虚拟用户只能下载文件和创建文件夹，无其他权限
virtual_use_local_privs=NO
write_enable=YES
anon_world_readable_only=NO
anon_mkdir_write_enable=YES

# 虚拟用户只能下载、删除和重命名文件，无其他权限
virtual_use_local_privs=NO
write_enable=YES
anon_world_readable_only=NO
anon_other_write_enable=YES

```

## 第三步：虚拟用户的建立和配置

```
//假如宿主用户为vsftpd
useradd -s /sbin/nologin vsftpd

//编辑虚拟用户名单文件：
vim /etc/vsftpd/virtusers

//第一行账号，第二行密码，注意：不能使用root做用户名，系统保留
test
123456


//添加到chroot管理列表
vim /etc/vsftpd/chroot_list
//用户列表
test

//安装db工具包
yum install-y compat-db47.x86_64

//利用db_load命令生成数据文件
db_load -T -t hash -f /etc/vsftpd/virtusers /etc/vsftpd/virtusers.db

//更改数据库文件权限
chmod 600 /etc/vsftpd/virtusers.db


//添加vsftpd的虚拟用户的验证
vim /etc/pam.d/vsftpd
-------------内容如下-----------------
#%PAM-1.0
auth sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/virtusers
account sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/virtusers
session    optional     pam_keyinit.so    force revoke
auth       required pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed
auth       required pam_shells.so
auth       include  password-auth
account    include  password-auth
session    required     pam_loginuid.so
session    include  password-auth

```

## 第四步：创建虚拟用户个人配置文件

```
// 建立虚拟用户个人vsftp的配置文件
mkdir -p /etc/vsftpd/vconf

// 进入目录
cd /etc/vsftpd/vconf

// 创建并编辑用户的配置文件
vim test

// 用户 test 配置目录
local_root=/home/vsftpd/test

// 允许本地用户对FTP服务器文件具有写权限
write_enable=YES
anon_world_readable_only=NO

// 允许匿名用户上传文件(须将全局的write_enable=YES,默认YES)
anon_upload_enable=YES

// 允许匿名用户创建目录
anon_mkdir_write_enable=YES

// 允许匿名用户删除和重命名权限(自行添加)
anon_other_write_enable=YES

// 创建用户目录
mkdir -p /home/vsftpd/test

// 权限设置
chown -R vsftpd:vsftpd /home/vsftpd

```

## 第五步：重启ftp

```
// 设置开机自启
systemctl enable vsftpd
// 服务操作
systemctl restart vsftpd.service  # 重启服务
systemctl start vsftpd.service    # 启动服务
systemctl status vsftpd.service   # 服务状态查看

```


## 其它注意事项：

### 修改阿里云服务的安全策略

打开阿里云服务器控制台 https://home.console.aliyun.com，点“我的资源”下面的云服务器进入你的实例列表

选择右边的更多–安全组配置–添加规则–添加安全组规则  规则配置如下：

![](https://farm2.staticflickr.com/1734/41865905765_02d6ea3364_o.jpg)

### 设置主动连接模式

下载ftp 软件，我用的是`yummy ftp pro`

阿里云不支持ftp被动模式，需采用主动模式进行连接。

## 参考资料

* [阿里云下Centos 7简单配置FTP服务器+root账户管理（通用Linux）](https://www.xiaoweigod.com/network/963.html)

* [在CentOS7.4中使用Vsftpd搭建FTP服务器](https://blog.csdn.net/qq_32786873/article/details/78730303)

* [CentOS 7.2 安装和配置 FTP 服务器](https://blog.csdn.net/aerchi/article/details/72820065)

* [CentOS 7 安装FTP服务器](https://blog.csdn.net/kxwinxp/article/details/78595044)

* [CentOS7安装配置vsftp搭建FTP](https://segmentfault.com/a/1190000008161400)

