### tcpdump

网络分析工具
network analysis tool

对于希望彻底了解TCP / IP的人来说，必须牢牢掌握这个功能强大的应用程序。

>参考 https://danielmiessler.com/study/tcpdump/#basics

#### 基本选项

```
-i any : Listen on all interfaces just to see if you’re seeing any traffic.
-i eth0 : Listen on the eth0 interface.
-D : Show the list of available interfaces
-n : Don’t resolve hostnames.
-nn : Don’t resolve hostnames or port names.
-q : Be less verbose (more quiet) with your output.
-t : Give human-readable timestamp output.
-tttt : Give maximally human-readable timestamp output.
-X : Show the packet’s contents in both hex and ascii.
-XX : Same as -X, but also shows the ethernet header.
-v, -vv, -vvv : Increase the amount of packet information you get back.
-c : Only get x number of packets and then stop.
-s : Define the snaplength (size) of the capture in bytes. Use -s0 to get everything, unless you are intentionally capturing less.
-S : Print absolute sequence numbers.
-e : Get the ethernet header as well.
-q : Show less protocol information.
-E : Decrypt IPSEC traffic by providing an encryption key.
```

#### 表达式Expressions


表达式用来筛选流量

* 主要有3种类型的表达式
  * Type - type `host net port`
  * Direction - dir `src dst 二者都有`
  * Protocol - proto `tcp udp icmp ...`

#### 例子Examples

```
#通过所有接口（all interfaces）查看基本通讯
tcpdump -i any


#指定接口（specific interface）查看特定接口
tcpdump -i eth0


#原始输出视图 - raw output view
#Verbose output, with no resolution of hostnames or port numbers, absolute sequence numbers, and human-readable timestamps.
tcpdump -ttttnnvvS


#按ip查找流量 - find traffic by ip
显示来自1.2.3.4的流量，包含src和dst

tcpdump host 1.2.3.4
```
