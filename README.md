#### 公开笔记

 * [SDL - 安全开发生命周期 实践与意义](sdl.md)
 * [SDL - API设计规范 CheckList](sdl_API.md)
 * [NTA - 网络流量分析 基础 抓包实践(wireshark/Tshark)](NTA_analysis.md)
 * [NTA - 网络流量分析 IDS/IPS 原理 引擎 规则(suricata)](NTA_suricata.md)
 * [sec - Elasticsearch及Elastic Stack](sec_Elasticsearch.md)
 * [web - vul - SQLi 原理 利用方式 修复方案](web_vul_SQLi.md)
 * [web - vul - SSRF 原理 利用方式 修复方案](web_vul_SSRF.md)
 * [web - vul - XSS  原理 利用方式 修复方案](web_vul_XSS.md)
 * [web - vul - CSRF 原理 利用方式 修复方案](web_vul_CSRF.md)
 * [web - vul - Path Traversal 原理 利用方式 修复方案](web_vul_PathTraversal.md)
 * [web - vul - Upload 原理 利用方式 修复方案](web_vul_upload.md)
 * [web - vul - Deserialization 原理 利用方式 修复方案](web_vul_Deserialization.md)
 * [web - vul - Command Injection  原理 利用方式 修复方案](web_vul_CommandInjection.md)
 * [web - vul - Open Redirection 原理 利用方式 修复方案](web_vul_OpenRedirection.md)
 * [web - vul - Clickjacking 原理 利用方式 修复方案](web_vul_ClickJacking.md)
 * [web - SQLi sqlmap 常用参数 tamper详解 实际案例](sec_sqlmap.md)
 * [web - WEB应用安全部署架构 及 WAFbypass](web_WAF_bypass.md)
 * [web - 基础 - 密码学基础及应用(TLS 数字证书) 非对称加密RSA](web_x_https_tls.md)
 * [web - 基础 - 浏览器的同源策略(SOP) 跨域方案(CORS JSONP) 内容安全策略(CSP)](web_x_SOP.md)
 * [web - 基础 - HTTP协议 HTTP/2](web_HTTP.md)
 * [web - 代码审计 - PHP 代码审计 思路 实践 Checklist (find grep)](web_code_audit_PHP.md) **private_update**
 * [web - 整理 - 靶场WebGoat8的搭建 WriteUp](z_web_webgoat.md)
 * [web - 整理 - 爬虫分类](z_web_crawl.md)
 * [web - burp 设置 插件 技巧](web_x_burp.md)
 * [web - burp Intruder的4种攻击方式](web_x_burp_Intruder.md)
 * [Red - 公网信息搜集 - 搜索引擎(Google/shodan) 代码仓库(Github) Domain IP 历史信息](sec_info_gathering.md)
 * [Red - APT Phishing - 鱼叉邮件 & 钓鱼网站](sec_Phishing.md)
 * [Red - APT Phishing - 鱼叉邮件中的payload载体(office EvilClippy)](sec_Phishing_mail_payload.md)
 * [Red - 多维度的免杀 无文件攻击 (对抗 终端安全 流量分析 逆向分析 行为分析)](sec_evasion.md) 
 * [Red - 后渗透 - 权限维持 信息搜集 白利用 (ATT&CK backdoor PostExploitation) BypassUAC](sec_RAT_post_exploitation.md)
 * [Red - DLL Hijacking 原理 利用方式](sec_vul_DllHijacking.md)
 * [Red - 主机信息搜集+提权 windows](sec_PrivilegeEscalation_windows.md)
 * [Red - 主机信息搜集+提权 linux](sec_PrivilegeEscalation_linux.md)
 * [Red - 主机信息搜集 应急响应 macOS](sec_mac.md)
 * [Red - 构建高适应性的C2基础设施](sec_C2.md)
 * [Red - msfvenom - msf的payload生成器](sec_msfvenom.md)
 * [Red - 反弹shell的各种方式(使用加密通信对抗检测)](sec_ReverseShellCheatsheet.md)
 * [Red - webshell管理工具与检测方法](sec_webshell.md)
 * [Red - Proxy tools 代理类工具](sec_proxy.md)
 * [Red - 基础 - Windows远控](sec_RAT.md)
 * [sec - 基础 - 字符与编码 格式转换](sec_encoder.md)
 * [sec - 基础 - regex 正则表达式](sec_regex.md)
 * [sec - 获取最新安全资讯的具体方式(RSS源 mail)](sec_get_news.md)
 * [sec - 企业信息泄露源 监测(github等代码仓库) + 应急处置](sec_info_leak.md)
 * [sec - 主机端口扫描 思路及自动化](host_sec_port_service_exp.md)
 * [bin - pwn linux下的二进制程序pwn](bin_pwn.md)
 * [bin - GDB调试 原理 命令](bin_GDB.md)
 * [net - iptables 网络策略管理](net_iptables.md)
 * [笔记 - Docker 命令](note_Docker.md)
 * [笔记 - headless Chrome & Puppeteer](web_headless.md)
 * [笔记 - Android应用安全](note_sec_android.md)
 * [笔记 - blockchain - ETH 智能合约 基本概念](blockchain_SmartContracts.md)
 * [笔记 - blockchain - 虚拟币挖矿的各种方式](blockchain_mining.md)
 * [笔记 - 实现各系统的模拟操作](z_auto_operation.md)

#### 公开案例

 * [Path Traversal to RCE - Atlassian Bitbucket Data Center](web_case_PathTraversal_to_RCE.md) CVE-2019-3397

#### 公开项目

|项目名称|属性|描述|
|:--:|--|---------|
|[1135-CobaltStrike-ToolKit](https://github.com/1135/1135-CobaltStrike-ToolKit)|/|Malleable C2 Files + AggressorScripts|
|[solr_exploit](https://github.com/1135/solr_exploit)|/|Apache Solr远程代码执行漏洞(CVE-2019-0193) Exploit 支持结果回显|
|[EquationExploit](https://github.com/1135/EquationExploit)|Java C++|在Windows下针对网段批量利用永恒之蓝漏洞(MS-17010 EternalBlue) |
|[VulSpiderX](https://github.com/1135/VulSpiderX)|node.js|后台持续运行，获取hackerone最新漏洞，发送邮件给安全人员|
|[VulSpider](https://github.com/1135/VulSpider)|python2|后台持续运行，获取最新漏洞及每日简报，发送邮件给安全人员|
|[dictionary](https://github.com/1135/dictionary)|txt|字典收集 包括user/name/pass/web|


#### Red-Team

|名称|描述|
|:-------------:|--|
|https://ired.team/| Red Teaming Experiments |
|https://attack.mitre.org/| ATT&CK™ 真实世界用到的、系统化的网络攻击技术|
|[blackhat-arsenal-tools](https://github.com/toolswatch/blackhat-arsenal-tools)|官方仓库 Black Hat Arsenal Security Tools Repository|
|[Red-Team-Infrastructure-Wiki](https://github.com/bluscreenofjeff/Red-Team-Infrastructure-Wiki)|红队基础设施Wiki(构建稳定C2)|
|[infosecn1nja/Red-Teaming-Toolkit](https://github.com/infosecn1nja/Red-Teaming-Toolkit)|red team tools.(infosecn1nja是Empire的作者)|
|[infosecn1nja/AD-Attack-Defense](https://github.com/infosecn1nja/AD-Attack-Defense)|Active Directory Security For Red & Blue Team|
|[Awesome-Red-Teaming](https://github.com/yeyintminthuhtut/Awesome-Red-Teaming)|List of Awesome Red Teaming Resources|

#### Awesome_and_cheat_sheets

|名称|描述|
|:-------------:|-----|
|[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)|8k★ 持续更新的好资源 useful payloads and bypass for Web and Pentest|
|[Awesome-Hacking](https://github.com/Hack-with-Github/Awesome-Hacking)|Awesome-Hacking|
|[Awesome-CheatSheet](https://github.com/wxyyxc1992/Awesome-CheatSheet)|全栈 多语言基本语法 CheatSheet|
|[Awesome-Go](https://github.com/avelino/awesome-go)|37k★ Awesome-Go|
|[Awesome-cryptography](https://github.com/sobolevn/awesome-cryptography)|2k★ 密码学资源|
|[Awesome-crawler](https://github.com/BruceDone/awesome-crawler)|3k★ 各语言爬虫 web crawler|
|[Awesome-static-analysis](https://github.com/mre/awesome-static-analysis)|4k★ 各语言静态分析工具Static analysis tools for all programming languages|
|[Awesome-pentest-cheat-sheets](https://github.com/coreb1t/awesome-pentest-cheat-sheets)|1k★ #pentesting |
|[Awesome-malware-analysis](https://github.com/rshipp/awesome-malware-analysis)|4k★ #流量分析 A curated list of awesome malware analysis tools and resources.|
