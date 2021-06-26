---
title: Struts2-002 漏洞分析
tags: Struts2
typora-root-url: ../
date: 2020-12-24 11:10:58
---

#### 漏洞概要

可参考官方安全公告：https://cwiki.apache.org/confluence/display/WW/S2-002

#### 漏洞分析

通过官网的安全公告，我们大概知道问题是出在标签`<s:url>`和 `<s:a>`标签

中，如下是我们的`index.jsp`代码

```jsp
<%@ taglib prefix="s" uri="/struts-tags" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <meta http-equiv="content-type" content="text/html; charset=UTF-8">
    <title>s2-002</title>
  </head>
  <body>
  <h2>s2-002 demo</h2>
  <p>link: <a href="https://cwiki.apache.org/confluence/dispaly/WW/S2-002">
  </a>https://cwiki.apache.org/confluence/dispaly/WW/S2-002 </p>
  <s:url action="login" includeParams="all" ></s:url>
  </body>
</html>

```

<!--more-->

由于s2的标签库都是集成与`ComponentTagSupport`类，` doStartTag`方法也是在该类里实现，所以我们直接从`ComponentTagSupport`类`doStartTag`方法进行断点调试, 首先我们看一下`doStartTag`方法：

![image-20201220160944540](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201220160944540.png)

![image-20201220161002678](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201220161002678.png)

由于我们这里处理的是 `s:url` 标签，所以这里用来处理标签的组件 `this.component`为`org.apache.struts2.components.URL`类对象。我们跟进 `URL:start()`方法。

![image-20201221141410686](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221141410686.png)

在 `URL:start() `方法中，我们看到当` includeParams=all`时，会调用 `mergeRequestParameters`方法。

![image-20201221141714877](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221141714877.png)

在 `mergeRequestParameters`方法中，程序会将 `this.req.getParameterMap()`获得的键值对数据存入 `this.parameters`属性。

`getParameterMap()`返回一个map类型的request参数

```
http://192.168.174.1:8888/Struts2_demo_war_exploded/?<script>alert(1)</script>
```

那么解析后的map就是 ： `key= <script>alert(1)</script>、vaule = ""` 并未看到对参数进行任何过滤，

getParameterMap()方法并不会对数据进行任何处理。[可见下文demo实例](#demo)

![image-20201221142411460](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221142411460.png)



![image-20201221142719076](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221142719076.png)

![image-20201221142946058](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221142946058.png)

最后进入`doEndTag`方法进行处理

![image-20201221170346052](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221170346052.png)

![image-20201221170317717](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221170317717.png)

`determineActionURL`方法中调用了` URLHelper`类处理 `this.parameters` 数据并进行返回

![image-20201221153821932](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221153821932.png)

![image-20201221153301168](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221153301168.png)

将其写入，导致XSS漏洞。

![image-20201221153500644](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221153500644.png)

![image-20201222113854119](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201222113854119.png)

`includeParams=get `时并不能触发 XSS 漏洞。

主要原因在于：当` includeParams=all `时，会多执行一个` mergeRequestParameters` 方法，而该方法会将 `this.req.getParameterMap()`数据设置到` this.parameters` 。如果 `includeParams=get `，那么 `this.parameters `中的数据，仅是来自 `this.req.getQueryString()`

![image-20201222113001419](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201222113001419.png)

![image-20201222113440372](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201222113440372.png)

而 `this.req.getParameterMap() `获得的数据会主动进行` URLdecode` ，但是` this.req.getQueryString() `不会。所以 `includeParams=get `时，返回的数据是被 `URLencode` 过的，因此不能触发 XSS 漏洞。[可见下文demo实例](#demo)

#### demo实例

<span id="demo">demo实例</span>

```java
package com.test;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Iterator;
import java.util.Map;

@WebServlet("/test")
public class Hello extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("getQueryString:"+"\n"+request.getQueryString());
        Map<String, String[]> parameterMap = request.getParameterMap();
        Iterator<Map.Entry<String, String[]>> iterator = parameterMap.entrySet().iterator();
        while (iterator.hasNext()){
            Map.Entry<String, String[]> next = iterator.next();
            System.out.println("getParameterMap:"+"\n"+"key="+next.getKey()+'\n'+"value="+next.getValue()[0]);
        }
    }
}

```

![image-20201221180716399](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201221180716399.png)

Poc:

```js
?<script>alert(1)</script>
```



#### 修复

根据公告，我们需要升级到Struts 2.0.11.1版本，未真正修复，仅仅是对script标签进行替换，仍然可以对其进行绕过

![image-20201222111554160](/img/Struts2-002-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201222111554160.png)

bypass POC:

```
?<script 1>alert(1)</script>
?<strong>script</strong>
...

```

#### 参考

https://cwiki.apache.org/confluence/display/WW/S2-002

https://xz.aliyun.com/t/7916

https://dean2021.github.io/posts/s2-002/