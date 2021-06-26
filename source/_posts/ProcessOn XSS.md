---
title: ProcessOn XSS
tags: XSS
typora-root-url: ../
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

#### 起因

因想跟团队的小伙伴分享一下当时学习XSS是画的流程图，导出时突然弹出了一个框，无意中发现网站存在Self-XSS...

<!--more-->

![image-20200510102108198](/img/Processon-XSS/image-20200510102108198.png)

#### ProcessOn

ProcessOn 隶属于北京大麦地信息技术有限公司，是一款专业在线作图工具和分享社区。它支持流程图、思维导图、原型图、网络拓扑图和UML等多种类型的绘制。

#### 复盘

当文本中插入恶意字符，如：```<script>alert('test')</script>```时，下载文件时会触发js语句从而导致弹框。

![image-20200510102150613](/img/Processon-XSS/image-20200510102150613.png)



![image-20200510102234719](/img/Processon-XSS/image-20200510102234719.png)

文件标签处（发布需要审核可以盲打~）：

![image-20200510102520451](/img/Processon-XSS/image-20200510102520451.png)

这里就不在进一步尝试进行是否可以组合利用扩大影响的测试了，已将相关信息反馈ProcessOn人员。

![image-20200510105215881](/img/Processon-XSS/image-20200510105215881.png)

#### 写到最后

请遵守《网络安全法》等相关法律法规。

