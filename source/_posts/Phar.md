---
title: phar://
tags: php
typora-root-url: ../
---

#### php支持的协议和封装协议

```
file://
http://
ftp://
php://
zlib://
data://
glob://
phar://
ssh2://
rar://
ogg://
expect://
```

#### Phar是什么？

PHAR (“Php ARchive”) 是PHP里类似于JAR的一种打包文件，可以方便地将多个文件组合成一个文件。在PHP 5.3 或更高版本中默认支持，这个特性使得 PHP也可以像 Java 一样方便地实现应用程序打包和组件化。一个应用程序可以打成一个 Phar 包，直接放到 PHP-FPM 中运行。可以像执行任何其他文件一样轻松地执行phar归档，无论是在命令行上还是在web服务器上。

<!--more-->

#### phar扩展中三种受支持的文件格式（Phar、Tar、Zip）对比

![image-20200610092038319](/img/Phar/image-20200610092038319.png)

这里主要概述Phar

#### Phar文件结构

All Phar archives contain three to four sections:

1. a stub
2. a manifest describing the contents
3. the file contents
4. [optional] a signature for verifying Phar integrity (phar file format only)

##### a stub

最小的stub如下：

```
<?php __HALT_COMPILER();
```

stub要以*__HALT_COMPILER（）*结尾的要求外，没有任何限制*。*否则phar扩展名将无法处理Phar。

##### a manifest describing the contents

Phar清单是一种高度优化的格式，允许按文件指定文件压缩，文件权限，甚至是用户定义的元数据（例如文件用户或组）。

其中Meta-data部分的信息会以反序列化的形式储存，这里就是**漏洞利用的关键点**

![image-20200610090122258](/img/Phar/image-20200610090122258.png)

##### the file contents

压缩文件的内容

##### [optional] a signature for verifying Phar integrity (phar file format only)

包含签名的Phar总是在加载器，清单和文件内容之后，将签名附加到Phar归档文件的末尾。 目前支持的两种签名格式是MD5和SHA1。

签名格式:

![image-20200610091746077](/img/Phar/image-20200610091746077.png)

#### phar文件生成

```
// 通过以下代码创建一个phar文件
<?php
    class User{
    	public $name="test";
    }
    $phar = new Phar("test.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stubb
    $o = new User();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest里
    $phar->addFromString("test.txt", "test"); // 添加要压缩的文件
    $phar->stopBuffering(); // 签名自动计算
?>
```

如果出现如下的异常提示，将php.ini中的phar.readonly修改为Off

```
Fatal error: Uncaught UnexpectedValueException: creating archive "test.phar" disabled by the php.ini setting phar.readonly in...
```

![](/img/Phar/image-20200610130230606.png)

生成得phar文件，并将以十六进制的形式进行查看，一部分序列化的内容就是上述说的Meta-Data

![](/img/Phar/image-20200610131216585.png)

##### 生成其他格式文件的Phar文件

因为phar文件php是通过其文件头的stub来识别的，更确切点来说是`__HALT_COMPILER(); `这段代码，对前面的内容或者后缀名是没有要求的。那么我们就可以通过添加任意的文件头信息+修改后缀名的方式将phar文件伪装成其他格式的文件。

```
// 通过以下代码创建一个其他格式的phar文件
<?php
    class User{
    	public $name="test";
    }
    $phar = new Phar("gif.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stubb，增加gif文件头
    $o = new User();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest里
    $phar->addFromString("test.txt", "test"); // 添加要压缩的文件
    $phar->stopBuffering(); // 签名自动计算
?>
```

![image-20200610160634614](/img/Phar/image-20200610160634614.png)

#### phar实现反序列化

##### 原理

[phar.c#L618](https://github.com/php/php-src/blob/29b56a878aa22310d645c3266110417e07ebe683/ext/phar/phar.c#L618)其调用了`php_var_unserialize`

![image-20200611102749956](/img/Phar/image-20200611102749956.png)

通俗点讲：使用phra://伪协议读取文件的时候，文件会被解析成phar对象，这个时候，刚才那部分的序列化的信息就会触发反序列化。因此可以构造一个特殊的phar包，使得攻击代码能够被反序列化。

##### Demo1

生成GIF格式文件的Phar文件，并将后缀名更改为gif

```
// 通过以下代码创建一个其他格式的phar文件
<?php
    class User{
    	public $name="test";
    }
    $phar = new Phar("gif.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stubb，增加gif文件头
    $o = new User();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest里
    $phar->addFromString("test.txt", "test"); // 添加要压缩的文件
    $phar->stopBuffering(); // 签名自动计算
?>
```

Test代码：

```
<?php
class User {
    function __destruct()
    {
        echo "Test run";
    }
    }
include('phar://gif.gif');
?>
```

成功反序列化识别文件内容，采用这种方法可以绕过很大一部分上传检测。

这里可以看到已经反序列化成功触发`__destruct`方法并且读取了文件内容。

![image-20200610170419400](/img/Phar/image-20200610170419400.png)





利用phar函数可以在不适用unserialize()函数的情况下触发PHP反序列化漏洞 。

漏洞点在使用phar://协议读取文件时，文件内容会被解析成phar对象，然后phar对象内的Metadata信息会被反序列化。

#### 漏洞利用

##### 可触发反序列化的操作函数

| 可触发反序列化的操作函数 |               |              |                   |
| ------------------------ | ------------- | ------------ | ----------------- |
| fileatime                | filectime     | file_exists  | file_get_contents |
| file_put_contents        | file          | filegroup    | fopen             |
| fileinode                | filemtime     | fileowner    | fileperms         |
| is_dir                   | is_executable | is_file      | is_link           |
| is_readable              | is_writable   | is_writeable | parse_ini_file    |
| copy                     | unlink        | stat         | readfile          |

##### 可触发条件

```
1、可以上传Phar文件到服务端（不支持远程调用）
2、有可以利用的魔术方法
3、文件操作函数的参数可控
4、无过滤: / phar等特殊字符没有被过滤
提示：
如果有限制不可以phar://出现在头几个字符，可以使用Bzip / Gzip协议绕过
Postgres、MySQL也可触发phar造成反序列化
```

##### CTF案例一

[记一次CTF拉练](https://scriptkid-beta.github.io/2020/06/12/记一次CTF拉练/#more)

##### CTF案例二

[SUCTF2019-Upload labs 2](https://github.com/team-su/SUCTF-2019/tree/master/Web/Upload%20Labs%202)

#### 参考

https://paper.seebug.org/680/

https://blog.zsxsoft.com/post/38

https://www.php.net/manual/zh/book.phar.php

[http://www.lmxspace.com](http://www.lmxspace.com/2018/11/07/重新认识反序列化-Phar/#0x01-起源)

[https://www.k0rz3n.com/](https://www.k0rz3n.com/2018/11/19/一篇文章带你深入理解PHP反序列化漏洞/#0X05-利用-phar-拓展-PHP-反序列化的攻击面)