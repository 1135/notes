### WEB应用-安全部署架构

WAF的基础架构：串联(会话链路)、旁路(无法阻断)

串联
```
高防IP（DDoS防护） ->  CDN（静态资源加速） -> Web应用防火墙（中间层，应用层防护） -> 源站（ECS/SLB/VPC/IDC…）
```

缺少任何产品时顺序不变 如:

缺少WAF时的架构为：`高防IP -> CDN -> ECS`

缺少CDN时的架构为：`高防IP -> WAF -> ECS`

**具体配置步骤**:
* 1、配置Web应用防火墙，在源站IP中填写SLB公网IP、ECS公网IP或本地服务器IP，并在"是否已使用了高防、CDN、云加速等代理？" 选项中选择是。
  * 该步骤得到了 WAF提供的`CNAME`
* 2、配置CDN（如果未启用CDN则跳过此步骤）将在CDN中配置的源站，改为WAF提供的`CNAME`
  * 该步骤得到了 CDN提供的`CNAME`
* 3、配置DDoS高防IP，在源站域名中填入CDN提供的CNAME，或者（未启用CDN，高防IP直接回源至WAF情况下）Web应用防火墙提供的CNAME。
  * 该步骤得到了 高防IP服务提供的`CNAME`
* 4、修改DNS解析的配置，将域名解析指向 高防IP服务提供的`CNAME`
  * 配置完成

### 基础概念

WAF:[web application firewall](https://en.wikipedia.org/wiki/Web_application_firewall)

接入WAF后，WAF代理了公网所有的客户端请求(过滤访问流量)，将过滤后的访问流量导向源站业务端口

所以源站业务端口收到的访问流量都是来自WAF的IP(而真实的客户端IP地址在HTTP请求头部的X-Forwarded-For字段中)

其中"WAF的源IP"就是回源IP地址(Return Source IP Address)

在源站上正确配置 "回源IP防护" 以避免攻击者直接攻击源站IP 策略如下:

1.允许WAF的IP段访问源站的业务端口(80/443)
```
规则方向：入方向
授权策略：允许
协议类型：TCP
授权类型：地址段访问
端口范围：80/443
授权对象：所有Web应用防火墙回源IP段
优先级：1
```

2.禁止公网IP访问源站的业务端口(80/443)
```
规则方向：入方向
授权策略：拒绝
协议类型：TCP
端口范围：80/443
授权类型：地址段访问
授权对象：0.0.0.0/0
优先级：100
```

此时则只有先经过WAF，从WAF出口的流量才可以访问到源站的业务端口。

### WAF/CDN 根本绕过方式

#### WAF/CDN 根本绕过方式1 - 查找源站IP

* 前置条件:针对目标必须是 开启了WAF但没有正确配置"回源IP防护"的网站 找到Origin IP(behind WAF)发起请求,流量不经过WAF/CDN等防护
  * 方式1 查找相关域名的A记录
    * DNS历史记录(DNS history records) - 查看该域名DNS解析的历史记录 主要是A记录 (可能是该站最开始没上WAF/CDN时的IP) 参考工具[vincentcox/bypass-firewalls-by-DNS-history](https://github.com/vincentcox/bypass-firewalls-by-DNS-history)
    * 子域名的A记录
    * 该公司其他域名的A记录
  * 方式2 其他服务 - 与该域名的其他服务(非web)进行交互
    * 邮件服务 - 获取到源站IP(设法让目标站发送邮件，如向一个不存在的地址aCLa21lc@domain.com发送邮件，通常会收到失败反馈邮件，下载邮件的.eml源文件，找到其中`Return-Path`字段中的IP地址、子域名)
    * 尝试通过IP访问目标 命令 `curl -k -H "Host: sub.domain.com" https://109.234.165.77`
  * 方式3 查询全网数据 - 利用"网络空间引擎"查询页面特征。并搜索特征 比如同一网页标题(title)、图标[favicon.ico](https://www.google.cn/favicon.ico)、robots.txt等
* 修复方案: 在源站上正确设置"回源IP防护" 以避免攻击者直接攻击源站IP 策略如下
  * 1.允许WAF的IP段访问源站的业务端口(80/443)
  * 2.禁止公网IP访问源站的业务端口(80/443)

#### WAF 根本绕过方式2 - 使用WAF无法识别的TLS加密算法

* 前置条件 - WAF和WebServer都配有SSL证书(这种情况不常见)
* 流量经过 - `requests` -> `WAF(配有SSL证书解密流量,解不开则放行)` -> `WebServer(配有SSL证书解密流量)`
* 绕过原理 - TLS Client(浏览器/curl)在TLS握手第一步时可主动选择SL/TLS ciphers"加密算法",如果找到WAF不支持的加密算法(WAF放行)且WebServer配有SSL证书支持解密该加密算法,则请求可绕过WAF直达WebServer

参考[LandGrey/abuse-ssl-bypass-waf](https://github.com/LandGrey/abuse-ssl-bypass-waf)

#### WAF 根本绕过方式3 - 利用WAF的"超时放行"特性

* 前置条件 - WAF开启了"超时放行"特性
* 流量经过 - `requests` -> `WAF(WAF开启了"超时放行"特性,压力大时会直接放行一部分请求)` -> `WebServer`
* 绕过原理 - WAF开启了"超时放行"特性:在流量过大等场景下会导致WAF压力大，为了避免因为拦截每一个请求导致响应超时影响业务可用性，会有一部分请求不会被WAF判断，直接放行到webServer
* 实际测试 - 测试某云WAF发现，并发发送多个完全相同的请求(携带payload)，有约1/3的请求被云WAF放过，并成功触发了XSS

### WAF - 绕过规则

* 1.识别WAF名称
  * 各家WAF研究 [Awesome-WAF: 🔥 Everything awesome about web-application firewalls (WAF).](https://github.com/0xInfection/Awesome-WAF)
  * [Ekultek/WhatWaf: Detect and bypass web application firewalls and protection systems](https://github.com/Ekultek/WhatWaf)
* 2.绕过指定WAF的规则 - 实际案例[evasion-techniques](https://github.com/0xInfection/Awesome-WAF#evasion-techniques)
  * 分块传输
  * `multipart/form-data`
  * Unicode Compatibility(利用Unicode的兼容性)
  * payload变形
  * ...


#### 常见WAF

* Appliance - 硬件设备WAF
  * Barracuda Networks WAF
  * Citrix Netscaler Application Firewall
  * F5 Big-IP ASM
  * Fortinet FortiWeb
  * Imperva SecureSphere
  * Penta Security WAPPLES
  * Radware AppWall
  * Sophos XG Firewall
* Cloud - 云WAF
  * Alibaba Cloud
  * Amazon Web Services AWS WAF
  * Cloudbric
  * Cloudflare
  * F5 Silverline
  * Fastly
  * Imperva Incapsula
  * Radware
* Open-source WAF
  * ModSecurity
  

#### 案例1 **payload变形**

某JavaWeb网站存在漏洞(有WAF防护)，如何绕过WAF成功利用Exp?
```
//有明显特征的请求 会被WAF拦截. 比如下面这行是payload字符串 因为有Exp特征 所以被WAF拦截:
System.out.println("1135");

//使用Unicode编码对payload字符串进行转换 "去除"了特征字符串 可绕过WAF规则的检测:
\u0053\u0079\u0073\u0074\u0065\u006d\u002e\u006f\u0075\u0074\u002e\u0070\u0072\u0069\u006e\u0074\u006c\u006e("\u0031\u0031\u0033\u0035");
```

#### 案例2 **Unicode Compatibility**

参考[WAF Bypassing with Unicode Compatibility](https://jlajara.gitlab.io/posts/2020/02/19/Bypass_WAF_Unicode.html)
* BypassWAF的前提: web后端使用了Unicode的兼容性.使用"规范化格式"`NFKC`或`NFKD`,将"不规范字符组成的payload字符串" 转换为 "规范字符组成的payload字符串".
* BypassWAF的原理: 
  * 1.攻击者可以构造"不规范字符组成的payload字符串",先绕过了WAF的规则检测.(如果后端不做任何处理的话 payload无效 无法触发)
  * 2.web后端 将 **"不规范字符组成的payload字符串" 进行"规范化格式"`NFKC`或`NFKD`处理，得到"规范字符组成的payload字符串"**.

* Unicode有4种标准的"规范化格式":
  * NFC: Normalization Form Canonical Composition
  * NFD: Normalization Form Canonical Decomposition
  * NFKC: Normalization Form Compatibility Composition
  * NFKD: Normalization Form Compatibility Decomposition

4个"规范化格式"中有`NFKC`与`NFKD`这2个会对数据做"兼容"(compatibility),即`不规范字符 -> 规范字符`,如代码:
```
# -*- coding: UTF-8 -*-
# python 3

import unicodedata
string = "𝕃ⅇ𝙤𝓃ⅈ𝔰𝔥𝙖𝓃"
print ('NFC: ' + unicodedata.normalize('NFC', string))
print ('NFD: ' + unicodedata.normalize('NFD', string))
print ('NFKC: ' + unicodedata.normalize('NFKC', string))
print ('NFKD: ' + unicodedata.normalize('NFKD', string))
```
输出为
```
NFC: 𝕃ⅇ𝙤𝓃ⅈ𝔰𝔥𝙖𝓃
NFD: 𝕃ⅇ𝙤𝓃ⅈ𝔰𝔥𝙖𝓃
NFKC: Leonishan
NFKD: Leonishan
```

启动一个web服务
```
# 部分代码

# 1.先经过WAF 假设WAF规则中的黑名单字符为
blacklist = ["~","!","@","#","$","%","^","&","*","(",")","_","_","+","=","{","}","]","[","|","\",",".","/","?",";",":",""",""","<",">"]

# 2.再到web服务后端 注意:这里使用了规范化格式"NFKD"【将数据转换为规范化格式】
# 将"不规范字符组成的payload字符串" 转换为 "规范字符组成的payload字符串"
unicodedata.normalize('NFKD', name) #NFC, NFKC, NFD, and NFKD
```


测试
```
# 直接输入普通payload
<img src=p onerror='prompt(1)'>
# 因为其中有黑名单字符 被WAF规则匹配到 并 block
```

BypassWAF
利用 https://www.compart.com/en/unicode 搜索字符 找到对应的"不规范字符"
```
# 构造出了"替代字符串"
＜img src⁼p onerror⁼＇prompt⁽1⁾＇﹥

# 绕过过程
# 1.流量经过WAF - 因为"不规范字符组成的payload字符串"不常见  WAF的规则常常不会匹配到.


# 2.流量到web服务后端
# 注意:这里使用了规范化格式"NFKD"【将数据转换为规范化格式】
# 将"不规范字符组成的payload字符串" 转换为 "规范字符组成的payload字符串"

<img src=p onerror='prompt(1)'>

# payload有效 可触发
```
