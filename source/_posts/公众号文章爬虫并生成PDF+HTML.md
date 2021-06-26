---
title: 公众号文章爬虫并生成PDF+HTML
tags: 爬虫
typora-root-url: ../
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

### 写在前面

因某些原因会导致一些公众号的文章会给发布者删除，所以就想着把公众号的文章进行爬虫到本地进行留档。

<!--more-->

### 爬虫方法

目前我了解的爬取微信公众号文章的方法主要是四种：

1、第三方的公众号聚合网站（如：weixin.sogou.com）；

2、网页版微信（现已无法限制登陆了）；

3、微信程序访问公众号文章的接口；

4、公众号平台引用文章接口（需要申请公众号）。

这里我们采用公众号平台引用文章接口的方式去实现。

### 通过公众号平台引用文章接口爬虫思路

#### 引用文章接口

```
登陆公众号>>首页>>新建群发>>自建图文>>超链接
```

![image-20200404122132481](/img/%E5%85%AC%E4%BC%97%E5%8F%B7%E6%96%87%E7%AB%A0%E7%88%AC%E8%99%AB%E5%B9%B6%E7%94%9F%E6%88%90PDF+HTML/image-20200404122132481.png)



通过网络请求，我们可以看到我们需要的一些数据（文章链接、标题等），如图：

![image-20200404122534217](/img/%E5%85%AC%E4%BC%97%E5%8F%B7%E6%96%87%E7%AB%A0%E7%88%AC%E8%99%AB%E5%B9%B6%E7%94%9F%E6%88%90PDF+HTML/image-20200404122534217.png)

#### 流程设计

![image-20200404191340454](/img/%E5%85%AC%E4%BC%97%E5%8F%B7%E6%96%87%E7%AB%A0%E7%88%AC%E8%99%AB%E5%B9%B6%E7%94%9F%E6%88%90PDF+HTML/image-20200404191340454.png)

#### 程序

这里就不做过程的一些分析了，所以直接贴EXE的可执行程序了（代码写的太垃圾了，就不放源码了）。

GitHub地址：https://github.com/ScriptKid-Beta/Wechat_Articles_Spider

##### 使用

安装wkhtmltopdf：

下载wkhtmltopdf：https://wkhtmltopdf.org/downloads.html  

![image-20200404180406270](/img/%E5%85%AC%E4%BC%97%E5%8F%B7%E6%96%87%E7%AB%A0%E7%88%AC%E8%99%AB%E5%B9%B6%E7%94%9F%E6%88%90PDF+HTML/image-20200404180406270.png)

将下载好的wkhtmltox-0.12.5-1.mxe-cross-win64 文件解压缩并放到跟程序同一个目录，如图：

![image-20200404184015589](/img/%E5%85%AC%E4%BC%97%E5%8F%B7%E6%96%87%E7%AB%A0%E7%88%AC%E8%99%AB%E5%B9%B6%E7%94%9F%E6%88%90PDF+HTML/image-20200404184015589.png)

安装chromedriver：

```
#查看版本信息
浏览器中输入：chrome://version/ 
```

chromedriver的版本一定要与Chrome的版本一致，不然可能就不起作用。

下载链接：

https://npm.taobao.org/mirrors/chromedriver/

http://chromedriver.storage.googleapis.com/index.html

下载好放在跟程序同一目录下即可。

![image-20200404183023403](/img/%E5%85%AC%E4%BC%97%E5%8F%B7%E6%96%87%E7%AB%A0%E7%88%AC%E8%99%AB%E5%B9%B6%E7%94%9F%E6%88%90PDF+HTML/image-20200404183023403.png)

双击打开wechat_pdf.exe程序运行。

![image-20200404191021202](/img/%E5%85%AC%E4%BC%97%E5%8F%B7%E6%96%87%E7%AB%A0%E7%88%AC%E8%99%AB%E5%B9%B6%E7%94%9F%E6%88%90PDF+HTML/image-20200404191021202.png)

##### 效果

![image-20200404190937280](/img/%E5%85%AC%E4%BC%97%E5%8F%B7%E6%96%87%E7%AB%A0%E7%88%AC%E8%99%AB%E5%B9%B6%E7%94%9F%E6%88%90PDF+HTML/image-20200404190937280.png)

### 参考文章

https://mp.weixin.qq.com/s/67sk-uKz9Ct4niT-f4u1KA?

https://mp.weixin.qq.com/s?__biz=MzAxMDM4MTA2MA==&mid=2455304609&idx=1&sn=b7496563aab42e92060bd68936bc4212&chksm=8cfd6bcabb8ae2dc606b060fecf3f837177e3ef22a05a30ee28ebefd75c6677b29df3e426692&token=2137480545&lang=zh_CN#rd