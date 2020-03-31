### 简介

WebGoat是由OWASP维护的、Java编写的Web应用程序（靶场程序），它被故意设置了各种漏洞，也包含一些简洁课程内容，帮助我们从实践中理解Web安全,学习渗透测试技术。

https://github.com/WebGoat/WebGoat/

### WebGoat环境搭建

(用Docker可能更方便) 

使用独立的jar包搭建靶机环境的过程:

注意:运行了WebGoat的机器极易受到攻击，要提前做好ACL仅供测试者访问.

启动web服务:
```
java -jar webgoat-server-8.0.0.M21.jar  --server.port=8000 --server.address=0.0.0.0

#默认情况下，WebGoat在端口8080上启动，--server.port指定其他端口。
# --server.address是把它绑定到不同的地址（WebGoat的默认配置只绑定到localhost）
```

访问WebGoat:
```
http://ip:8000/WebGoat/login
```

注册一个账号即可登录使用。

### WebWolf环境搭建

启动web服务:
```
java -jar webwolf-8.0.0.M21.jar --server.port=9090 --server.address=0.0.0.0
```

访问:
```
http://10.211.55.4:9090/login
```

直接用WebGoat的账号密码登录即可。

* WebWolf的3个功能:
  * 1.文件上传.  有助于测试SSRF等漏洞 `http://webwolf.com/files/{username}/{filename}`
  * 2.收件箱.
  * 3.请求记录器.  `http://webwolf/landing` 记录完整的HTTP请求

### WriteUp

```
git clone https://gitlab.com/BlackSheepSpicy/WebGoat.git
```
