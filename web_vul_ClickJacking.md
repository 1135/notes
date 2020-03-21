### 简介

* 漏洞名称:点击劫持(ClickJacking)  UI覆盖(UI Redressing)
* 攻击条件:HTTP Response Header 中 `X-FRAME-OPTIONS` 没有设置为 `DENY` 或 `SAMEORIGIN`

### 漏洞危害

* 引诱用户在第三方web网站`3.com`中点击页面某处，其实是对目标网站`pay.com`某处的点击，从而实现恶意操作。如“创建用户”、“更改密码”、“删除帐户”等

### 测试

POC生成:burp

### SDL - 防御与修复方案

* 在HTTP Response Header 中设置 `X-Frame-Options: DENY` 或 `X-Frame-Options: SAMEORIGIN`
