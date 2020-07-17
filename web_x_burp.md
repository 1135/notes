### burpsuite代理HSTS请求(macOS)

* 解决问题：burpsuite无法拦截某些https的请求，浏览器报错信息为 `ERR_SSL_VERSION_OR_CIPHER_MISMATCH.协议不受支持。`
  * 0.关闭浏览器的HSTS保护
    * firefox 访问 `about:config` 找到选项`network.stricttransportsecurity.preloadlist` 改为`false`
    * chrome 访问 `chrome://net-internals/#hsts`  `Delete domain security policies` 可以删除某个域名
  * 1.burp设置 `project options`选项卡 - `SSL`选项卡
    * SSL协商设置(`SSL negotiation`) 将`Use the default protocols and ciphers of your Java Installation`改为`Use custom protocols and ciphers`
  * 2.操作系统导入burpsuite的证书
    * 访问 http://burp/cert 下载证书,打开macOS的钥匙串访问`keychain` 导入证书并且修改`PortSwigger CA`为始终信任

### proxy

  * User options - connections - SOCKS proxy
    * TCP Level - This setting is applied at the TCP level, and all outbound requests will be sent via this proxy.
    * DNS - If the option "Do DNS lookups over SOCKS proxy" is enabled, then all domain names will be resolved by the proxy. No local lookups will be performed.


具体使用有3种情况
* 1 如果只使用了SOCKS proxy 没有使用Upstream proxy servers
  * 所有请求都走SOCKS proxy
* 2 如果使用了SOCKS proxy  同时使用了Upstream proxy servers 则根据Upstream proxy servers的规则 有2种流向
  * 使用了SOCKS proxy 则通过SOCKS proxy 再根据Upstream proxy servers的规则 发送到规则中配置的那个upstream proxy server
  * 使用了SOCKS proxy 则通过SOCKS proxy 发送请求
* 3 如果没有使用SOCKS proxy 只使用了Upstream proxy servers
  * 根据Upstream proxy servers的规则 发送到规则中配置的upstream proxy server
  * 直接发送请求

### Burpsuite - Macros

Brupsuite的Macros(宏)是一个预先定义好的HTTP请求序列，这个序列中可以包含一个或多个HTTP请求。

在Burpsuite的会话管理规则`Session Handling Rules`中使用Macros实现自动化

* 作用1 利用宏实现自动登录 保证cookie的有效性
  * 利用宏实现无验证码时的自动登录 从而保证cookie的有效性 所以后续扫描请求会正常执行
  * 可保证Intruder多次发送请求的有效性
* 作用2 提取某些值
  * 发送 HTTP请求1 获取 响应1 中的某些值(如CSRF Token)，把它作为 HTTP请求2 中的参数

参考 [Burpsuite中宏的使用 - FreeBuf](https://www.freebuf.com/articles/web/156735.html)


### burpsuite扩展 - Turbo Intruder

快速发送构造的HTTP请求到单个目标IP.

灵活性高: HTTP请求中payload的构造由python代码完成(使用字典等外部资源、各种编码等).

* 版本
  * 插件版 - BAPP store安装
  * 独立版 - https://github.com/PortSwigger/turbo-intruder/releases

### burpsuite扩展 - bypassWAF

BApp安装bypassWAF后，

在burp的选项卡`Project options` - `Session Handling Rules` - `Rules Actions` - `Add`-`Invoke a Burp Extension` 选择`bypassWAF`

在`scope`里指定：工具范围 url范围 参数范围

这里只需要在`URL scope`里增加一个`Any`即可

### burpsuite扩展 - 检测nginx alias错误配置

alias traversal via NGINX misconfiguration

[PortSwigger/nginx-alias-traversal](https://github.com/portswigger/nginx-alias-traversal)

### burpsuite扩展 - NoPE Proxy

安装：

[summitt/Burp-Non-HTTP-Extension: Non-HTTP Protocol Extension (NoPE) Proxy and DNS for Burp Suite.](https://github.com/summitt/Burp-Non-HTTP-Extension)

使用参考：

[Burpsuite抓取非HTTP流量 - FreeBuf](https://www.freebuf.com/articles/network/158589.html)


### Burpsuite - Basic auth 暴力枚举

[brute force - basic auth](https://securityonline.info/use-burp-suite-brute-force-http-basic-authentication/)

* Payload Sets
  * Payload Type: `Custom iterator`
* Payload Options
  * position 1: `admin`
  * Separator for position 1: `:`
  * position 2: `passlist`
* Payload Processing
  * encode - `Base64-encode`
* Payload Encoding
  * 取消打勾 (对`=`等符号不进行URL编码)

### burpsuite扩展 - JOSEPH

JSON Web Token Attacker (JOSEPH)
 
BApp安装
