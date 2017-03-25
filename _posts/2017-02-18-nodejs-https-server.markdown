---
layout: post
title: 通过Node.js搭建HTTPS服务器
date: 2017-02-18
---

最近在开发一个对接其他系统的需求，代码写完了但是要联调只能去类生产环境，实在太麻烦了。
为了能够提前验证功能，避免上类生产环境之后再发现阻塞问题，特意写了一个测试桩来模拟对方的系统。
对方的系统是https协议的，刚开始想用tomcat+java实现，但最后选择了node.js实现。原因如下：

- tomcat+java 实现过程中除了java代码的开发外，还需要开发对应的web.xml、生成https证书、修改tomcat的server.xml等多个配置文件，耗费的时间太多
- tomcat实现的模拟器太厚重，涉及的文件太多，不利于项目组内共享
- 后期要想修改java代码实现的功能，还得将其导入IDE，修改完后还得将class部署到tomcat，不如node.js方便

想要搭建https的服务器，第一步就是安装openssl了，因为要用它产生证书。安装openssl的方法网上有很多，
但是也有人说git自带了openssl，我到C:\Program Files目录（我的软件安装目录）下搜了下，发现git和VMware都带了openssl.exe。
那就用git自带的吧，openssl.exe所在目录的截图如下：
![openssl路径]({{ site.url }}/images/17-2-18/openssl.png)

#### 使用openssl生成证书文件
<pre>
#创建私钥key文件：
C:\Program Files\Git\mingw32\bin>openssl genrsa -out privatekey.pem 1024
Generating RSA private key, 1024 bit long modulus
..................++++++
...................................................++++++
e is 65537 (0x10001)

#通过私钥生成CSR证书签名
C:\Program Files\Git\mingw32\bin>openssl req -new -key privatekey.pem -out certrequest.cs
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
#下面的内容可以根据自己的实际情况输入，输入后回车即可
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Guangdong
Locality Name (eg, city) []:shenzhen
Organization Name (eg, company) [Internet Widgits Pty Ltd]:xxx.com
Organizational Unit Name (eg, section) []:xxx
Common Name (e.g. server FQDN or YOUR name) []:msmf
Email Address []:nsndg@xxx.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

#通过私钥和证书签名生成证书文件
C:\Program Files\Git\mingw32\bin>openssl x509 -req -in certrequest.csr -signkey privateke
e.pem
Signature ok
subject=/C=CN/ST=Guangdong/L=shenzhen/O=xxx.com/OU=xxx/CN=msmf/emailAddress=nsndg@xxx.com
Getting Private key

C:\Program Files\Git\mingw32\bin>
</pre>

#### 将新生成的三个文件烤出到自己的代码目录
- privatekey.pem: 私钥
- certrequest.csr: CSR证书签名
- certificate.pem: 证书文件


#### 使用Node.js编写httpServer.js
具体可以参考http://nodejs.cn/api/https.html
```JavaScript
const https = require('https'); 
const fs = require('fs'); 
 
const options = { 
  key: fs.readFileSync('./cer/privatekey.pem'), 
  cert: fs.readFileSync('./cer/certificate.pem') 
}; 
 
https.createServer(options, (req, res) => { 
  res.writeHead(200); 
  res.end('Hello.'); 
}).listen(443);
```

#### 启动服务器
<pre>
#在代码目录下执行命令
C:\Users\1\Desktop\httpsServer>node httpServer.js
</pre>

#### 通过浏览器访问https://127.0.0.1:443
由于证书是我们自己生成的，不是合法机构颁发的，所以浏览器会有如下提示页面，点击“高级”即可。
![openssl路径]({{ site.url }}/images/17-2-18/first.png)

点击“继续前往127.0.0.1（不安全）”
![openssl路径]({{ site.url }}/images/17-2-18/second.png)
此时我们就可以看到服务器响应了
![openssl路径]({{ site.url }}/images/17-2-18/third.png)

#### Demo下载（其中包含了已经生生成的证书）
 [Demo]({{ site.url }}/download/httpsServer.zip) 
