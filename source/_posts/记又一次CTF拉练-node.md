---
title: 记又一次CTF拉练-Node.js
typora-root-url: ../
---

本文仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担。

<div style="text-align: right"> 小维</div>

#### 前言

这篇文章记述了又一次的CTF拉练，Node.js的白盒审计题...

#### 源码

```javascript
// 捕捉全局异常
process.on('uncaughtException', function (err) {
    console.log('Caught exception: ', err);
});  

////引入模块
var express = require('express') 
var session = require('express-session');
var fs = require('fs');
var path = require('path');
var config = require('./config');
var marked = require('marked');
var morgan = require('morgan');
var bodyParser = require('body-parser');
var AccessControl = require('express-ip-access-control');

var port = process.env.PORT;
var app = express()
var sourceCode;

marked.setOptions({
    highlight: function (code) {
        return require('highlight.js').highlightAuto(code).value
    }
})

// 系统文件读写操作
fs.readFile('app.js', 'utf8', (err, data) => {
    if (!err) {
        markdown = `\`\`\`node\n${data}\n\`\`\``;
        sourceCode = marked(markdown);
    }
});

var options = {
    mode: 'allow',
    denys: [],
    allows: ['10.0.0.6'],
    forceConnectionAddress: false,
    log: function (clientIp, access) {
        if (!access)
            console.log(`${clientIp} denied.`);
    },
    statusCode: 404,
    redirectTo: '',
    message: '404 Not Found...Don\'t fuck me Please......'
};

// 访问控制'express-ip-access-control'
app.use(AccessControl(options));
// 设置views文件夹，应用程序视图目录 ejs后缀
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');
//public下所有文件都会以静态资料文件形式返回（如样式、脚本、图片素材等文件）
app.use(express.static(path.join(__dirname, 'public')));

//session
app.use(session({ secret: 'ctf', resave: true, saveUninitialized: true, name: 'SID' }));

//morgan日志格式化 自定义format名ctf
morgan.format('ctf', '[ctf] [:remote-addr/:req[x-forwarded-for]] - ":method :url HTTP/:http-version" :status :res[content-length] ":referrer" ":user-agent"');
app.use(morgan('ctf'));

//配置body-parser中间件  application/x-www-form-urlencoded，extended: true 任何数据类型 设置数据限制100mb  参数限制1000000
app.use(bodyParser.urlencoded({ limit: '100mb', extended: true, parameterLimit: 1000000 }));

// 网站根路径
app.get('/', function(req, res) {
    // console.log(sourceCode);
    res.render('index', {code: sourceCode});
});

// 中间件处理
app.use(function (req, res, next) {
    // 判断url中是否有/login 或者 session值是否未定义
    if (isProtectUrl(req.originalUrl, req.query) && (typeof req.session['username'] == 'undefined')) {
        res.redirect('/');
    } else {
        next();
    }
    
    // 判断url是否存在有/login
    function isProtectUrl(url, query_url) {
        var isProtectUrl = true;

        var unProtectUrl = [];
        unProtectUrl.push('/login');

        for (var i = 0; i < unProtectUrl.length; i++) {
            if (unProtectUrl[i] == url || (url.indexOf('#') < 0 && url.indexOf(unProtectUrl[i]) >= 0 && JSON.stringify(query_url).indexOf(unProtectUrl[i]) < 0)) {
                isProtectUrl = false;
                break;
            }
        }
        return isProtectUrl
    }
});

// login页面
app.get('/login', function(req, res) {
    res.render('login');
});

// 登录post请求
app.post('/login', function (req, res) {
    var username = req.body['username'];
    var password = req.body['password'];
    if (username == config.username && password == config.password) {
        req.session["username"] = "admin";
        res.send("login success.");
    } else {
        res.send('login failed.');
    }    
});

// getflag页面
app.get('/getflag', function (req, res) {
    // console.log(config);
    res.render('flag', {flag: config.flag}); //加载flag
});

// 侦听
app.listen(port, '0.0.0.0', () => {
    console.log(`ksctf app listening at http://0.0.0.0:${port}`);
})
```

<!--more-->

#### 思路解析

首先分析代码, 先是引入一些模块，定义网站根路径加载源码，中间用isProtectUrl进行判断url中是否有/login及session['username']是否定义来处理请求，不包含/login或者未定义seesion['username']则跳转到网站根路径；定义了一个login页面，可以post验证用户名及密码，如果用户密码对了则session赋值；定义了一个getflag页面，该页面加载flag显示。

题目主要请求访问/getflag来查看flag文件。

#####  初步想法

看是否有什么Cookie伪造之类的达到session赋值从而绕过typeof req.session['username'] == 'undefined')或者能否绕过isProtectUrl(req.originalUrl, req.query) 使两者中其中一个不成立后访问/getflag。

##### 本题考点

**绕过isProtectUrl(url, query_url)；**

req.query 一个对象，为每一个路由中的query string参数都分配一个属性。

1、不能获取原型链的属性

2、如果没有query string，它就是一个空对象，属性的值为{}。

...

#### 解题步骤

根据上述特性我们可以构造成`getflag?__proto__=/login`，`getflag?=/login`等等。

##### 本地环境调试

1、`getflag?=/login`

![image-20210626122959208](/img/%E8%AE%B0%E5%8F%88%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83-node.js/image-20210626122959208.png)

2、`getflag?__proto__=/login`

![image-20210626122805010](/img/%E8%AE%B0%E5%8F%88%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83-node.js/image-20210626122805010.png)

##### CTF环境

请求/getflag 获取flag

```
https://xxx.xxx.com/getflag?__proto__=/login
```

![image-20210625214459295](/img/%E8%AE%B0%E5%8F%88%E4%B8%80%E6%AC%A1CTF%E6%8B%89%E7%BB%83-node.js/image-20210625214459295.png)