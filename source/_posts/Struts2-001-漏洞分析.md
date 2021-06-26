---
title: Struts2-001 漏洞分析
tags: Struts2
typora-root-url: ../
date: 2020-12-24 11:09:58
---

#### 漏洞概要

可参考官方安全公告：https://cwiki.apache.org/confluence/display/WW/S2-001

#### 漏洞分析

在HTTP请求被Struts2处理时，首先读取`web.xml`文件，这个是网站配置文件，里面有个过滤器，叫：`org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter`然后这个过滤器执行完之后，会经过一系列的拦截器，这些拦截器可以是默认的，也是可以用户自定义的。

Struts2请求处理流程（来自攻击JavaWeb应用[5]）：

![image-20201225113205710](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201225113205710.png)

```
这里科普几个概念
拦截器概念
拦截器(Interceptor)是Struts2框架的核心功能之一,Struts 2是一个基于MVC设计模式的开源框架, [3]  主要完成请求参数的解析、将页面表单参数赋给值栈中相应属性、执行功能检验、程序异常调试等工作。Struts2拦截器是一种可插拔策略,实现了面向切面的组件开发,当需要扩展功能时,只需要提供对应拦截器,并将它配置在Struts2容器中即可,如果不需要该功能时,也只需要在配置文件取消该拦截器的设置,整个过程不需要用户添加额外的代码。拦截器中更为重要的概念即拦截器栈(Interceptor Stack),拦截器栈就是Struts2中的拦截器按一定的顺序组成的一个线性链,页面发出请求,访问Action对象或方法时,栈中被设置好的拦截器就会根据堆栈的原理顺序的被调用。 

说人话：struts2是框架，封装的功能都是在拦截器里面，封装很多功能，有很多拦截器，不是每次这些拦截器都执行，每次执行默认的拦截器，默认拦截器位置struts2-core-2.0.8.jar!\struts-default.xml，在执行拦截器，执行过程使用aop思想，在action没有直接调用拦截器方法，而是使用配置文件进行操作，在执行拦截器时候，执行很多的拦截器，这个过程使用责任链模式，例如：执行三个拦截器，执行拦截器1->执行完放行->执行拦截器2->执行完放行->执行拦截器3->执行完放行->执行action方法。


拦截器什么时候执行呢？
在action对象之后，action方法执行之前
```

<!--more-->

例如下图`struts.xml `中的`package` 继承了`struts`默认的拦截器(struts-default)，具体可以查看`struts-default.xml`文件。

![image-20201218102840514](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/simage-20201218102840514.png)

![image-20201218104357386](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218104357386.png)

这里我们要关注`params`这个拦截器，代码位置：`xwork-2.0.3.jar!\com\opensymphony\xwork2\interceptor\ParametersInterceptor.class`

![image-20201218111703265](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218111703265.png)

![image-20201218113626884](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218113626884.png)

经过一系列的拦截器处理后，数据会成功进入实际业务 `Action `。程序会根据` Action` 处理的结果，选择对应的 `JSP`视图进行展示，并对视图中的 `Struts2` 标签进行处理。

在本实例中`Action`处理用户登录是返回`error`

![image-20201218144456659](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218144456659.png)

根据返回结果以及先前在` struts.xml`中定义的视图，程序将开始处理 `index.jsp`

![image-20201218144537239](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218144537239.png)

![image-20201218144810408](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218144810408.png)

从代码里我们可以看得到，`struts2`使用了自定义标签库，也就是`/struts-tags`, 通过阅读 `struts2-core-2.0.8.jar!/META-INF/struts-tags.tld `文件，我们得知这个`textfield`标签实现类是`org.apache.struts2.views.jsp.ui.TextFieldTag`

![image-20201220153156591](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201220153156591.png)

![image-20201220153226282](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201220153226282.png)

了解jsp自定义标签的同学应该知道，这时候我们需要找的是`doStartTag`方法，因为解析标签是从这个方法开始，具体可以参考 [TagSupport详解](https://blog.csdn.net/zljjava/article/details/17420809), 通过在`TextFieldTag`类的`ComponentTagSupport`父类我们找到`doStartTag`方法

当在` JSP` 文件中遇到 `Struts2 `标签 时，由于s2的标签库都是集成与`ComponentTagSupport`类，程序会先调用 `doStartTag` ，并将标签中的属性设置到 `TextFieldTag `对象相应属性中。

![image-20201218154513302](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218154513302.png)

最后，在遇到 `/>`结束标签的时候调用 `doEndTag` 方法。

![image-20201218152120932](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218152120932.png)

```java
public int doEndTag() throws JspException {
        this.component.end(this.pageContext.getOut(), this.getBody());
        this.component = null;
        return 6;
    }
```

我们跟进`this.component.end`方法，该方法调用了 `this.evaluateParams();`方法来填充` JSP `中的动态数据。

![image-20201218152528873](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218152528873.png)

跟进`this.evaluateParams`方法，发现如果开启`OGNL`表达式支持(this.altSyntax())，会进行属性字段添加`OGNL`表达式字符(%{name})

![image-20201218152851979](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218152851979.png)

然后使用` findValue`方法从值栈中获得该表达式所对应的值，跟进`findValue`方法

![image-20201218153309872](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218153309872.png)

`findValue`在开启了`altSyntax`且`toType`为`class.java.lang.string`时调用`TextParseUtil.translateVariables`方法

![image-20201218153519551](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218153519551.png)

跟进该方法

![image-20201218153626893](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218153626893.png)

发现该方法重名加载

![image-20201218153722178](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218153722178.png)

我们传入` translateVariables` 方法的表达式 `expression` 为 `%{password}` ，经过 `OGNL `表达式解析，程序会获得其值 `%{1+1} `(这里就是我们传入的payload)。由于此处使用的是 `while`循环来解析` OGNL` ，所以获得的` %{1+1}`又会被再次循环解析，最终也就造成了任意代码执行。

![image-20201218154234294](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201218154234294.png)

关键代码：

```java
public static Object translateVariables(char open, String expression, ValueStack stack, Class asType, TextParseUtil.ParsedValueEvaluator evaluator) {
        Object result = expression;

        while(true) {
            int start = expression.indexOf(open + "{");
            int length = expression.length();
            int x = start + 2;
            int count = 1;

            while(start != -1 && x < length && count != 0) {
                char c = expression.charAt(x++);
                if (c == '{') {
                    ++count;
                } else if (c == '}') {
                    --count;
                }
            }

            int end = x - 1;
            if (start == -1 || end == -1 || count != 0) {
                return XWorkConverter.getInstance().convertValue(stack.getContext(), result, asType);
            }

            String var = expression.substring(start + 2, end);
            Object o = stack.findValue(var, asType);
            if (evaluator != null) {
                o = evaluator.evaluate(o);
            }

            String left = expression.substring(0, start);
            String right = expression.substring(end + 1);
            if (o != null) {
                if (TextUtils.stringSet(left)) {
                    result = left + o;
                } else {
                    result = o;
                }

                if (TextUtils.stringSet(right)) {
                    result = result + right;
                }

                expression = left + o + right;
            } else {
                result = left + right;
                expression = left + right;
            }
        }
    }
```

因此究其原因，在于在`translateVariables`中，递归解析了表达式，在处理完`%{password}`后将`password`的值直接取出并继续在`while`循环中解析，若用户输入的`password`是恶意的`OGNL`表达式，比如`%{1+1}`，则得以解析执行。

POC：

```
%{1+1}
```

#### 修复

增加了了递归解析的判断

![image-20201224171538096](/img/Struts2-001-%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90/image-20201224171538096.png)

#### 参考

https://xz.aliyun.com/t/7915

https://xz.aliyun.com/t/2044

https://dean2021.github.io/posts/s2-001/

https://cwiki.apache.org/confluence/display/WW/S2-001

