### 获取安全资讯

* 模式
  * 被动接收为主 (营造环境/建立机制 让资讯主动展示在眼前 避免不必要的消耗)
  * 主动获取为辅（主动获取略耗费精力时间 按需获取)


### 信息来源1 - RSS订阅

订阅以下rss地址


|标题|rss地址|
|:-------------:|-----|
| ProjectZero | https://googleprojectzero.blogspot.com/feeds/posts/default |
| Threat Research(FireEye.com) | [Threat Research Blog](https://www.fireeye.com/blog/threat-research/_jcr_content.feed) |
| Security Research & Defense(Microsoft.com) | https://blogs.technet.microsoft.com/srd/feed/|
| Unit42 (Paloalto威胁分析团队) | http://feeds.feedburner.com/Unit42 |
| [RedTeam] Strategic Cyber LLC(cobaltstrike blog)| https://blog.cobaltstrike.com/feed/ |
| [RedTeam] Outflank Blog| https://outflank.nl/blog/feed/ |
| [RedTeam] Threat Express| http://threatexpress.com/feed/ |
| [RedTeam] SpecterOps Team[专家成员](https://specterops.io/who-we-are/the-team) | https://posts.specterops.io/feed|
| [RedTeam] harmj0y| http://www.harmj0y.net/blog/feed/|
| [RedTeam] Dirk-jan Mollema(researcher)|https://dirkjanm.io/feed.xml|
| [RedTeam] adsecurity.org | https://adsecurity.org/?feed=rss2 |
| DEVCORE 戴夫寇爾(台湾知名安全公司)| https://devco.re/rss |
| 360 CERT | https://cert.360.cn/daily/feed |
| 360 Technology Blog(360核心安全) | http://blogs.360.cn/rss.html|
| 360 netlab |https://blog.netlab.360.com/rss/|
| 安全客 | https://api.anquanke.com/data/v1/rss|
| Malwarebytes Labs | https://blog.malwarebytes.com/feed/|
| [crowdstrike](https://www.crowdstrike.com/blog/) | 无rss |
| 知道创宇|https://www.seebug.org/ |
| 知道创宇 Papers - Last paper |https://paper.seebug.org/|
| Seebug 最新漏洞 | https://www.seebug.org/rss/new/|
| Seebug 近期最热漏洞 | https://www.seebug.org/rss/hot/|
| 腾讯安全响应中心 | https://security.tencent.com/index.php/feed/blog/0|
| FreeBuf | https://www.freebuf.com/feed|
| 先知安全技术社区 | https://xz.aliyun.com/feed |
| 绿盟科技博客 | http://blog.nsfocus.net/feed/ |
| Research – Check Point Blog | https://blog.checkpoint.com/category/research/feed/ |
| [Research - ZecOps Blog](https://blog.zecops.com/) | https://blog.zecops.com/feed/ |
| PortSwigger(burpsuite公司) - Web Security Blog | https://portswigger.net/blog/rss |
| PortSwigger(burpsuite公司) - The Daily Swig | https://portswigger.net/daily-swig/rss |
| acunetix blog | https://www.acunetix.com/blog/feed/ |
| KitPloit - PenTest Tools! | https://feeds.feedburner.com/PentestTools |
| Akamai Security Intelligence and Threat Research Blog | https://blogs.akamai.com/sitr/atom.xml |
| Trend Micro Simply Security | http://feeds.trendmicro.com/TrendMicroSimplySecurity |
| www.nccgroup.trust - US Blog RSS Feed |https://www.nccgroup.trust/us/about-us/newsroom-and-events/blog/us-blog-rss-feed/|
| Malware-Traffic-Analysis.net(流量分析) | https://www.malware-traffic-analysis.net/blog-entries.rss |
| [Blog - NotSoSecure](https://www.notsosecure.com/blog/) | https://www.notsosecure.com/blog/feed/ |
| [Anomali Blog(APT/threat)](https://www.anomali.com/blog) | https://www.anomali.com/site/blog-rss |
| [Blog – SentinelOne](https://www.sentinelone.com/blog/) | 无法订阅 |
| https://www.gosecure.net/blog/ | 无法订阅 |
| Thomas Orlita's blog| https://blog.thomasorlita.cz/feed/ |
| Somdev Sangwan(XSStrike AwesomeXSS author) | https://somdev.me/feed |
| 个人博客 Nytro (web 二进制 Defcon BlackHat) |https://nytrosecurity.com/feed/|
| 个人博客 0x09al | https://0x09al.github.io/feed.xml|
| CORBEN LEO |https://www.corben.io/feed|
| Somdev Sangwan(XSStrike AwesomeXSS author) | https://somdev.me/feed |
| InQuest Blog (Blackhat USA 2019 speaker)| https://inquest.net/blog/rss |
| 个人博客 XPN Redteamer windows | https://blog.xpnsec.com/rss/ |
| 个人博客 m0chan windows域渗透 | https://m0chan.github.io/ |
| [hackndo](https://en.hackndo.com/archives/) windows域渗透| 无法订阅 |

|名称|属性|描述|
|:-------------:|--|-----|
|[OWASP](https://www.owasp.org/index.php/Main_Page)|知识库|自由开放的软件安全社区|
|-|知识库|OWASP web测试指南V4 [owasp-testing-guide-v4(中文)](https://kennel209.gitbooks.io/owasp-testing-guide-v4/content/zh/index.html)|
|-|知识库|#反序列化安全# [Deserialization Cheat Sheet](https://www.owasp.org/index.php/Deserialization_Cheat_Sheet)|
|[每日安全 - 腾讯安全玄武实验室出品](https://sec.today/)|/|实时安全技术信息推送 https://sec.today/pulses/|





如 使用Slack客户端(PC/mobile) 进行订阅:

访问 https://YOURNAME.slack.com/apps/manage 并搜索 RSS安装，添加上述RSS地址即可。效果如图：

![slack](https://images2.imgbox.com/04/ec/AVZP9xil_o.png)

### 信息来源2 - 自写爬虫邮件推送

* 邮件推送(自写爬虫爬取无法rss订阅的网站)
  * [1135/VulSpider: 本程序在后台持续运行，获取最新漏洞及每日简报，发送邮件给安全人员。](https://github.com/1135/VulSpider)
  * [1135/VulSpiderX: 本程序在后台持续运行，通过headless chrome获取hackerone最新漏洞，发送邮件给安全人员。](https://github.com/1135/VulSpiderX)

效果如图
![all](https://github.com/1135/notes/blob/master/imgs/vulspider.png?raw=true)
![all](https://github.com/1135/notes/blob/master/imgs/vulspiderX.png?raw=true)
