---
title: VMware 安装黑苹果系统(MacOS)
typora-root-url: ../
---

#### 写在前面

经常有时候需要测试MAC的一些软件，所以就想着用VM来安装黑苹果来进行一些软件的测试。

#### 环境介绍

```
操作系统：Windows 10 10.0.18363
VMware版本：VMware® Workstation 15 Pro 15.5.2 build-15785246
```

<!--more-->

#### 安装

##### 准备工作

```
MK-Unlocker-VM15.5.zip（Unlocker补丁，支持 Vmware 15.5.5）
macOS.Catalina.10.15.5.01.LY.iso（MacOS镜像）
下载地址：链接：https://pan.baidu.com/s/1SOUCyZ0Ys7PQcxQpAhKTHg 提取码：down
```

##### VMware虚拟机安装

这里安装VMware虚拟机不在赘述，可自行百度进行下载安装。

##### 解锁

默认的 VMware 是不支持识别苹果系统镜像的，所以需要用到Unlocker工具进行解锁，进行对MK-Unlocker-VM15.5.zip解压，然后以管理员身份运行win-install.cmd脚本

![image-20210126172049200](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126172049200.png)

等待运行完毕即可。

![image-20210126172832152](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126172832152.png)

##### MacOS

打开Vmware虚拟机，新建虚拟机，选择macOS.Catalina.10.15.5.01.LY.iso，选择Apple Mac OS x的选项（未解锁是没有这个选项），之后默认即可，也可自己进行配置，直到完成。

![image-20210126173349851](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126173349851.png)

![image-20210126173627968](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126173627968.png)

完成后不要立即启动，先找到保存虚拟机文件的目录，找到后缀为 .vmx 的文件，进行编辑，在最后一行添加

`smc.version = 0`，保存并退出。

![image-20210126174636509](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126174636509.png)



启动虚拟机

![image-20210126174747413](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126174747413.png)

选择语言，这里选择"简体中文"，点击"箭头"下一步

![image-20210126175117143](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126175117143.png)

选择"磁盘工具"，点击"继续"进行对磁盘分区，注意分区安装系统的磁盘需要大于25GB以上。

![image-20210126175242992](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126175242992.png)

![image-20210126180200605](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126180200605.png)

![image-20210126180544001](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126180544001.png)



![image-20210126185615418](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126185615418.png)

点击"退出磁盘工具"

![image-20210126182728141](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126182728141.png)

选择"安装macOS",点击"继续"

![image-20210126181159749](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126181159749.png)

![image-20210126183123501](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126183123501.png)

安装完成后，弹出欢迎使用和设置界面，接下来就是一些设置，设置完即可进入苹果系统了

![image-20210126194401893](/img/VMware-%E5%AE%89%E8%A3%85%E9%BB%91%E8%8B%B9%E6%9E%9C%E7%B3%BB%E7%BB%9F(MacOS)/image-20210126194401893.png)

#### 参考

https://www.cnblogs.com/deshun/p/10652385.html