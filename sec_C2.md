### 构建高适应性的C2基础设施
Building resilient C2(command and control) infrastructures

* 持久访问方式(Persistent access)
  * 备用低频方式(long-haul beacons)stage 1
    * 功能极简 - 当所有的 日常高频方式(short-haul beacons) 都失败时 可以恢复对目标网络的访问(Backup channel)
    * 隐蔽性高 - 以尽量不让防御者发现的方式运作的方式：低频率的回调(calls back)、数据量小(如只传递payload的 delivery地址) ...
  * 日常高频方式(short-haul beacons)stage 2
    * 功能完整 - 功能强大的RAT`Cobalt Strike` `Empire` `Slingshot`...
    * 隐蔽性低 - 攻击者日常使用的、较频繁的对目标网络进行操作的方式
* 健壮的C2基础设施 特征
  * channel多样性 - 具有多个channel的持久访问 稳定性强（避免丢失目标网络的权限）
  * channel独立性 - 如果一个channel被发现，不会直接暴露其他channel（避免关联发现其他channel）
  * channel隐蔽性 - 通信流量加密形式或与正常的网络流量混合（避免各种方式的调查）

### 常规过程

`stage 1` -> `stage 2(RAT)`

C2基础设施的稳定性 取决于`stage 1`的方案

### stage 1 的方案 - DNS over HTTPS (DoH)

* 参考
  * 通过https协议进行DNS解析 协议规范[RFC 8484 - DNS Queries over HTTPS (DoH)](https://tools.ietf.org/html/rfc8484)
  * DNS-over-HTTPS 提供DoH服务的域名 https://github.com/curl/curl/wiki/DNS-over-HTTPS
  * DNS-over-HTTPS用作C2 参考 https://outflank.nl/blog/2018/10/25/building-resilient-c2-infrastructues-using-dns-over-https/

* stage 1 : DNS over HTTPS (DoH) 优势与特点
  * 可控字符串 `响应中有部分内容是攻击者可控的`
  * “白”域名 `Google Public DNS` https://dns.google.com/resolve?name=qq.com&type=TXT
  * 流量加密 `Over an SSL-encrypted channel`且许多已实施SSL检查的客户将所有`Google domains`排除在检查范围之外（因为Google产品中的证书，流量负载，隐私等）


**原理**

* 攻击者设置`stager 1`
  * 通过设置该域名 的[TXT record](https://en.wikipedia.org/wiki/TXT_record)中的`SPF records`中的字符串(通常这里放的是 `IP addresses` `domains` `server names`) 在此处放上域名是很合理的 符合隐蔽性高的要求
  * 如`"v=spf1 include:spf.mail.qq.com -all"`  `"v=spf1 ip4:192.0.2.0/24 ip4:198.51.100.123 a -all"`

**发起请求**

`DoH via Google`https GET

`https://dns.google.com/resolve?name=qq.com&type=TXT`

（请求中的url参数值`qq.com`为我们可控的域名evildomain）

**得到响应**

response:
```
{
  "Status": 0,
  "TC": false,
  "RD": true,
  "RA": true,
  "AD": false,
  "CD": false,
  "Question": [
    {
      "name": "qq.com.",
      "type": 16
    }
  ],
  "Answer": [
    {
      "name": "qq.com.",
      "type": 16,
      "TTL": 3329,
      "data": "\"v=spf1 include:spf.mail.qq.com -all\""
    }
  ]
}
```

在响应中看到 可控字符串（其中data字段中的字符串内容为我们可控的）


* 目标主机的`stage 1`执行顺序
  * `SPFstager`定期查询内容：提取DNS`TXT record`中`SPF records`的信息
  * `SPFtrigger`
    * 触发条件
      * 当发现了`SPF records`中的新域名时 得到了域名evildomain
    * 提取隐藏信息-方案a`http-robots.txt`
      * `SPFtrigger`程序根据evildomain域名 访问`http://evildomain/robot.txt`提取`payload` 进行decode得到raw payload
      * https://github.com/outflanknl/DoH_c2_Trigger/blob/master/DoHtrigger.ps1
    * 提取隐藏信息-方案b`http-xxx.png`
      * `SPFtrigger`程序根据evildomain域名 访问`http://evildomain/date.png`提取`payload` 进行decode得到raw payload
      * https://github.com/peewpw/Invoke-PSImage
  * 触发执行 - `payload triggering`(stage 2)
