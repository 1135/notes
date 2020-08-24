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

wireshark更多用法 [Wireshark Tutorial and Tactical Cheat Sheet](https://hackertarget.com/wireshark-tutorial-and-cheat-sheet/)

恶意软件分析工具与资源 [awesome-malware-analysis](https://github.com/rshipp/awesome-malware-analysis)

### filter

Wireshark/tshark都有2个filter.

* Capture filter - 设置捕获条件 实现只捕获想要捕获的流量
  * Wireshark 抓包之前 直接在GUI上的`capture filter`框输入筛选表达式
  * tshark 使用`-f`指定display filter的筛选表达式 如`tshark -f "predef:MyPredefinedHostOnlyFilter"`
* Display filter - 设置显示条件 实现只显示出想要显示的流量
  * Wireshark 抓包之后 直接在GUI上的`display filter`框输入筛选表达式
  * tshark 使用`-Y`指定display filter的筛选表达式 如`tshark -r 1.pcap -Y 'http.request.method==GET'`

### filter syntax

参考 [wireshark-filter - The Wireshark Network Analyzer 3.2.3](https://www.wireshark.org/docs/man-pages/wireshark-filter.html)

* 逻辑条件组合(适用于对各种筛选条件进行逻辑组合)
  * 与 `and`
  * 或 `or`
  * 非 `!(表达式)` `not(表达式)` **注意格式**  如 `not(ip.addr == 192.168.1.222)`
  * 等于`eq` `==`
  * 包含某字符串 `contains`


* 以下语法可用于 Capture filter 和 Display filter
  * 筛选条件 - 常用
    * 只关注某个端口的流量 如只关注53端口(TCP或UDP) DNS traffic `udp.port == 53 || tcp.port == 53`
    * 关注域名解析 `(dns.qry.name contains microsoft)`
  * 筛选条件 - ip地址
    * 只关注某IP的流量(to&from) `ip.addr == 192.168.10.1`
    * 只关注某网段范围内的流量(to&from) `ip.addr == 10.17.3.0/24`
    * 只关注某网段范围内的流量(from) `ip.src == 10.17.3.0/24`
  * 筛选条件 - TCP端口 (UDP也类似)
    * 只关注某TCP端口 源端口+目的端口的流量 `tcp.port==80`
    * 只关注某TCP端口 目的端口的流量 `tcp.dstport==59602`
    * 只关注某TCP端口 源端口的流量 `tcp.detport==443`
  * 明文web流量
    * http GET数据包 `http.request.method==GET`
    * http 包的内容(包含header url responseCode ...) `http contains "User-Agent: "`
    * http URL地址特征 `http.request.uri == "/logo.png"`
    * HTTP/2协议的流量(to&from) `http2`
  * 所有非加密的web流量 即: HTTP流量 + HTTPS流量中的`Client Hello`和`Server Hello, Certificate`
    * `(http) or (tls.handshake.type == 1) or (tls.handshake.type == 2)`
    * 注:[TLS/SSL握手过程中的4次通信 都是明文传输 其中前2次都能看到域名](web_x_https_tls.md#qa)
    * 注:[TLS/SSL握手过程中的4次通信 都是明文传输 在此过程存在特征 可生成指纹用于**威胁检测**](web_x_https_tls.md#qa)
  * 筛选条件 - MAC地址
    * MAC地址是20:dc:e6:f3:78:cc的流量 源MAC地址+目的MAC地址 `eth.addr==20:dc:e6:f3:78:cc`
    * 源MAC地址 `eth.src==20:dc:e6:f3:78:cc`
    * 目的MAC地址 `eth.dst==20:dc:e6:f3:78:cc`
  * 筛选条件 - 协议
    * 选中某协议 `tcp` `udp` `dns` `http` `icmp` `ssl` `ssh` `ftp` `smtp` `msnms` `arp` ...
    * 排除某协议 `!ssl` `not ssl`
  * ...

---

更多筛选条件:

Only to show the outgoing packets from the management frame.

```
wlan.fc.type==0
```

To show incoming, outgoing packets through control frame.

```
wlan.fc.type==1
```

To show packets transferred over the data frame.

```
wlan.fc.type==2
```

Association lists the requests.

```
wlan.fc.type_subtype==0
```

Association lists the answers.

```
wlan.fc.type_subtype==1
```

Probe lists requests.

```
wlan.fc.type_subtype==4
```

Lists the probe responses.

```
wlan.fc.type_subtype==5
```

Lists Beacon signals / waves.

```
wlan.fc.type_subtype==8
```

Lists the Authentication requests.

```
wlan.fc.type_subtype==11
```

Lists deauthentication requests.

```
wlan.fc.type_subtype==12
```

Lists the HTTP Get requests.

```
http.request
```

Lists packages for the source or destination mac address.

```
wlan.addr == MAC-Address
```

The source lists packages that have a mac address.

```
wlan.sa == MAC-Address
```

Lists packages that have a target mac address.

```
wlan.da == MAC-Address
```

### 确定设备的信息

* 确定设备的信息(系统信息、IP地址、MAC地址、计算机名、系统帐户名)
  * 方式1 通过HTTP请求中的User-Agent判断设备类型、系统版本. 参考案例: [对恶意软件Dridex的流量分析](https://www.freebuf.com/articles/es/195832.html)
    * Windows NT 5.1：Windows XP
    * Windows NT 6.0：Windows Vista
    * Windows NT 6.1：Windows 7
    * Windows NT 6.2：Windows 8
    * Windows NT 6.3：Windows 8.1
    * Windows NT 10.0：Windows 10
    * Android设备 - User-Agent中通常有系统版本(如Android 7.1.2), 和设备型号(如LM-X210APM)
    * Apple设备 - User-Agent中只能看到系统版本(如iOS 12.1.3)和设备类型(如iPhone/iPad/iPod), 而无法判断设备型号(如iPhone型号)

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
