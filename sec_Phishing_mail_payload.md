### APT案例 - 钓鱼邮件中的payload载体

* 凭证窃取(钓鱼网站)
  * 直接 - 超链接跳转到钓鱼网站
  * 间接 - 无害文件(如.pdf)中附带 超链接跳转到钓鱼网站
* 恶意程序(各种形式的可执行文件) - 需要符合目标的系统环境(Windows Mac ...)
  * 直接 - 邮件附件
  * 间接 - 共有云盘链接

|恶意程序|文件后缀|原理与案例|
|:----:|-|----|
|快捷方式文件|`.lnk`|蓝宝菇APT 内嵌PowerShell脚本的LNK文件|
|内嵌恶意Marco的offcie文档|`.doc`|内嵌恶意宏的Word文档 需要目标点击"启动宏"才能使payload执行|
|word文档|`.doc`|利用漏洞(如CVE-2017-11882)实现打开`.doc`文件即可无交互地执行payload|
|HTA文件|`.hta`|APT|
|使用Office软件图标的可执行文件|/|VBA Marco - [蔓灵花(BITTER) APT](https://s.tencent.com/research/report/615.html)的邮件中投递了使用word图标的exe自解压文件|
|.rar压缩文件|`.rar`|如果目标使用的rar程序存在CVE-2018-20250漏洞,当打开evil.rar常常会把恶意程序解压到"启动"`startup`文件夹|
|SFX自解压文件|/|APT|

参考项目 [SocialEngineeringPayloads](https://github.com/bhdresh/SocialEngineeringPayloads)


### 实践 - 构建包含payload的Office文档

* 工具 - [outflanknl/EvilClippy](https://github.com/outflanknl/EvilClippy) [参考官方说明](https://outflank.nl/blog/2019/05/05/evil-clippy-ms-office-maldoc-assistant/)
* [恶意文档之 VBA 进阶之旅](https://mp.weixin.qq.com/s?timestamp=1564024589&src=3&ver=1&signature=5qCWwdoC3R-ovJyrkClXoQRo6yJKXys8mWW8Im6PKLEJ8GKti7erXOiKRWpjFM48P8DzIs3pZpcikyioCqEV69x0ATWtBbALMn5MkhjaDa8V-BCQfg7kGLJRjj3q0ncqHR22LmaXLVmqvlXooa6MZ3V0P993xuRPJmE5l9JRC58=)
