### 简介

跨站请求伪造Cross-Site Request Forgery(CSRF)，也被称为one click attack/session riding

### 原理

"借刀杀人"

CSRF通过构造get/post等请求，设法使已登录用户victim不知情的情况下，设法让victim携带自身Cookie发出"非其主观意愿的"HTTP请求

* CSRF局限性:如果只有单一的CSRF漏洞，攻击者无法得到该HTTP请求的内容(Cookie可用但无法查看)，也无法得到任何回显(HTTP响应等)
* CSRF扩展性:可联合其他漏洞(如XSS)进行攻击效果最大化

### CSRF类型与利用方式

* CSRF类型
  * GET - 利用方式 使victim点击构造的url 发出GET请求 `http://get.csrf.com/payto?name=hacker&moneynumb=100`
  * POST - 利用方式 尝试转变为GET (可尝试将post请求体改成get形式放至url参数中 如果可以正常响应 则可以使用GET请求进行csrf攻击)
* 漏洞联合 - 使用XSS绕过CSRF保护机制 无交互地利用CSRF漏洞
   * 1.利用自身域名的XSS漏洞绕过CSRF防御机制 - 有的anti-CSRF机制为后端判断CSRFtoken的值，使用JavaScript找到CSRFtoken参数值并构造出"合法的"GET/POST请求 全程不存在跨域问题
   * 2.利用自身/兄弟/父子域名的XSS漏洞绕过CSRF防御机制 - 有的anti-CSRF机制是后端通过判断Referer的值，如果Referer的值 是自身/兄弟/父子域名下的url 就是"合法"请求
   * 3.利用任意第三方域名的XSS漏洞 - 使victim访问攻击者可控的第三方网站`https://3.com` 该第三方站点上用javascript构造GET请求(通过img标签 使victim发出GET请求) POST请求(构造目标站的表单数据并自动提交 使victim发出POST请求)
   * html注入漏洞 (参考通过XSS间接利用CSRF)
* 漏洞联合 - 通过已有的CSRF漏洞 利用self-XSS漏洞(变废为宝)
  * 利用过程 - 事实上self-XSS漏洞无法直接使对方触发，然而通过已有的CSRF漏洞构造"触发该self-XSS漏洞的"请求，对方触发CSRF漏洞即触发XSS漏洞

#### 利用任意第三方域名的XSS漏洞 发出GET请求

通过javascript 在 http://www.3.com/demo 中注入以下html代码:
```
<img src="http://get.csrf.com/payto?name=hacker&moneynumb=100">
```

victim访问http://www.3.com/demo 则发出GET请求


get.csrf.com收到GET请求:
请求中的Referer头说明了该get请求是从第三方url发起的 (如果后端验证Referer的值 则CSRF利用失败)
```
GET /payto?name=hacker&moneynumb=100 HTTP/1.1
Host: get.csrf.com
Connection: keep-alive
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36
Accept: image/webp,image/apng,image/*,*/*;q=0.8
Referer: http://www.3.com/demo
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
```
       
#### 利用任意第三方域名的XSS漏洞 发出POST请求

通过javascript 在 https://3.com 中注入以下html代码:
```
  <script>history.pushState('', '', '/')</script>
    <form name="frm" action="https://post.csrf.com/search" method="POST">
      <input type="hidden" name="ip" value="1&#46;1&#46;1&#46;1" />
      <input type="hidden" name="offset" value="0" />
      <input type="hidden" name="limit" value="20" />
      <input type="submit" value="Submit request" />
    </form>
<script>document.frm.submit()</script>
```

victim访问https://3.com 则发出POST请求


post.csrf.com收到POST请求:并可看出HTTP请求从哪里发起
```
Origin: https://3.com
Referer: https://3.com/
```

```
POST /search HTTP/1.1
Host: post.csrf.com
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
* 二次验证 - 安全但影响用户体验(适用于仅对敏感功能处进行防御)
  * CAPTCHA 增加验证码机制
  * 再次输入密码
  * 手机验证码
* Cookie安全 - 后端Set-Cookie使用SameSite属性 (由Google引入的用于缓解CSRF攻击的cookie属性 其值为Strict或Lax)
  * 如`Set-Cookie: JSESSIONID=xxx; SameSite=Strict`则从其他站对目标站web后端发起的http请求(CSRF) 不会携带`JSESSIONID`这一Cookie字段
* (不推荐)通过`Referer/Origin`校验来源域名
  * 缺陷1.只能防御从"不被信任"的域发起的伪造的http请求（如果 **父、子、兄弟域名** 或 **CORS中被信任的域名** 有XSS漏洞 配合构造一个伪造请求 此时referer和Origin的值都是被信任的域 此时“校验来源域名”无法防御）
  * 缺陷2.正常业务如果有302跳转 不携带Origin

参考OWASP [Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.md)
