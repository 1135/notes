### APT案例 - 钓鱼邮件中的payload载体

* 前期准备
  * 邮件往来建立初步信任
* 凭证窃取(依靠"公网链接")
  * 直接 - 邮件正文含有"公网链接" 跳转到钓鱼网站
  * 间接 - 邮件附件是个“无害文件”(如PDF文件其中内容含有"公网链接" 跳转到钓鱼网站
* 恶意程序(依靠各种形式的"Payload") 成功的前提是`payload执行所需的系统环境=目标的系统环境(Windows Mac ...)`
  * 直接 - 邮件附件是个"恶意程序"
  * 间接 - 邮件正文含有"公网链接"
    * 常规 点击"公网链接"下载恶意程序. 仅当目标主动打开恶意程序时 才执行恶意程序
    * 高级 点击一次即可触发 点击"公网链接"触发浏览器0day 如果符合0day触发条件 则执行恶意程序

|Payload|文件后缀|原理与案例|
|:----:|-|----|
|浏览器0day链接|/|发送钓鱼邮件 诱导点击"公网链接" 触发浏览器0day. 案例[Responding to Firefox 0-days in the wild - The Coinbase Blog](https://blog.coinbase.com/responding-to-firefox-0-days-in-the-wild-d9c85a57f15b)|
|快捷方式文件|`.lnk`|蓝宝菇APT 内嵌PowerShell脚本的LNK文件|
|内嵌恶意Marco的offcie文档|`.doc`|内嵌恶意宏的Word文档 需要目标点击"启动宏"才能使payload执行|
|word文档|`.doc`|利用漏洞(如CVE-2017-11882)实现打开`.doc`文件即可无交互地执行payload|
|HTA文件|`.hta`|APT|
|使用Office软件图标的可执行文件|`*`|VBA Marco - [蔓灵花(BITTER) APT](https://s.tencent.com/research/report/615.html)的邮件中投递了使用word图标的exe自解压文件|
|.rar压缩文件|`.rar`|如果目标使用的rar程序存在CVE-2018-20250漏洞,当打开evil.rar常常会把恶意程序解压到"启动"`startup`文件夹|
|SFX自解压文件|`.sfx`|SFX(SelF-eXtracting)自解压文件是一种压缩文件 双击该文件就能自动执行解压缩|
| SCR文件 |`.scr`|TransparentTribe APT(来自巴基斯坦)|

参考项目 [SocialEngineeringPayloads](https://github.com/bhdresh/SocialEngineeringPayloads)


### 实践 - 构建包含payload的Office文档

* 工具 - [outflanknl/EvilClippy](https://github.com/outflanknl/EvilClippy) [参考官方说明](https://outflank.nl/blog/2019/05/05/evil-clippy-ms-office-maldoc-assistant/)
* [恶意文档之 VBA 进阶之旅](https://mp.weixin.qq.com/s?timestamp=1564024589&src=3&ver=1&signature=5qCWwdoC3R-ovJyrkClXoQRo6yJKXys8mWW8Im6PKLEJ8GKti7erXOiKRWpjFM48P8DzIs3pZpcikyioCqEV69x0ATWtBbALMn5MkhjaDa8V-BCQfg7kGLJRjj3q0ncqHR22LmaXLVmqvlXooa6MZ3V0P993xuRPJmE5l9JRC58=)
