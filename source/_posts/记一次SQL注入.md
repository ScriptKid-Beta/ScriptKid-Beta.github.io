---

title: 记一次SQL注入
password: 132456
abstract: 该文章已受密码保护, 请您输入密码查看。
message: 该文章已受密码保护, 请您输入密码查看。
wrong_pass_message: 密码不正确，请重新输入！
wrong_hash_message: 文章不能被校验, 不过您还是能看看解密后的内容！
typora-root-url: ../
tags: SQLi

---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

#### 写在前面

某一次授权测试下发现一个SQL注入，手法还是很基础（SQL注入的基本操作），第一次遇到这种所以这里进行记录一下。

#### 案例

##### 正常流程

某综合管理系统，统计报表处，点击查询，发现先进行了POST传递时间范围值，再进行GET请求的POST请求中的sessionID值进行返回。

![image-20200422153809986](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1SQL%E6%B3%A8%E5%85%A5/image-20200422153809986.png)

![image-20200422154142638](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1SQL%E6%B3%A8%E5%85%A5/image-20200422154142638.png)

![image-20200422154325216](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1SQL%E6%B3%A8%E5%85%A5/image-20200422154325216.png)

#### SQL注入测试

发现逻辑是先进行POST传值后进行GET请求返回结果，进行Burp抓包，进行Url解码后加入单引号('')，发送POST请求后，我们根据sessionID进行GET请求查看返回结果。

![image-20200422160115866](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1SQL%E6%B3%A8%E5%85%A5/image-20200422160115866.png)

GET请求返回出错页面并提示ORA-00933，可以得知是Oracle数据库。

![image-20200422160330258](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1SQL%E6%B3%A8%E5%85%A5/image-20200422160330258.png)

再进行添加一个单引号(')进行POST传输后再GET请求后，返回正常页面。

![image-20200422160701586](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1SQL%E6%B3%A8%E5%85%A5/image-20200422160701586.png)

![image-20200422160826015](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1SQL%E6%B3%A8%E5%85%A5/image-20200422160826015.png)

接下来我们就进入常规的SQL注入手法，利用UTL_INADDR.get_host_address函数进行基于错误回显进行数据库版本查询。

````
'and 1=utl_inaddr.get_host_name((select banner from sys.v_$version where rownum=1))-- 
````

![image-20200422161143485](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1SQL%E6%B3%A8%E5%85%A5/image-20200422161143485.png)

![image-20200422161228762](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1SQL%E6%B3%A8%E5%85%A5/image-20200422161228762.png)

#### 写到最后

由于业务系统较为重要这里不再深入进行，以上测试过程纯属本人杜撰，请遵守《网络安全法》等相关法律法规。