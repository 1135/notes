### 信息搜集/应急响应

#### 用户账户
```
# 获取用户账户名(user accounts) - 全部账户
dscl . list /Users UniqueID

# 获取用户账户名(user accounts) - 排除以下划线开头的账户(通常这些账户是系统使用 但攻击者也可以创建下划线开头的账户)
dscl . list /Users UniqueID | grep -v ^_

# 获取正处于活跃状态(active)的账户
w

# 获取用户登录、关机、重启的时间(默认顺序为时间最近的最靠前)
last
# 结果如
# root      console                   Tue May 21 11:22 - shutdown  (00:01)
```


#### 持久化 - LaunchAgent

```
# macOS上持续存在的最常见方式为通过 LaunchAgent 实现自启动
# 用户LaunchAgents文件夹 - Mac上的每个用户都能在自身账户的文件夹`Library/LaunchAgents`中指定每次用户登录时运行的代码
# 检查当前用户的LaunchAgents文件夹
cd ~/Library/LaunchAgents

# 计算机级别的LaunchAgents文件夹 - 可以为登录的所有用户运行代码 系统完整性保护(System Integrity Protection) 默认禁止访问该文件夹
# LaunchAgents下的属性列表文件(property list files) 后缀为`.plist`  内容为xml格式
```

#### 持久化 - LaunchDaemons

```
# 文件夹 /Library/LaunchDaemons 中的plist文件中指定的程序可以自启动(root权限运行) 该文件夹受SIP保护
# 查看该文件夹下的plist文件
cd /Library/LaunchDaemons
```

如果合法的LaunchDaemons指向了未签名的代码/程序(它们可被替换为恶意程序)
```
# 例如Wireshark
cat /Library/LaunchDaemons/org.wireshark.ChmodBPF.plist
# 输出为
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>org.wireshark.ChmodBPF</string>
	<key>RunAtLoad</key>
	<true/>
	<key>Program</key>
	<string>/Library/Application Support/Wireshark/ChmodBPF/ChmodBPF</string>
</dict>
</plist>

# 路径上的程序未进行签名 攻击者可以使用高权限把白程序ChmodBPF替换为恶意程序
/Library/Application Support/Wireshark/ChmodBPF/ChmodBPF
```

#### 持久化 - Profiles

参考Apple官方文档 [Configuration Profile Reference](https://developer.apple.com/business/documentation/Configuration-Profile-Reference.pdf)

```
# 查看文件夹
cd /Library/Managed\ Preferences
```

#### 持久化 - Kexts

Kexts(Kernel extensions)内核扩展:相对难以创建且可以轻松删除

开源键盘记录程序 https://github.com/SlEePlEs5/logKext

#### 持久化 - AppleScript

原理:如利用Mail规则实现持久化,通过在向受害者发送带有特定主题的电子邮件后触发后门

检测方法 查看`ubiquitous_SyncedRules.plist`文件和`SyncedRules.plistiCloud`文件，确认是否存在可疑的邮件规则
```
# 查看调用了AppleScripts的邮件规则
grep -A1 "AppleScript" ~/Library/Mail/V6/MailData/SyncedRules.plist
```

#### 持久化 - LoginHooks和LogoutHooks

```
# 当用户登录或注销时运行的代码(该方法在macOS Mojave下仍然有效)
sudo defaults read com.apple.loginwindow
```

#### Open Ports & Connections

Apple自身使用的TCP和UDP端口 [TCP and UDP ports used by Apple software products - Apple Support](https://support.apple.com/en-us/HT202944)

```
# 查看services "正在监听"或"已建立连接的"具体port
netstat -na | egrep 'LISTEN|ESTABLISH'

# 列出"连接"所使用的文件 进程pid号
# list all files with an open IPv4, IPv6 or HP-UX X25 connection
lsof -i

# 查看进程的启动命令(CMD)
ps -p <pid>

# 如
  PID TTY           TIME CMD
  103 ??         4:08.03 /System/Library/CoreServices/loginwindow.app/Contents/MacOS/loginwindow console
```

#### 进程信息

```
# root权限下 获取所有进程的完整列表
ps -axo user,pid,ppid,%cpu,%mem,start,time,command

# 如
USER               PID  PPID  %CPU %MEM STARTED      TIME COMMAND
root                 1     0   0.0  0.1 19Jun19 161:37.36 /sbin/launchd
```

> 参考 https://www.sentinelone.com/blog/malware-hunting-macos-practical-guide/
