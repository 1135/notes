
### 简介

pwn:通过二进制/系统调用等方式获得目标主机的shell.

### 基本概念


ELF中 实现代码共享(通常以动态库的形式)的关键是PLT表和GOT表
* Global Offset Table(GOT)
  * 定义 - 存储地址的表 (存储在ELF文件的data section中)
  * 功能 - 已经执行的程序在运行时通过GOT表来帮助寻找"全局变量的地址". 因为在编译时无法得知动态库(dynamic libraries)中的全局变量的偏移量,所以需要在运行时从GOT表中读取.
  * 更新 - GOT表在进程引导(process bootstrap)过程中由动态链接器更新.
* Procedure Linkage Table(PLT)
  * 功能 - PLT包含了用于调用共享函数(shared functions)的跳板代码(trampoline code)


### 系统保护机制

|windows|linux|描述|
|:-------------:|--|-----|
|DEP|NX|将数据所在内存页标识为不可执行. 当程序溢出成功尝试执行shellcode会失败(当在数据页面上执行指令时CPU必定抛出异常)|
|ASLR|PIE|地址随机化选项. 即程序每次运行的地址都不同|
|GS protection|Canary|防止栈溢出. 进程重启每次的Canary都不同|

Fortify Source
* linux保护机制
  * Position Independent Code (PIE)
  * NoeXecute (NX)
  * Canary

* windows保护机制
  * Address Space Layout Randomization (ASLR)
  * Data Execution Prevention (DEP)
  * GS protection


```
# 关闭PIE
sudo -s echo 0 > /proc/sys/kernel/randomize_va_space
# 确认关闭
cat /proc/sys/kernel/randomize_va_space
```


### 检测ELF二进制文件的属性

linux下使用[checksec.sh](https://github.com/slimm609/checksec.sh)检测ELF二进制可执行程序的属性/保护机制

用gcc编译的一个ELF二进制程序默认(不加额外的参数改变编译设置)即开启所有的安全措施.


```
# 下载
wget https://github.com/slimm609/checksec.sh/archive/1.11.1.tar.gz
# 解压
tar -xvf 1.11.tar.gz
# 切换目录
cd checksec.sh-1.11.1/

# 检测命令
 ./checksec --file /bin/ls

# 输出结果
./checksec --file /bin/ls
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable  FILE
Partial RELRO   Canary found      NX enabled    No PIE          No RPATH   No RUNPATH   No Symbols      Yes	5		15	     /bin/ls
```


具体解释:

* RELRO
  * Partial RELRO - 对GOT表具有写权限
  * FULL RELRO - 无法修改GOT表
* STACK CANARY
  * Canary found - 无法直接用溢出去覆盖栈中返回地址。要进行绕过:通过改写指针与局部变量、leak canary、overwrite canary
* NX
  * NX enabled - NX(即No-eXecute)表示不可执行,则栈中数据没有执行权限，经常用的call esp或者jmp esp的方法就无法使用. 可以利用ROP这种方法绕过NX enabled
* PIE
  * PIE enabled - 开启地址随机化选项. 即程序每次运行的地址都不同
  * No PIE (0x400000) - 没有开PIE，那么程序的基地址就是括号内的数据0x400000
* RPATH
* RUNPATH
* Symbols
* FORTIFY
  * FORTIFY_SOURCE机制对格式化字符串有2个限制
(1)包含%n的格式化字符串不能位于程序内存中的可写地址。
(2)当使用位置参数时，必须使用范围内的所有参数。所以如果要使用%7$x，你必须同时使用1,2,3,4,5和6
* Fortified
* Fortifiable
* FILE


#### 工具_二进制_debug_pwn

|名称|属性|描述|
|:-------------:|--|-----|
|[gdbgui](https://github.com/cs01/gdbgui)|/|6k★ 基于浏览器可视化gdb调试 [下载安装ubuntu14.04+](https://gdbgui.com/downloads.html)|
|[peda](https://github.com/longld/peda)|python|6k★ Python Exploit Development Assistance for GDB(包含checksec等)|
|[pwntools](https://github.com/Gallopsled/pwntools)|python2.7(best Ubuntu)|4k★ CTF framework and exploit development library(写exp/poc的利器)|
|[ROPgadget](https://github.com/JonathanSalwan/ROPgadget)|python|在二进制文件中搜索gadgets以便进行ROP利用|
|[one_gadget](https://github.com/david942j/one_gadget)|Ruby|The best tool for finding one gadget RCE in libc.so.6  如快速寻找libc中的调用exec('bin/sh')的位置|
|[heap-viewer](https://github.com/danigargu/heap-viewer)|python| [IDA Pro plugin] examine the glibc heap, focused on exploit development|
|[cea-sec/miasm](https://github.com/cea-sec/miasm)|python|2k★ (PE/ELF)Reverse engineering framework|

|名称|属性|描述|
|:-------------:|--|-----|
|http://syscalls.kernelgrok.com/|syscalls|系统调用参考 主要用sys_execve|
|http://shell-storm.org/shellcode/|shellcode|各种平台下的shellcode包括:BSD/Cisco/FreeBSD/Linux/OSX/Windows|


#### 工具_二进制_fuzz

|名称|属性|描述|
|:-------------:|--|-----|
|[halfempty](https://github.com/googleprojectzero/halfempty)|C|[projectZero] testcase minimization tool.|
