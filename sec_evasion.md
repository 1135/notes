### 简介

"免杀"(AntiVirus Evasion)

完全不被检测 FUD(Fully undetectable)

[github.com Topic: antivirus-evasion](https://github.com/topics/antivirus-evasion)

### 威胁检测技术

* 静态扫描(Signatured Static Scanning) 即文件查杀
* 内存扫描(Run-time Analysis)
* 流量分析(NIPS/NIDS)
* 行为分析(Behavior Monitoring)
* 机器学习(Machine Learning)

### 免杀的维度

* 对抗-静态扫描
  * 1.shellcode加密
    * 逃避原理:避免被杀软直接获取到真正shellcode(因为性能等原因 杀软不会暴力枚举以解密内容)
    * 局限:使用了加解密函数 文件size可能会变大
  * 2.文件不落地
    * 逃避原理:文件不落地 难以被捕获样本
    * 局限:不落地方案的任何操作都被提示。如Powershell的任何操作都会被杀软提示“是否允许执行”。另外微软为Powershell等程序提供了扫描接口`Antimalware Scan Interface (AMSI)`以便对接受的数据进行扫描。
  * 3.源码级修改
    * 逃避原理:自主开发程序，源码级的修改较容易实现"变种",避免杀软报毒(因为从未被捕获分析 应该不会被杀 所以建议一个样本仅对某一目标攻击)
    * 局限:彻底改写源码难度大
  * 其他
* 对抗-内存扫描
  * 1.DLL加载
    * 逃避原理:杀软为了不影响系统性能，通常对DLL加载的审查宽松(加载途径:自己编程实现dll加载器 或 进行"dll hijacking")
    * 局限:文件落地
  * 2.自定义加载器(Customer Loader)
    * 逃避原理:使用不同编程语言实现的shellcode Loader，运行机制较不同，杀软可能没有足够精力跟进各种形式的加载器(如C#使用虚拟机解释后运行 Golang编译运行 等)
    * 局限:文件落地
  * 其他
* 对抗-流量分析
  * 1.白域名方法 - "Domain Fronting"
    * 逃避原理:借用CDN实现"隐藏"C2服务器的真实ip 避免溯源分析  C2流量 -> CDN ip -> C2 Server ip
    * 局限:使用CDN，可能需要实名认证(可考虑以其他身份注册)
  * 2.白域名方法 - 借用知名网站
    * 逃避原理:将"命令数据"加密处理发布在知名网站 即可实现被控端通过知名域名获取C2命令 避免流量特征告警 避免溯源分析
      * 国内 可被公开访问的页面 `weibo.com` `users.qzone.qq.com`
      * 国外 可被公开访问的页面 `google、twitter、pastbin、telegram ...`
   	* 局限:使用知名站点，可能需要实名认证(可考虑以其他身份注册)
  * 3.DGA域名 - DGA域名生成算法
    * 逃避原理:使用dga可以试图连接多个C2域名 从而避免"单一C2域名/ip特征被当作IoC后引发报警"这一报警机制(黑名单) 大大增加了C2架构稳定性
    * 局限:dga生成的域名可能被机器学习识别为恶意的.  dga需要注册不止一个域名. 可被持续追溯得到多个C2服务器ip.
  * 4.DNS over HTTPS
    * 逃避原理:通过https在线查询DNS记录(A TXT ...) 获取C2 server的IP. 参考实际案例[Godlua Backdoor分析报告](https://blog.netlab.360.com/an-analysis-of-godlua-backdoor/)
    * 局限:单次查询的数据传输量有限
  * 其他
* 对抗-行为分析
  * 1.指定特定的运行条件
    * 逃避原理:即符合一定的条件才会进行"恶意的"操作行为。从而避免该程序在VM、沙箱、逆向人员的眼皮底下进行恶意操作
    * 局限:可能会限制运行环境，导致缩小可运行的目标范围(如只能在某个指定域中计算机才能运行该程序)
  * 其他
* 对抗-逆向分析
  * 1.加壳
    * 逃避原理:通过加壳等方式延长被逆向分析人员彻底分析的时间
    * 局限:没有绝对的反逆向保护，只能增加逆向分析难度
  * 其他
* 对抗-机器学习
  * client ML - 杀软客户端内置的轻量级机器学习模型
  * Cloud ML - 相关ML技术及训练数据不得而知

**注意:逃避技术是和杀软技术的长期对抗。所有高超的逃避技巧都迟早被威胁检测技术发现。因为他们有尖端的技术和人才。**

### 免杀工具

|名称|属性|针对目标|描述|
|:-------------:|--|---|----|
|[EgeBalci/Amber](https://github.com/EgeBalci/Amber)|go+asm|仅针对 Windows| Reflective PE packer. 将PE文件打包成reflective payloads 可以无文件执行 像shellcode一样加载并执行自身|
|[avet](https://github.com/govolution/avet)|C+shell+python|仅针对Windows| AntiVirus Evasion Tool 运行于kali2|
|[Veil-Evasion](https://github.com/Veil-Framework/Veil-Evasion)|shell+C+python2.7|可针对Windows exe| generate metasploit payloads that bypass common anti-virus solutions|
|[tokyoneon/Armor](https://github.com/tokyoneon/Armor)|shell|MacOS|Armor is a simple Bash script designed to create encrypted macOS payloads capable of evading antivirus scanners.|
|[GreatSCT](https://github.com/GreatSCT/GreatSCT)|python3|/|/|
|[ASWCrypter](https://github.com/AbedAlqaderSwedan1/ASWCrypter)|shell+python|/|An Bash&Python Script For Generating Payloads that Bypasses All Antivirus so far FUD.实测无法过360ZhuDongFangyu|
|[tokyoneon/Armor](https://github.com/tokyoneon/Armor)|shell|仅针对MacOS|Armor is a simple Bash script designed to create encrypted macOS payloads capable of evading antivirus scanners.|
|[csvoss/onelinerizer](https://github.com/csvoss/onelinerizer)|python|/|把Python代码给编译成一句话的形式工具 Convert any Python file into a single line of code.|


#### 无文件攻击

* "无文件"的优势 - 不落地 样本难以被捕获

|名称|描述|针对目标|
|--|--|-----|
|[无文件攻击的现实实例FCL(Fileless Command Lines)](https://github.com/chenerlich/FCL)|包括:恶意软件名称/执行过程/恶意命令行/正则表达式用于检测/技术报告/沙箱报告链接/笔记|windows|


### 免杀实例1 - 全面免杀

[渗透利器Cobalt Strike - 第2篇 APT级的全面免杀与企业纵深防御体系的对抗 - 先知社区](https://xz.aliyun.com/t/4191)

### 免杀实例2 - 从远程获取加密payload

简介：使用Armor工具生成加密的payload，MacOS下的one_liner从远程地址获取加密payload，解密，执行。

参考作者介绍[Hacking macOS: How to Create an Undetectable Payload « Null Byte](https://null-byte.wonderhowto.com/how-to/hacking-macos-create-undetectable-payload-0189715/)

**原理解析**

预先创建一个MacOS下可执行的常规raw payload(shell command)

|顺序|攻击机(使用Armor工具)|网络|靶机|
|:-------------:|--|--|-----|
|1加密|使用密钥对payload进行加密，得到enc.txt|攻击机的443端口与victim无通信流量|/|
|2待解密|Ncat监听443端口 以供victim访问并获取解密密钥enc.key|攻击机的443端口与victim无通信流量|/|
|3解密|Ncat监听443端口|攻击机的443端口与victim通信该过程是https(抓到流量也无法得到解密密钥)|执行enc.txt 第1步获取解密密钥:连接到攻击机的443端口 获取解密密钥enc.key 第2步:解密成功并执行 常规raw payload(shell command)|
|4结束|Ncat结束监听443端口 用于密钥传输的端口(默认443)只会被连接1次 传输完密钥后立即关闭|攻击机的443端口与victim无通信流量|/|


* 实现效果
  * 流量分析 流量分析无法获取密钥(https通信)
  * 终端安全 杀软无法解密得到加密命令的含义(无密钥)
