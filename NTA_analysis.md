### 流量分析

网络流量分析(Network traffic analysis,NTA)是拦截，记录和分析网络流量通信模式以检测和响应"安全威胁"的过程。

NTA is the process of intercepting, recording and analyzing network traffic communication patterns in order to detect and respond to security threats. 

### 流量分析工具

|名称|描述|
|:-------------:|-----|
|Wireshark|图形化 官方文档[Wireshark User’s Guide](https://www.wireshark.org/docs/wsug_html_chunked/)|
|tshark|命令行 官方文档[tshark](https://www.wireshark.org/docs/man-pages/tshark.html) |
|[suricata](https://github.com/OISF/suricata) |强大的网络威胁检测引擎 real time IDS, IPS, network security monitoring (NSM) and offline pcap processing.|
[security-onion](https://github.com/Security-Onion-Solutions/security-onion)|一个liunx系统镜像 内含 Elasticsearch, Logstash, Kibana, Snort, Suricata, Bro, Wazuh, Sguil, Squert, CyberChef, NetworkMiner, and many other security tools. |
|Zeek(Bro)| Network Security Monitor https://www.zeek.org/|

恶意软件分析工具与资源 [awesome-malware-analysis](https://github.com/rshipp/awesome-malware-analysis)

### Wireshark/tshark - capture filter


* 通过capture filter设置捕获条件 从而只捕获想要捕获的流量
  * Wireshark 抓包前 直接在capture filter框输入筛选表达式
  * tshark 使用`-f`指定display filter的筛选表达式 如`tshark -f "predef:MyPredefinedHostOnlyFilter"`
* 常用的捕获表达式 - Capture filter expression
  * 关注某主机 只捕获该ip的流量 `host 172.18.5.4`
  * 关注网段 只捕获该ip范围内的"进+出"流量(to&from) `net 192.168.0.0/24` 或`net 192.168.0.0 mask 255.255.255.0`
  * 关注网段 只捕获该ip范围内的"出"流量(from) `src net 192.168.0.0/24`
  * 只捕获指定端口的流量 如DNS (port 53) traffic `port 53`
  * 捕获某主机的非http且非smtp流量 `host www.example.com and not port 80 and not port 25`
  
### Wireshark/tshark - display filter

* 通过display filter设置筛选条件 从而只显示出想要显示的流量
  * Wireshark 抓包后 直接在display filter框输入筛选表达式
  * tshark 使用`-Y`指定display filter的筛选表达式 如`tshark -r 1.pcap -Y 'http.request.method==GET'`

* 常用筛选表达式 - Display filter expression
  * 查看所有web流量 `(http.request) or (ssl.handshake.type == 1)` 查看HTTP请求的URL + HTTPS(SSL/TLS)流量中使用的域名
  * http GET数据包`http.request.method==GET`
  * http 包的内容(包含header url responseCode ...) `http contains "User-Agent: "`
  * http URL地址特征 `http.request.uri == "/logo.png"`
  * 关注域名解析 `(dns.qry.name contains microsoft)`
  * 关注某主机 发到 / 来自 某ip 的数据包 `host 10.3.1.1` `ip.addr == 192.168.10.1`
  
* 逻辑条件组合(适用于对各种筛选条件进行逻辑组合)
  * 与 `and`
  * 或 `or`
  * 非 `!(表达式)` `not(表达式)` **注意格式**
  * 等于`eq` `==`
  * 包含某字符串 `contains`
* 筛选条件 - ip地址
  * 目的ip地址 `ip.dst==192.168.101.8` `dst host 192.168.101.8`
  * 源ip地址`ip.src==1.1.1.1` `src host 1.1.1.1`
  * 发到 / 来自 某ip 的数据包 `host 10.3.1.1` `ip.addr == 192.168.10.1`
  * 发到 / 来自 某域名解析的IP地址的数据包 `host www.1.com`
* 筛选条件 - 端口
  * 源端口 或 目的端口为 80 的数据包`tcp.port==80`
  * 目的端口为 80 的数据包`tcp.dstport==80`
  * 源端口为 80 的数据包`tcp.srcport==80`
* 筛选条件 - MAC地址
  * MAC地址是20:dc:e6:f3:78:cc的数据包  包括 源MAC地址 和 目的MAC地址`eth.addr==20:dc:e6:f3:78:cc`
  * 源MAC地址 `eth.src==20:dc:e6:f3:78:cc`
  * 目的MAC地址 `eth.dst==20:dc:e6:f3:78:cc`
* 筛选条件 - 协议
 * 选中某协议 `tcp` `udp` `dns` `http` `icmp` `ssl` `ftp` `smtp` `msnms` `arp` ...
 * 排除某协议 `!ssl` `not ssl`

### 确定受影响主机的信息

* 确定被感染主机的信息(系统信息、IP地址、MAC地址、计算机名、系统帐户名)
  * [对恶意软件Dridex的流量分析 - FreeBuf互联网安全新媒体平台](https://www.freebuf.com/articles/es/195832.html)
  * 通过HTTP请求中的User-Agent判断设备类型、系统版本
    * Windows NT 5.1：Windows XP
    * Windows NT 6.0：Windows Vista
    * Windows NT 6.1：Windows 7
    * Windows NT 6.2：Windows 8
    * Windows NT 6.3：Windows 8.1
    * Windows NT 10.0：Windows 10
    * Android设备 - User-Agent中通常有系统版本(如Android 7.1.2)和设备型号(如LM-X210APM)
    * Apple设备 - User-Agent中只能看到系统版本(如iOS 12.1.3)和设备类型(如iPhone/iPad/iPod) 而无法判断设备型号(如iPhone型号)

### Wireshark - 插件

[pentesteracademy/patoolkit: PA Toolkit is a collection of traffic analysis plugins focused on security](https://github.com/pentesteracademy/patoolkit)

* WiFi (WiFi network summary, Detecting beacon, deauth floods etc.)
* HTTP (Listing all visited websites, downloaded files)
* HTTPS (Listing all websites opened on HTTPS)
* ARP (MAC-IP table, Detect MAC spoofing and ARP poisoning)
* DNS (Listing DNS servers used and DNS resolution, Detecting DNS Tunnels)检测DNS隧道

### 知名的流量分析博客

[Malware-Traffic-Analysis.net](https://www.malware-traffic-analysis.net/)

此博客专注于 与恶意软件感染相关的网络流量(network traffic related to malware infections)

其中每个博文基本都有相关的流量文件（.pcap files）和 恶意软件样本(malware samples)
 
[Malware-Traffic-Analysis.net - 流量分析练习Traffic Analysis Exercises](https://www.malware-traffic-analysis.net/training-exercises.html)

下载的zip文件解压密码为`infected`
