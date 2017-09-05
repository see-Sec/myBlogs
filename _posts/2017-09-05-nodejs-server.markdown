---
layout: post
title:  使用Node.js实现简单的web服务器
date: 2017-09-05
---

由于项目组的前端开发人员较多，经常会出现几个人同时替换一个环境上的文件导致环境不可用的情况，导致大家的开发效率都很低。还有就是一个人做的需求涉及修改的文件太多，去环境上验证的话替换文件太麻烦了。为了替大家解决这个问题（本地搭建tomcat也可以解决，只是我想借此拿Node.js练练手），我用Node.js实现了一个简单的web服务器，它的功能很简单，只是根据请求的URL将对应的本地文件响应给浏览器。具体实现如下：

### 1. 新建app.js文件，内容如下：
```
"use strict";
var http = require('http');
var url = require('url');
var fs = require('fs');
var path = require('path');

//webapp的根路径
const APP_ROOT_PATH = "G:\\github\\myBlogs"; 

var mimes = {
    "css": "text/css",
    "gif": "image/gif",
    "html": "text/html",
    "ico": "image/x-icon",
    "jpeg": "image/jpeg",
    "jpg": "image/jpeg",
    "js": "text/javascript",
    "json": "application/json",
    "pdf": "application/pdf",
    "png": "image/png",
    "svg": "image/svg+xml",
    "swf": "application/x-shockwave-flash",
    "tiff": "image/tiff",
    "txt": "text/plain",
    "wav": "audio/x-wav",
    "wma": "audio/x-ms-wma",
    "wmv": "video/x-ms-wmv",
    "xml": "text/xml"
};

function processreq(req, resp) {
    var reqUrl = req.url;
    var pathName = url.parse(reqUrl).pathname;
    pathName = decodeURI(pathName);

    //获取静态文件的绝对路径
    var filePath = path.resolve(APP_ROOT_PATH + pathName);
	
    //文件的后缀名，返回值包含”.”，通过slice方法来剔除掉”.”
    var ext = path.extname(filePath);
    ext = ext ? ext.slice(1) : 'unknown';

    //匹配不到多媒体类型的文件使用"text/plain"类型
    var contentType = mimes[ext] || "text/plain";
	fs.readFile(filePath, function (err, data) {
        if (err) {
           resp.writeHead(404, { "content-type": "text/html" });
           resp.end("<h1>404 Not Found</h1>");
        }
		else{
		    resp.writeHead(200, { "content-type": contentType });
            resp.end(data,"binary");
		}
   });
}

//创建web服务器，监听8080端口
http.createServer((req, res) => {
    processreq(req, res);
}).listen(8080, function(){
   console.log("You can visit http://127.0.0.1:8080");
});
```

### 2. 在app.js文件所在的路径下执行命令 node app.js
```
G:\github\myBlogs\js>node app.js
You can visit http://127.0.0.1:8080
```

### 3. 通过浏览器访问
可以访问代码中常量APP_ROOT_PATH对应路径下的静态文件，例如 http://127.0.0.1:8080/images/17-2-18/cer.png