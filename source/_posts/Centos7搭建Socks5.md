---
title: Centos7搭建Socks5
tags: Socks
typora-root-url: ../
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

### 环境介绍

```
操作系统：Centos7
管理工具：Xshell
```

<!--more-->

### Socks5

####  Socks5安装

```
#安装依赖项
yum install gcc openldap-devel pam-devel openssl-devel -y
```

```
wget http://nchc.dl.sourceforge.net/project/ss5/ss5/3.8.9-8/ss5-3.8.9-8.tar.gz # 下载安装包
tar -xzvf ss5-3.8.9-8.tar.gz # 解压缩
cd ss5-3.8.9 # 切换目录
./configure && make && make install # 编译安装
```

![image-20200404214505383](/img/Centos7%E6%90%AD%E5%BB%BASocks5/image-20200404214505383.png)

#### Socks5配置

##### 修改配置文件

```
vim /etc/opt/ss5/ss5.conf
```

```
auth 0.0.0.0/0 – -
改为
auth 0.0.0.0/0 – u
```

```
permit – 0.0.0.0/0 – 0.0.0.0/0 - - - - -
改为
permit u 0.0.0.0/0 – 0.0.0.0/0 - - - - -
```

![image-20200404192600090](/img/Centos7%E6%90%AD%E5%BB%BASocks5/image-20200404192600090.png)

##### 添加用户

```
vim /etc/opt/ss5/ss5.passwd
格式：[用户名] [密码] #一个用户一行
```

![image-20200404192352676](/img/Centos7%E6%90%AD%E5%BB%BASocks5/image-20200404192352676.png)

##### Socks5 启动

```
service ss5 start
```

##### 查看状态

```
service ss5 stutas # 服务状态
netstat -lntp  | grep ss5 #网络连接 
注：安全策略需要放行对应的端口策略
```

![image-20200404192302483](/img/Centos7%E6%90%AD%E5%BB%BASocks5/image-20200404192302483.png)

#### 客户端连接验证

```
curl --socks5 xx.xx.xx.xx:1080 --proxy-user user:password http://ipinfo.io/ip
```

![image-20200404214334247](/img/Centos7%E6%90%AD%E5%BB%BASocks5/image-20200404214334247.png)