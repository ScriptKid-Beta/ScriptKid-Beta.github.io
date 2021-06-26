---
title: 某SRC任意文件包含漏洞 包含日志Getshell
password: 132456+
abstract: 该文章已加密, 请输入密码查看。
message: 该文章已加密, 请输入密码查看。
wrong_pass_message: 密码不正确，请重新输入！
wrong_hash_message: 文章不能被校验, 不过您还是能看看解密后的内容！
tags: ThinkCMF
typora-root-url: ../
date: 2020-10-20 15:52:58
---



#### 案例

通过子域名收集到以下资产。

```
cp.***.com
cps.***.com
dept.***.com
sdk.***.com
```

发现上述域名均存在ThinkCMF任意文件包含漏洞。

```
http://**.***.com/?a=display&templateFile=/etc/passwd
```

![image-20201020152035870](/img/%E6%9F%90SRC%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E-%E5%8C%85%E5%90%AB%E6%97%A5%E5%BF%97Getshell/image-20201020152035870.png)

```
#包含日志利用条件
1、日志的存储路径：data/runtime/Logs/Portal/20_10_19.log（20_10_19.log 年_月_日.log）或可尝试一些常用的日志默认路径
2、日志可读
```

发现当无法加载控制器时会将报错信息写入log日志中去。

![image-20201020115405685](/img/%E6%9F%90SRC%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E-%E5%8C%85%E5%90%AB%E6%97%A5%E5%BF%97Getshell/image-20201020115405685.png)

![image-20201020115100297](/img/%E6%9F%90SRC%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E-%E5%8C%85%E5%90%AB%E6%97%A5%E5%BF%97Getshell/image-20201020115100297.png)

利用通过记录错误日志写入恶意数据，浏览器url特殊字符自动转换编码，所以需要抓包修改。

![](/img/%E6%9F%90SRC%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E-%E5%8C%85%E5%90%AB%E6%97%A5%E5%BF%97Getshell/image-20201020114711936.png)

已将恶意数据写入log日志中配合文件包含漏洞成功解析。

```
https://**.***.com/?a=display&templateFile=data/runtime/Logs/Portal/20_10_19.log
```

![image-20201020115534944](/img/%E6%9F%90SRC%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E6%BC%8F%E6%B4%9E-%E5%8C%85%E5%90%AB%E6%97%A5%E5%BF%97Getshell/image-20201020115534944.png)



#### 任意文件包含漏洞

##### 漏洞描述

&emsp;&emsp;任意文件包含漏洞（Unrestricted File Inclusion），是一种常见的Web安全漏洞，Web程序在引入文件时，由于传入的文件名没有经过合理的校验，或者检验被绕过，从而操作了预想之外的文件，导致意外的敏感信息泄露，甚至恶意的代码注入，使攻击者获取到网站服务器权限。
&emsp;&emsp;当被包含的文件在服务器本地时，形成本地文件包含漏洞；被包含的文件在第三方服务器时，形成远程文件包含漏洞。

##### 修复方案

1. 关闭危险的文件打开函数。
2. 过滤特殊字符，如：“.”(点)、“/”(斜杠)、“\”(反斜杠)。
3. 使用web检测文件内容。