### 简介


https://www.explainshell.com/explain/8/iptables

* iptables - 针对ipv4  无法对ipv6做限制
* ip6tables - 针对ipv6

### 原理


OSI参考模型(七层模型) 从逻辑理论上做分层


Iptables主要工作在: 表示层 会话层 传输层

```
应用层(Application layer) 
功能 - 与实现通信功能的软件应用程序进行交互
协议数据单元 - Data

表示层(Presentation layer)
功能 - 网络服务和应用程序之间的数据转换:字符编码(character encoding),数据压缩(data compression),加密解密(encryption decryption)
协议数据单元 - Data

会话层(Session layer)
功能 - 管理通信会话
协议数据单元 - Data

传输层(Transport layer)
功能 - 在网络上的各点之间可靠地传输 数据段(data segments) :分段(segmentation),确认(acknowledgement),多路复用(multiplexing)
协议数据单元 - Segment, Datagram

网络层(Network layer)
功能 - 构建和管理多节点网络:寻址(addressing),路由(routing),流量控制(traffic control)
协议数据单元 - Packet

数据链路层(Data Link layer)
功能 - 在已经被物理层连接起来的两个节点之间可靠地传输 数据帧(data frames)
协议数据单元 - Frame

物理层(Physical layer)
功能 - 通过物理介质发送和接收 原始比特流(raw bit streams)
协议数据单元 - Symbol
```


### 常用命令

查看当前的策略`iptables -L`
```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```
