### 简介

Suricata - network threat detection engine

参考Suricata官方 [All features](https://suricata-ids.org/features/all-features/)

* 作用
  * 实时入侵检测 - real time intrusion detection (IDS)
  * 入侵防御 - inline intrusion prevention (IPS)
  * 网络安全监控 - network security monitoring (NSM)
  * 离线pcap处理 - offline pcap processing

* 检测威胁的原理
  * 大量有效的检测规则
  * 签名语言(signature language)
  * Lua脚本(高度自定义 便于检测复杂威胁)

* 优点
  * 使用标准输入和输出格式 `yarm` `json` 便于与已有的SIEM类系统、Elk(Elastisearch Logstash Kibana)配合使用
  * 配置文件可读(YAML config file)
  * ...

* IP Reputation - IP信誉
  * loading of large amounts host based reputation data - 加载大量基于主机的信誉数据
  * matching on reputation data in the rule language using the “iprep” keyword - 使用关键字iprep匹配规则语言中的"信誉数据"
  * live reload support - 支持实时重新加载
  * supports CIDR ranges - 支持使用CIDR表示IP范围

### Suricata的数据包获取

* Packet acquisition
  * High performance capture - 高性能捕获
    * AF_PACKET - experimental eBPF and XDP modes available
    * PF_RING
    * NETMAP
  * Standard capture - 标准捕获
    * PCAP
    * NFLOG (netfilter integration)
  * IPS mode
    * Netfilter based on Linux (nfqueue) - fail open support
    * ipfw based on FreeBSD and NetBSD
    * AF_PACKET based on Linux
    * NETMAP
  * Capture cards and specialized devices
    * Endace
    * Napatech
    * Tilera

### Suricata的流量解析引擎

* TCP/IP engines
  * Scalable flow engine - 可扩展的流量引擎
  * Full IPv6 support
  * Tunnel decoding
    * Teredo
    * IP-IP
    * IP6-IP4
    * IP4-IP6
    * GRE
  * TCP stream engine
    * tracking sessions - 追踪会话
    * stream reassembly - 流的重组
    * target based stream reassembly - 基于目标的流的重组
  * IP Defrag engine
    * target based reassembly - 基于目标的重组

* Protocol parsers - 协议解析
  * Support for packet decoding of
    * IPv4, IPv6, TCP, UDP, SCTP, ICMPv4, ICMPv6, GRE
    * Ethernet, PPP, PPPoE, Raw, SLL, VLAN, QINQ, MPLS, ERSPAN
  * App layer decoding of:
    * HTTP, SSL, TLS, SMB, DCERPC, SMTP, FTP, SSH, DNS, Modbus, ENIP/CIP, DNP3, NFS, NTP, DHCP, TFTP, KRB5, IKEv2
    * New protocols developed in the Rust language, for safe and fast decoding.

* HTTP engine
  * Stateful HTTP parser built on libhtp -基于libhtp的"有状态的"HTTP解析器
  * HTTP request logger
  * File identification, extraction and logging - 文件识别、提取和日志记录
  * Per server settings — limits, personality, etc
  * Keywords to match on (normalized) buffers - 关键字匹配
    * uri and raw uri
    * headers and raw headers
    * cookie
    * user-agent
    * request body and response body
    * method, status and status code
    * host
    * request and response lines
    * decompress flash files
    * and many more


### Suricata的检测引擎

* Detection engine
  * Protocol keywords
  * Multi-tenancy per vlan or capture device - 每个VLAN或捕获设备的多租户
  * xbits – flowbits extension
  * PCRE support
    * substring capture for logging in EVE
  * fast_pattern and prefilter support
  * Rule profiling
  * File matching
    * file magic
    * file size
    * file name and extension
    * file MD5/SHA1/SHA256 checksum — scales up to millions of checksums
  * multiple pattern matcher algorithms that can be selected
  * extensive tuning options - 广泛的调优选项
  * live rule reloads — use new rules w/o restarting Suricata  实时规则重新加载-使用新规则而不重新启动Suricata
  * delayed rules initialization
  * Lua scripting for custom detection logic - 用于自定义检测逻辑的Lua脚本
  * Hyperscan integration


### Suricata的输出与筛选

* Outputs
  * Eve log, all JSON alert and event output
  * Lua output scripts for generating your own output formats - 自定义输出格式
  * Redis support
  * HTTP request logging
  * TLS handshake logging
  * Unified2 output — compatible with Barnyard2
  * Alert fast log
  * Alert debug log — for rule writers
  * Traffic recording using pcap logger
  * Prelude support
  * drop log — netfilter style log of dropped packets in IPS mode
  * syslog — alert to syslog
  * stats — engine stats at fixed intervals
  * File logging including MD5 checksum in JSON format
  * Extracted file storing to disk, with deduplication in the v2 format
  * DNS request/reply logger, including TXT data
  * Signal based Log rotation
  * Flow logging

* Alert/Event filtering - 警报/事件的筛选
  * per rule alert filtering and thresholding - 每个规则的警报进行过滤和和阈值设置
  * global alert filtering and thresholding - 全局警报过滤和阈值设置
  * per host/subnet thresholding and rate limiting settings - 每个主机/子网阈值和速率限制设置

### Suricata规则集

|规则集名称|资源地址|
|:-------------:|-----|
|Emerging Threats Ruleset|Suricata默认规则集.BSD协议开源 由安全社区维护 https://rules.emergingthreats.net/open/suricata/rules/|
|Emerging Threats Pro Ruleset|收费规则集.由Proofpoint/ET研究团队维护|
|Suricata PT Open Ruleset|[ptresearch/AttackDetection](https://github.com/ptresearch/AttackDetection)|

### Suricata规则类型

ET官方对规则类型的[解释说明](https://doc.emergingthreats.net/bin/view/Main/EmergingFAQ#What_is_the_general_intent_of_ea)

|规则名|更新频率|描述|
|:-------------:|-----|-----|
|Attack-Response Rules||捕获成功攻击后的响应. 如"id=root" 或 错误信息|
|BotCC Rules||僵尸网络及其C2主机. |
|Compromised Rules|daily|已知被感染主机. 因为会明显增加传感器的负载 所以建议只使用BotCC规则|
|DOS Rules||inbound DOS 以及 outbound指标(IoC)|
|DROP Rules|daily|主要是专业垃圾邮件发送者|
|DShield Rules||[DShield](http://www.dshield.org)公布的顶级攻击者|
|Exploit Rules||检测exploits如windows exploit. 而sql注入等漏洞的检测不在这里|
|Game Rules||检测网络游戏(无威胁的流量) 如Starcraft等|
|Malware Rules||恶意软件规则 最重要的规则|
|P2P Rules||检测P2P流量 如Bittorrent|
|Scan Rules||检测侦察探测类的流量 如NessusNikto portscanning等|
|Web Rules||检测Web攻击流量 如SQLi等. 负载非常合理|
|Web-SQL-Injection Rules||大型规则集 细化到Web攻击的确切信息. 如 web应用程序 Web服务器 等|
|...|||

### 规则管理

使用Suricata-Update对Suricata的规则集进行统一管理，比如更新、启用、停用等

[suricata-update documentation](https://suricata-update.readthedocs.io/en/latest/)

* 安装
  * `pip install --pre --upgrade suricata-update`
* 更新Suricata规则集的库
  * `suricata-update`
* 更新一下 更新源
  * `suricata-update update-sources`
* 列出源的列表
  * `suricata-update list-sources`
* 启用某规则集
  * `suricata-update enable-source ptresearch/attackdetection` `suricata-update`
* 列出已开启的源
  * `suricata-update list-enabled-sources`
* 关闭某个源
  * `suricata-update disable-source et/pro`
* 删除某个源
  * `suricata-update remove-source et/pro`
  
