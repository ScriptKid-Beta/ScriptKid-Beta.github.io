---
title: Telnet 测试邮件协议
tags: mail
typora-root-url: ../
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

##### smtp

![image-20210414093817371](/img/Telnet%20SMTP/image-20210414093817371.png)

<!--more-->

```bash
telnet smtp.163.com 25 # telnet 连接
220 163.com Anti-spam GT for Coremail System (163com[20141201])
helo 163.com #163.com可以随便任意字符；EHLO 是扩展的简单邮件传输协议 (ESMTP) 命令动词，该命令动词是在 RFC 2821 中定义的。ESMTP 服务器可在初始连接时公布其功能。这些功能包括其最大的可接受邮件大小以及其支持的身份验证方法。HELO 是 RFC 821 中定义的旧版 SMTP 命令动词。多数 SMTP 邮件服务器都支持 ESMTP 和 EHLO。
250 OK
auth login	#认证登陆
334 dXNlcm5hbWU6
aXR4aWFvd2VpNzU1	#邮箱用户名base64编码
334 UGFzc3dvcmQ6
VUdFRVhRU0hGS0dBQ1hJRA==	#邮箱密码(或授权码)base64编码
235 Authentication successful
mail from:<itxiaowei755@163.com> #发件人
250 Mail OK
rcpt to:<79898326@qq.com>	#收件人
250 Mail OK
data	#开始写邮件
354 End data with <CR><LF>.<CR><LF>
from:itxiaowei755@163.com	#发件人名称，此项可以任意填入，将显示在收件箱的发件人一栏
to:79898326@qq.com	#收件人名称，可任意填入，将显示在收件箱的收件人一栏。
date:10/10/2021 #发信日期
subject:hello smtp	#邮件主题
					#需空一行表示正文开始
tyachedtothisletter.Imnowwritingtoaskifyoucanwriteareferencefor... #正文内容
.	#.回车 发送邮件
250 Mail OK queued as smtp7,C8CowABnYuAHRnZgpUBPXA--.60300S2 1618364054 #返回250表示发送成功
quit #退出
```

##### pop3

![image-20210413180236367](/img/Telnet%20SMTP/image-20210413180236367.png)

```bash
telnet pop.163.com 110 # telnet连接
user **** # 用户名
pass **** # 密码
+OK 24 message(s) [643260 byte(s)] # 成功会显示OK 24代表24封邮件 643260代表总邮件的字节数

命令列表
stat #查看统计,执行后,POP3服务器会响应一个正确应答,它以“+OK”开头,接着是两个数字,第一个是邮件数目,第二个是邮件的大小
list #格式list [n] 参数n可选，n为邮件编号；查看邮件列表
uidl #格式uidl [n] 参数n可选，n为邮件编号；查看邮件唯一邮件标识码
retr #格式retr [n] 参数n可选，n为邮件编号；查看邮件的内容
dele #格式dele [n] 参数n可选，n为邮件编号；删除指定的邮件(注意:dele n命令只是给邮件做上删除标记,只有在执行quit命令之后,邮件才会真正删除)
top  #格式top [n][m]  参数n m 必选，n为邮件编号，m为行数；读取指定邮件正文的行数,如果m=0,则只读出邮件的邮件头部分
noop #POP3服务器不执行任何操作,仅返回一个正确响应"+OK"
quit #退出
```

##### 错误代码

```
4xx代码：
  421个#4.4.5此时的许多TLS会话
  421个#4.4.5从您的主机的许多连接
  421个#4.4.5对此的许多连接主机
  421个#4.4.5对此监听程序的许多连接
  421个#4.x.2此会话的许多消息
  421不可用<hostname>的服务，关闭处理信道
  421超出了允许的连接时间
  421超过了坏SMTP命令限制，断开
  评估许可证超时的421
  451个#4.3.0服务器错误
  452个#4.3.1全双工的队列
  452种#4.3.1以后服务器资源低的再试一次
  452个#4.3.1临时系统错误(12)
  452个#4.5.3许多收件人
  454 TLS不可用由于一个临时原因

5xx代码：
  500个#5.5.1没被认可的命令
  500太长的线路
  501 #5.0.0 EHLO要求域地址
  501个#5.5.2语法错误XXX
  对验证命令的501个#5.5.4无效参数
  501未知xxx命令
  501未知的选项XXX
  501未知值XXX
  不可用503 #5.3.3的验证
  在邮件处理时没允许的503 #5.5.0验证
  已经验证的503个#5.5.0
  首先503 #5.5.1 MAIL
  首先503 #5.5.1 RCPT
  503 commandsDATA Bad顺序在mailmerge处理内的
  503 commandsXPRT Bad顺序在无格式处理内的
  503接收零件的commandsnow Bad顺序
  503不在mailmerge处理
  504个#5.5.1验证机制XXX不是可用的
  504命令参数XXX无法识别
  504个无效XDFN语法
  504无效部件号
  504无效部件号XXX
  504没有指定的可变值
  仍然未命中其他504的部分
  504保留变量名称
  504在*parts语法的语法错误
  504个XDFN命令不能包含零字符
  530个#5.7.0必须首先发出STARTTLS命令
  530个#5.7.0此发送方必须首先发出STARTTLS命令
  要求的530验证
  538个#5.7.11要求的加密
  552个#5.3.4信息标题大小超过限制
  552个#5.3.4消息大小超过限制
  552超过的大小限制
  554个#5.3.0服务器错误
  554许多跳
  554消息主题包含非法仅有的CR/LF字符。
```