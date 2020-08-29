### 信息搜集/应急响应

#### 用户账户

```
# 获取用户账户名(user accounts) - 全部账户
dscl . list /Users UniqueID


# 获取用户账户名(user accounts) - 排除以下划线开头的账户(通常这些账户是系统使用 但攻击者也可以创建下划线开头的账户)
dscl . list /Users UniqueID | grep -v ^_


# 获取当前正处于活跃状态(active)的账户
w


# last命令
# 输出users和ttys上次 登录/关机/重启 的时间点(indicate last logins of users and ttys)
# 默认顺序 时间最近的最靠前


last命令 列出指定的users/ttys/hosts的会话.
输出的每一行都包含: 用户名, 执行会话的tty, 任何主机名, 会话的开始时间, 会话的停止时间, 会话的持续时间.
如果会话仍在继续,或由于crash或shutdown而中断, last将这样表示.
Last will list the sessions of specified users, ttys, and hosts, in reverse time order. 
Each line of output contains the user name, the tty from which the session was conducted, any hostname, the start and stop times for the session, and the duration of the session.  If the session is still continuing or was cut short by a crash or shutdown, last will so indicate.

# 结果如
# root      console                   Tue May 21 11:22 - shutdown  (00:01)
```


#### 权限维持 - LaunchAgent

```
# macOS上持续存在的最常见方式为通过 LaunchAgent 实现自启动
# 用户LaunchAgents文件夹 - Mac上的每个用户都能在自身账户的文件夹`Library/LaunchAgents`中指定每次用户登录时运行的代码
# 检查当前用户的LaunchAgents文件夹
cd ~/Library/LaunchAgents

# 计算机级别的LaunchAgents文件夹 - 可以为登录的所有用户运行代码 系统完整性保护(System Integrity Protection) 默认禁止访问该文件夹
# LaunchAgents下的属性列表文件(property list files) 后缀为`.plist`  内容为xml格式
```

#### 权限维持 - LaunchDaemons

LaunchDaemons只存在于"计算机级"(computer level) 和 "系统级"(system level), 并且在技术上是为"不与用户交互的代码"保留的. 这对于恶意软件来说是完美的.
对于攻击者来说, 需要"管理员级别的权限"(administrator level privileges), 才能将一个daemon(守护程序)放到目录`/Library/LaunchDaemons`
然而, 由于大多数Mac用户也是管理员用户, 并且习惯了在软件安装组件时提供授权, 所以这个门槛并不是很高.

LaunchDaemons only exist at the computer and system level, and technically are reserved for persistent code that does not interact with the user – perfect for malware.
The bar is raised for attackers as writing a daemon to `/Library/LaunchDaemons` requires administrator level privileges.
However, since most Mac users are also admin users and habitually provide authorisation for software to install components whenever asked, the bar is not all that high.

```
# 文件夹 /Library/LaunchDaemons (该文件夹受SIP保护) 中的plist文件中指定的程序, 会在启动时 和 每个用户登录时 启动(且以root权限运行).

# 查看该文件夹下的plist文件
cd /Library/LaunchDaemons
ls -al
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

#### 权限维持 - LaunchAgent

因为 很常用的权限维持方式.

```
# 用户级的LaunchAgents文件夹(user LaunchAgents)
# macOS上的每一个用户都可以有自己的LaunchAgents文件夹 该用户每次登录时运行代码.
/Users/xxx/Library/LaunchAgents

# 计算机级(computer level)
# 在计算机级也存在一个LaunchAgents文件夹 所有用户每次登录时运行代码.

# 系统级(system level)
# 还有一个LaunchAgents文件夹 由macOS本身使用 默认情况(未关闭SIP)不会被恶意软件更改.
```

#### 权限维持 - Profiles

参考Apple官方文档 [Configuration Profile Reference](https://developer.apple.com/business/documentation/Configuration-Profile-Reference.pdf)

```
# 查看文件夹 该文件夹可能不存在
cd /Library/Managed\ Preferences
```
#### 权限维持 - Login Items

* Login Items
  * 通过"系统偏好设置"(System Preferences.app) 可查看 "登录项"(Login Items).
  * 哪些应用程序包含一个"包内的登录项"(bundled login item). 使用[脚本](https://github.com/1135/Applescript_demos/blob/master/Enumerate_All_User_Login_Items.applescript) 通过分析这个文件查看. `/Library/Application Support/com.apple.backgroundtaskmanagementagent/backgrounditems.btm`


一种较新的机制使任何"已安装的应用程序"(installed application)都可以在登录时启动自己: 只需在其自己的"包"(bundle)中包含一个"登录项"(Login Item).
虽然这种机制的目的是为合法的开发者提供的, 为了实现通过应用程序的用户界面来控制"登录项"(Login Item). 

A newer mechanism makes it possible for any installed application to launch itself at login time simply by including a Login Item in its own bundle.
While the intention of this mechanism is for legitimate developers to offer control of the login item through the app’s user interface.

#### 权限维持 - Kexts

Kexts(Kernel extensions)内核扩展.

因为它相对难以创建, 却可以被轻松删除, 通常少见这种方式.

开源键盘记录程序 https://github.com/SlEePlEs5/logKext

#### 权限维持 - AppleScript

原理:如利用Mail规则实现持久化,通过在向受害者发送带有特定主题的电子邮件后触发后门

检测方法 查看`ubiquitous_SyncedRules.plist`文件和`SyncedRules.plistiCloud`文件，确认是否存在可疑的邮件规则
```
# 查看调用了AppleScripts的邮件规则
grep -A1 "AppleScript" ~/Library/Mail/V6/MailData/SyncedRules.plist
```

#### 权限维持 - Periodics

Periodics are system scripts that are generally used for maintenance and run on a daily, weekly and monthly schedule.

Periodics是 按 日/周/月 定期运行的系统脚本, 通常用于维护.

```
cd /etc/periodic
```

会发现有3个文件夹: daily monthly weekly


#### 权限维持 - LoginHooks和LogoutHooks

```
# 当用户登录或注销时运行的代码(该方法在macOS Mojave下仍然有效)
sudo defaults read com.apple.loginwindow
```

#### 权限维持 - At Jobs

查看Jobs.

```
cd /var/at/jobs
```

Jobs are prefixed with the letter a and have a hex-style name.

Jobs以字母`a`开头, 并具有十六进制风格的名称.


#### 权限维持 - Emond

苹果可能已经放弃了其他机制的开发，但在macOS 10.14 Mojave上Emond(The Event Monitor)仍然可用.

Apple introduced a logging mechanism called emond. It appears emond was never fully developed and development may have been abandoned by Apple for other mechanisms, but it remains available even on macOS 10.14 Mojave.

```
cd /private/var/db/emondClients
ls -al
# 正常情况下该文件夹下不会有文件.
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

#### 进程信息 - Investigate Running Processes

调查正在运行的进程

```
ps -axo user,pid,ppid,%cpu,%mem,start,time,command
# 作用: (root权限执行)可获取所有进程的信息 pid ppid cpu mem start time command

# 输出
USER               PID  PPID  %CPU %MEM STARTED      TIME COMMAND
root                 1     0   0.0  0.1 19Jun19 161:37.36 /sbin/launchd

# 注意
# PPID为1 表明该进程的父进程是 系统进程 /sbin/launchd
# PPID不为1 表明该进程的父进程 不是系统进程/sbin/launchd   应该关注一下这些进程
```

```
lsappinfo list

# 输出应用程序的具体信息
# 包括:
# bundle identifier 标志符
# executable path 可执行文件的路径
# pid
# launch time 运行的时间点

# 如
10) "聚焦" ASN:0x0-0x2c02c:
    bundleID="com.apple.Spotlight"
    bundle path="/System/Library/CoreServices/Spotlight.app"
    executable path="/System/Library/CoreServices/Spotlight.app/Contents/MacOS/Spotlight"
    pid = 7129 type="UIElement" flavor=3 Version="2076.7" fileType="APPL" creator="????" Arch=x86_64
    checkin time = 2020/08/19 14:02:45 ( 2 days, 57 minutes, 15.2166 seconds ago )
```

```
launchctl print user/501
# 可替换501为任意用户的UID

launchctl print system
# To target the system-wide domain.
```

### 文件信息 - Investigate Open Files


调查打开了的文件

```
```

* 参考 [A Guide to macOS Threat Hunting and Incident Response](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ&lb-mode=overlay)
