---
title: 蓝凌OA Getshell 0day
password: 132456+1
abstract: 该文章已受密码保护, 请您输入密码查看。
message: 该文章已受密码保护, 请您输入密码查看。
wrong_pass_message: 密码不正确，请重新输入！
wrong_hash_message: 文章不能被校验, 不过您还是能看看解密后的内容！
---

```
# 蓝凌OA存在任意用户登录后台上传getshell
# 枚举存在用户，任意用户密码重置
http://100.69.181.14:8080/sys/organization/sys_org_retrieve_password/validateUser.jsp
http://100.69.181.14:8080/sys/organization/sys_org_retrieve_password/sysOrgRetrievePassword.do?method=saveNewPwd
# 上传文件
http://100.69.181.14:8080/sys/profile/portal/uploadLoginTemplate.jsp
```

<!--more-->

来自”蓝凌OA任意用户登录后台上传getshell零日漏洞攻击事件分析报告“

jqIbNM30@

{% pdf https://scriptkid-beta.github.io/pdf/蓝凌OA.pdf%}



