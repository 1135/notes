# Phishing

#### 钓鱼的作用

* 突破边界 - 控制终端办公机 直接进入办公内网
* 信息搜集 - 获取各种信息

#### 鱼叉攻击在APT中的应用

很多apt攻击都从鱼叉攻击开始，掌握了攻击技巧成功率很高

>[2018年全球十大APT攻击事件盘点 - 安全客，安全资讯平台](https://www.anquanke.com/post/id/169007)

>[腾讯安全2018年高级持续性威胁（APT）研究报告 - 腾讯安全](https://s.tencent.com/research/report/623.html)
2018年，鱼叉攻击依然是APT攻击的最主要方式，使用鱼叉结合社工类的方式，投递带有恶意文件的附件，诱使被攻击者打开。虽然该方式攻击成本极低，但是效果却出人意料的好。这也进一步体现了被攻击目标的人员的安全意识亟需加强。从曝光的APT活动来看，2018年使用鱼叉攻击的APT活动比例高达95%以上。


#### 钓鱼文案 - 标题

* 社会高度关注的热点
  * `xx暴恐事件最新通报`
  * `公务员工资收入改革方案`
* 针对行业的信息
  * `xx最新调整`
* 针对个人的社工信息
  * `[xxx.com]您的账号在异地登录 请尽快修改密码`

#### 钓鱼文案 - 内容

* 案例1 [针对银行钓鱼事件的分析 - FreeBuf](https://www.freebuf.com/articles/web/129079.html)
  * 官方风格:钓鱼页面上的字体 页面布局 联系电话 都极力模仿Santander银行的风格
  * 诱导理由:警告用户的计算机上的安全模块已过期，如果不及时更新到最新版本，银行将向他们发出246.67BRL的罚单（约$80.00）
  * 触发场景:点击图片(超链接) 跳进钓鱼站点

#### 实践 - 邮件钓鱼全过程

* 编辑邮件（用自己邮箱发给自己一封邮件）
  * 标题
  * 正文内容（正文字体 图片 链接...各种编辑）
  * 附件
  * 等等
* 获取源文件(从发件箱下载到这封邮件的eml文件)
* 编辑源文件
  * 去除数据 - Date及其之前的都去掉
  * 去除数据 - 其他不必要的数据（一些header）
  * 修改数据 - 修改邮件发件人、收件人。
  * 注意：如果此处修改邮件收件人为a@x.com 但在最后命令中指定发给b@x.com 则实际上b@x.com收到了一封邮件，但收件人中只有a@x.com
* 使用swaks发送钓鱼邮件
  * 官方使用说明 http://jetmore.org/john/code/swaks/latest/doc/ref.txt
  * Mac下安装swaks  `brew install swaks`
* 发送钓鱼邮件（测试）
  * 按需修改"源文件"与"命令行"中的收件人与发件人
  * 命令 `swaks --to shoujian@a.net --from fajian@a.net --data /Users/x/Downloads/mail2755.eml.txt`
  * 效果查看 自己查看发送效果是否足够真实
* 发送钓鱼邮件（实战）
  * 按需修改"源文件"与"命令行"中的收件人与发件人

#### 钓鱼的工具

|名称|属性|运行环境|描述|
|:-------------:|--|--|-----|
|[gophish](https://github.com/gophish/gophish)|Golang|all|2k★ Open-Source Phishing Toolkit. 用于对企业进行定期的钓鱼测试. 启动后先在`Sending Profiles`中配置真实可用的`mail server`,发起一个Campaign:钓鱼邮件(内含钓鱼网站) [使用视频](https://www.youtube.com/watch?v=knc6Iq-hNcw)|
|[blackeye](https://github.com/flagellantX/blackeye)|.sh .php|/| 钓鱼页面Phishing Pages 含各大网站`Facebook,Google,Twitter,Microsoft`等|
|[UndeadSec/SocialFish](https://github.com/UndeadSec/SocialFish)|python3|/|终极的 Ultimate phishing tool. 使用xx.ngrok.io得到公网域名 含各大网站`Facebook,Google,Twitter,LinkedIn,GitHub,StackOverflow,Instagram`等|
|[kgretzky/evilginx2](https://github.com/kgretzky/evilginx2)|Golang|linux|2k★ MITM attack framework used for phishing. 效果好 只能通过域名识别出是钓鱼网站. [bolg - bBREAKDEV](https://breakdev.org/)|


在线邮箱

|名称|描述|
|:-------------:|--|
|https://emkei.cz/| #伪造邮件 在线发送伪造邮件(可用于钓鱼 如伪造邮件发送者xx@163.com发送到目标的qq邮箱)|
|http://www.yopmail.com/zh/| #临时邮箱 可随意进入xxx@yopmail.com收发邮件|
|http://www.mailinator.com/| #临时邮箱 可随意进入asd@mailinator.com收邮件|
|https://10minutemail.net/| #临时邮箱 十分钟邮箱 短期 每次邮箱地址不同 适用于注册小号|
|https://protonmail.com/| #匿名邮箱 瑞士的加密邮箱 长期 google验证码认证后可注册.[福布斯评价：NSA唯一无法进入的邮件服务](http://www.forbes.com/sites/hollieslade/2014/05/19/the-only-email-system-the-nsa-cant-access/#5060612155ed) |
|https://www.irongeek.com/homoglyph-attack-generator.php| #辅助工具 外观相似的替换字符 常用于钓鱼|


防御检测

|名称|描述|
|:-------------:|-----|
|http://spf.myisp.ch/|#防御 SPF Record Checker. 邮件服务器SPF配置检查.|
