---
title: 帆软报表（FineReport） privilege.xml管理密码解密
tags: FineReport
typora-root-url: ../
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

#### 什么是FineReport

FineReport 是帆软软件有限公司自主研发的一款企业级 web 报表软件产品，它“专业、简捷、灵活”，仅需简单的拖拽操作便可以设计出复杂的中国式报表、参数查询报表、填报表、驾驶舱等，轻松搭建数据决策分析系统。

<!--more-->

#### 报表主要配置文件

```
config.xml文件
保存管理平台和服务器配置中的一些信息

datasource.xml文件
保存的是报表数据连接的一些信息。

fsconfig.xml文件
保存的是op=fs数据决策系统平台属性设置等信息（这是在jar包在20170717之前的老版本里），20170717之后的新版本里，权限信息存储在finedb了。

functions.xml文件
保存的是一些自定义函数的设置。

privilege.xml文件
保存的是管理员账号密码以及管理平台(op=fs)上的权限配置信息（注意，仅仅是保存设置信息，比如是否开启了多级权限及模板权限，不存储具体的权限明细内容）。

widgets.xml文件
保存的是控件管理中定义的预定义控件与自定义控件的信息。

chartPreStyle.xml文件
保存的是图表预定义样式中的信息。

map.xml文件
保存的是内置地图和自定义地图的配置信息，包括地图对应的图片，以二进制形式保存在map.xml中。
注：这个是位图地图的文件

web.xml文件
保存的是Web工程下面的一些信息。
```

#### 一个小实例

##### 利用场景

任意文件读取

![image-20200428145059548](/img/%E5%B8%86%E8%BD%AF%E6%8A%A5%E8%A1%A8%EF%BC%88FineReport%EF%BC%89privilege.xml%E7%AE%A1%E7%90%86%E5%AF%86%E7%A0%81%E8%A7%A3%E5%AF%86/image-20200428145059548.png)

##### privilege.xml文件

文件位于\WEB-INF\resources\privilege.xml，该文件保存的是管理员账号密码以及管理平台上的权限配置信息，并使用了硬编码的方式，在Version7.0以上管理密码是进行了加密的，但解密函数已经内置在jar包里了，在这种情况下拿到加密字符串等同于拿到了密码。

![image-20200428104018863](/img/%E5%B8%86%E8%BD%AF%E6%8A%A5%E8%A1%A8%EF%BC%88FineReport%EF%BC%89privilege.xml%E7%A1%AC%E7%BC%96%E7%A0%81%E5%AF%86%E7%A0%81%E8%A7%A3%E5%AF%86/image-20200428104018863.png)

##### JAR包里内置的解密

代码位于com.fr.stable.CodeUtils中

![image-20200428105150473](/img/%E5%B8%86%E8%BD%AF%E6%8A%A5%E8%A1%A8%EF%BC%88FineReport%EF%BC%89privilege.xml%E7%A1%AC%E7%BC%96%E7%A0%81%E5%AF%86%E7%A0%81%E8%A7%A3%E5%AF%86/image-20200428105150473.png)

##### 解密脚本

根据jar里的内置方法进行编写解密脚本，以下提供python和java两种解密方式：

Python：

```Python
#coding:utf-8

def Decode(Key):
    PassWordArray = [19,78,10,15,100,213,43,23]
    print ('密文：',Key)
    if (Key != None and Key.startswith("___")):
        Key = Key[3:]
        Text = []
        Step = 0
        for i in range(0,(len(Key) - 4) + 1,4):
            if (Step == len(PassWordArray)):
                Step = 0
            str = Key[i:i+4]
            num = int(str,16) ^ PassWordArray[Step]
            Text.append(chr(num))
            Step+=1
    Text = ''.join(Text)
    print ('解密为：',Text)

                
if __name__ == "__main__":
    #密文：___0072002a00670066000a  >> admin
    Decode('___0072002a00670066000a')
```

Java：

```java
public class decode {
    public static void main(final String[] args) {
		final int[] PassWordArray = { 19, 78, 10, 15, 100, 213, 43, 23};
        String Key = "___0072002a00670066000a"; // 密文
        if (Key != null && Key.startsWith("___")) {
            Key = Key.substring(3);
            final StringBuilder stringBuilder = new StringBuilder();
            byte Step = 0;
            for (byte i = 0; i <= Key.length() - 4; i += 4) {
                if (Step == PassWordArray.length) {
                    Step = 0;
                }
                final String str = Key.substring(i, i + 4);
                final int num = Integer.parseInt(str, 16) ^ PassWordArray[Step];
                stringBuilder.append((char)num);
                Step++;
            }
            Key = stringBuilder.toString();
        }
        System.out.println(Key);
    }
}
```

##### 效果

![image-20200428114003117](/img/%E5%B8%86%E8%BD%AF%E6%8A%A5%E8%A1%A8%EF%BC%88FineReport%EF%BC%89privilege.xml%E7%A1%AC%E7%BC%96%E7%A0%81%E5%AF%86%E7%A0%81%E8%A7%A3%E5%AF%86/image-20200428114003117.png)



#### 参考

https://help.finereport.com/finereport9.0/doc-view-833.html