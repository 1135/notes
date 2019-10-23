### webgoat8简介


WebGoat是由OWASP维护的、Java编写的Web应用程序（靶场程序），它被故意设置了各种漏洞，也包含一些简洁课程内容，帮助我们从实践中理解Web安全,学习渗透测试技术。



### WebGoat环境搭建

注意：
运行此程序的机器极易受到攻击，要先做好ACL仅供自己访问，或断开网络连接。

项目地址
https://github.com/WebGoat/WebGoat/releases

如果使用Java 9以下版本 按如下方式运行WebGoat：
```
java -jar webgoat-server-8.0.0.M21.jar  --server.port=8000 --server.address=0.0.0.0


#默认情况下，WebGoat在端口8080上启动，--server.port指定其他端口。
# --server.address是把它绑定到不同的地址（WebGoat的默认配置只绑定到localhost）
```


如果使用Java 9或更高版本，按如下方式运行WebGoat：
```
java --add-modules java.xml.bind -jar webgoat-server-8.0.0.VERSION.jar
```


访问WebGoat:
```
http://ip:8000/WebGoat/login
```

在自己搭建的系统中注册一个账号即可登录使用。


### WebWolf环境搭建


Java 9以下，这样运行WebWolf：
```
java -jar webwolf-8.0.0.M21.jar --server.port=9090 --server.address=0.0.0.0
```


访问:
```
http://10.211.55.4:9090/login
```

账号密码使用WebGoat注册的即可。


3个功能：
1.文件
必要文件上传到这里，可供下载
http://0.0.0.0:9090/files/{username}/{filename}

2.收件箱
在webgoat中发送邮件到：
{user}@{random}
如你的用户名为testtest则发送
testtest@xxx.xxx
都会显示在WebWolf中testtest用户的收件箱。

3.请求记录器
http://webwolf/landing
记录http完整请求


### WriteUp

参考
从最开始的HTTP Basics开始做
http://www.xianxianlabs.com/2018/06/03/webgoat1/

