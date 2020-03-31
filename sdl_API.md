### 设计与需求阶段

#### Authentication

身份认证 设计原则:使用成熟且安全的组件 不要造轮子

* 使用标准的认证协议(如 [JWT](https://jwt.io/), [OAuth](https://oauth.net/)). 不要使用Basic Auth
* 使用标准的 Authentication, token generating, password storing
* 在登录中使用 Max Retry 和自动封禁功能
* 加密所有的敏感数据

#### JWT

介绍[JSON Web Token](https://jwt.io/introduction/)

* JWT由3部分组成 这3部分用2个`.`连接成一个JWT:
  * Header
  * Payload
  * Signature

```
JWT的格式:
Base64Urlencode(Header).Base64Urlencode(Payload).Signature

比如这个JWT:
eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE1ODY1MjUyNzcsImFkbWluIjoiZmFsc2UiLCJ1c2VyIjoiSmVycnkifQ.BWbSmWbTfsJBc5YMaKCXY4SlvxPZXuobf4vfAFJEXu00qC5nXeyA7csmC7PErf5YoxmbDzFVPobnzhndFe10xQ

分解为3部分:
Header:    eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9 --Base64Urldecode-> {"alg":"HS512","typ":"JWT"}
Payload:   eyJpYXQiOjE1ODY1MjUyNzcsImFkbWluIjoiZmFsc2UiLCJ1c2VyIjoiSmVycnkifQ --Base64Urldecode-> {"iat":1586525277,"admin":"false","user":"Jerry"}
Signature: BWbSmWbTfsJBc5YMaKCXY4SlvxPZXuobf4vfAFJEXu00qC5nXeyA7csmC7PErf5YoxmbDzFVPobnzhndFe10xQ

--------
Header
Header通常包含2个内容:
1.ALGORITHM - 用到的签名算法(signing algorithm)
2.TOKEN TYPE - token的类型

如:
{
  "alg": "HS512",
  "typ": "JWT"
}

--------
Payload
Payload包含了"声明"(claims):有3种claims
Registered claims - 一组非强制性但建议的预定义声明 如 iss (issuer), exp (expiration time), sub (subject), aud (audience) ...
Public claims
Private claims

如:
{
  "iat": 1586525277,
  "admin": "false",
  "user": "Jerry"
}

--------

Signature
Signature的创建需要获取3个参数:
1.encoded header
2.encoded payload
3.a secret - 我设置它为`secret_secret_2020_0331`

算法已经在header中指定.


如:
使用HMACSHA512算法创建JWT的Signature的计算过程

HMACSHA512(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
) 

签名的作用:用于验证消息未被更改. 并且如果使用"私钥"(private key)进行签名,则该签名可用于验证JWT的发送者(是否为它所说的发送者).
```

JWT安全设计原则
* 使用随机的复杂的密钥`JWT Secret` 这样暴力枚举的难度极大
* 服务端不要从header中提取算法(algorithm) 在服务端强制使用指定的高强度算法 如`HS256`,`RS256`...
* token过期时间 设置尽量短.  token expiration (`TTL`, `RTTL`)
* 不要在JWT的"payload部分"中存储敏感数据 因为对"payload部分"进行base64Urldecode就能看到明文. [在线decode](https://jwt.io/#debugger-io)

#### OAuth

API访问授权标准 安全且开放

* 后端必须始终验证`redirect_uri` 且只允许白名单中的URL
* 始终尝试交换code 绝不能交换token (不允许 `response_type=token`)
* 使用 `state` 参数并填充随机的hash 以防御OAuth authentication过程中的CSRF(跨站请求伪造)
* 对不同的应用 分别定义每个应用的 `default scope`参数(默认的作用域) 和 `validate scope`参数(有效的作用域)

#### Access

访问

* 限制requests(Throttling) 以避免DDoS和暴力枚举攻击
* 使用HTTPS 以避免MITM (Man in the Middle Attack)
* 使用`HSTS` 以避免SSL Strip攻击


#### Input

输入

* 根据操作 使用恰当的HTTP方法. 如果请求的方法不适用于请求的资源则返回 `405 Method Not Allowed`
  * HTTP方法(用途): `GET (read)`, `POST (create)`, `PUT/PATCH (replace/update)`,`DELETE (to delete a record)` 
* 验证`content-type` 只允许后端支持的格式 并在不满足条件的时候返回 `406 Not Acceptable`
  * content-type: `application/xml`, `application/json`等
* 验证POST请求中的`content-type`是否和后端所接受的格式相同
  * content-type: `application/x-www-form-urlencoded`, `multipart/form-data`, `application/json`等
* 验证用户输入 以避免常规web漏洞
  * 如`XSS`, `SQL-Injection`, `Remote Code Execution`等
* 不要在URL中使用任何敏感的数据 而是使用标准的Authorization header(认证请求头)
  * 敏感数据: `credentials`, `Passwords`, `security tokens`,`API keys`等 
* 验证用户输入 以避免常规web漏洞
  * 常规漏洞: 如`XSS`, `SQL-Injection`, `Remote Code Execution`等
* 使用API Gateway服务 来启用 缓存(caching) 和 访问速率限制策略(Rate Limit policies). 
  * API Gateway services: Quota, Spike Arrest, Concurrent Rate Limit

#### Processing

处理

* 检查是否所有的endpoints都被身份认证保护 以避免破坏认证体系(authentication process).
* 避免直接使用用户自有资源的资源id. 使用 `/me/orders` 替代 `/user/654321/orders`
* 使用`UUID` 避免使用自增id
* 如果需要解析 XML 文件, 确保实体解析(entity parsing)是关闭的 以避免XXE(XML external entity attack)
* 如果需要解析 XML 文件, 确保实体扩展(entity expansion)是关闭的 以避免通过"指数实体扩展攻击"(exponential entity expansion attack)实现`Billion Laughs/XML bomb`
* 文件上传功能 使用CDN
* 如果需要处理大量的数据, 后端使用Workers和Queues来快速返回响应.以避免HTTP阻塞
* 记得关闭DEBUG模式

#### Output

输出(响应)

* Response中 加上以下header
  * `X-Content-Type-Options: nosniff` 
  * `X-Frame-Options: deny`
  * `Content-Security-Policy: default-src 'none'`
* Response中 删除指纹header
  * `X-Powered-By`, `Server`, `X-AspNet-Version` ...
* Response中 强制加上`content-type`header 并且根据响应内容指定具体值
  * 通常是`application/json`
* 不能返回敏感数据
  * `credentials`
  * `Passwords`
  * `security tokens`
  * ...
* 操作结束时 返回恰当的状态码
  * `200 OK`
  * `400 Bad Request`
  * `401 Unauthorized`
  * `405 Method Not Allowed`
  * ...


#### CI & CD

持续集成&持续部署

* 使用 单元测试/集成测试 来审计程序的设计和实现
* 使用代码审查(code review)流程 禁止自行批准更改
* 在推送到生产环境之前 确保服务的所有组件(包括第三方库和其他)都经过杀毒软件的静态扫描
* 为本次部署设计一个回滚解决方案

---

#### 参考

* [yosriady/api-development-tools](https://github.com/yosriady/api-development-tools) - A collection of useful resources for building RESTful HTTP+JSON APIs. 包括API规范 API框架 API文档 API监控 API网关 API测试 以及 各语言的API实现.
* [API-Security-Checklist](https://github.com/shieldfy/API-Security-Checklist/blob/master/README.md)
