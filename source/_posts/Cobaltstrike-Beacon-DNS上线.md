---
title: Cobaltstrike Beacon DNS上线
tags: Cobaltstrike 
typora-root-url: ../
date: 2020-12-02 10:41:58
---

#### 写在前面

因为最近捣鼓了一下Cobaltstrike DNS上线，发现网上文章大多数千篇一律(复制粘贴)，形成很多误导。

#### 环境介绍

```
域名平台：阿里云
CobaltStrike版本：4.1
```

#### 环境配置

##### 域名配置

<!-- more -->

```
添加一条A记录指向服务端地址，然后添加两条（可一条）NS记录指向A记录
```

![image-20201201125358965](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201125358965.png)

##### Listener 配置

```
Name: 自定义
Payload: Beacon DNS
DNS Hosts: 域名的NS记录(一个以上)
DNS Hosts(Stager): DNS Hosts的中的一个（只有一个的情况就写一样的）
DNS Port(Bind): 空
```

![image-20201201144725118](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201144725118.png)

##### 环境检测

```
nslookup ns记录
默认情况下看是否返回0.0.0.0，返回则表示成功。（可通过profile来更改的，其进行流量隐藏等，具体可见参考）
```

![image-20201201130948405](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201130948405.png)

#### 生成木马

这里生成一种进行演示，可自行尝试其他方式。

![image-20201201132840569](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201132840569.png)

![image-20201201133135684](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201133135684.png)

#### 上线

目标靶机执行生成的木马文件

![image-20201201133647578](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201133647578.png)

```
出现小黑电脑右键->Interact
输入命令
checkin #强制回连
注：很多文章需输入mode dns-txt （默认就为dns-txt模式）
根据官方文档描述，CS4中有三种数据传输模式，A、AAAA、TXT，默认是TXT
```

显示蓝色电脑如图则成功上线

![image-20201201133927679](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201133927679.png)

执行whoami

![image-20201201135019135](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201135019135.png)

#### 免杀

这里演示一下加壳免杀过某绒。

免杀前，生成的文件直接给某绒自动处理了。

![image-20201201143243313](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201143243313.png)

进行VMP加壳处理

![image-20201201143432147](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201143432147.png)



进行检测

![image-20201201144944327](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201144944327.png)

测试

![image-20201201154931556](/img/Cobaltstrike-Beacon-DNS%E4%B8%8A%E7%BA%BF/image-20201201154931556.png)

#### 参考

https://xz.aliyun.com/t/7938

https://www.nctry.com/1655.html

https://choge.top/2020/08/16/Cobaltstrike%E4%B9%8B%E6%B5%81%E9%87%8F%E9%9A%90%E8%97%8F/