---
title: Struts2-004 漏洞分析
tags: Struts2
typora-root-url: ../
date: 2020-12-24 11:12:58
---

#### 漏洞概要

可参考官方安全公告：https://cwiki.apache.org/confluence/display/WW/S2-004

#### 漏洞分析

攻击者可以使用双重编码的url和相对路径来遍历目录结构并下载“静态”内容文件夹之外的文件。

<!--more-->

根据官方概述，得知漏洞存在的类为 `FilterDispatcher`过滤器，一般在`doFilter`方法中进行操作，我们将`doFilter`进行断点

![image-20201223101612422](/img/Struts2-004-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201223101612422.png)

当`resourcePath`的路径为`/struts`会调用`findStaticResource`方法

![image-20201223102109094](/img/Struts2-004-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201223102109094.png)

访问的静态文件不能以` .class` 结尾，其实这个限制没有什么用，然后遍历配置好的静态文件目录并调用 `findInputStream`![image-20201223103005752](/img/Struts2-004-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201223103005752.png)

![image-20201223102453441](/img/Struts2-004-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201223102453441.png)

将访问的路径跟目录拼接在一起，然后 URL 解码，再调用 `getResourceAsStream `开始读取文件，就造成了目录遍历漏洞。

![image-20201223102649847](/img/Struts2-004-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201223102649847.png)

![image-20201223105441336](/img/Struts2-004-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201223105441336.png)

POC：

```
/struts/..%252f/
/struts/..%252f..%252f..%252fWEB-INF/web.xml
```

#### 修复

加上了 cleanupPath、URL.getFile 和 endWith 来进行限制。

![image-20201224120316800](/img/Struts2-004-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201224120316800.png)

![image-20201224120423548](/img/Struts2-004-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201224120423548.png)

![image-20201224121310332](/img/Struts2-004-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201224121310332.png)

![image-20201223105230910](/img/Struts2-004-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201223105230910.png)

#### 参考

https://aluvion.gitee.io/2020/07/16/struts2%E7%B3%BB%E5%88%97%E6%BC%8F%E6%B4%9E-S2-004/#%E5%89%8D%E8%A8%80

https://xz.aliyun.com/t/7967

https://cwiki.apache.org/confluence/display/WW/S2-004