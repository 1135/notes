### 简介

* 漏洞名称:点击劫持(ClickJacking)  UI覆盖(UI Redressing)
* 攻击条件:HTTP响应中 `X-FRAME-OPTIONS` 没有设置为 `DENY` 或 `SAMEORIGIN`
* 漏洞影响:引诱用户在某web页面进行点击 即可实现对目标网站的点击操作 如“更改数据” “删除帐户”等

### 测试

POC生成:burp

### SDL - 防御与修复方案

* 将 `X-FRAME-OPTIONS` 设置为 `DENY` 或 `SAMEORIGIN`
