### 简介

Command Injection(命令注入)

### 漏洞危害
* 能够执行系统命令的地方 如果传入的实参用户可控 则可实现命令执行
  * 编程语言的函数 `Runtime.exec` ...
  * 脚本 `.sh` `.py` ...
  * ...

### 漏洞检测

* 自动化测试
  * [commixproject/commix: Automated All-in-One OS command injection and exploitation tool.](https://github.com/commixproject/commix)
  * burp

### SDL - 防御与修复方案

* 白名单方法 - 使传入的实参用户不可控:固定为几个可选项 并用数字代替可选项
