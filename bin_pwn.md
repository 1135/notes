### 简介

pwn:通过二进制/系统调用等方式获得目标主机的shell.

### 学习资料

* [Nightmare](https://guyinatuxedo.github.io/index.html) 基于CTF学习 binary exploitation / reverse engineering

### 在线工具

* Online x86 / x64 Assembler and Disassembler
  * https://disasm.pro/  支持多种Arch 常用格式可互相转换格式 很方便 `\x48\xC7\xC0\x00  <-->  48 C7 C0 00 `
  * https://defuse.ca/online-x86-assembler.htm#disassembly2

### 基本概念

#### Compiling

编译过程 如`c code ---compiler--> assembly code(ran on the processor)`

* assembly code architectures
  * `x64` - 64 bit ELF (Executable Linux File).
  * `x86` - 32 bit ELF (Executable Linux File).
  * ...

Different types of processors can run different types of assembly code architectures.

#### Registers

Registers are essentially places that the processor can store memory.

寄存器是处理器中存储内存的地方(可以理解为 处理器可以在Registers中存储信息)


there are different sizes for registers.

* 寄存器有不同大小:
  * `x64`架构 - 使用 8 byte registers 如`rax, eax, ax, al ...`
  * `x86`架构 - 常用 4 byte registers(这是x86可用的最大size的寄存器) 如`ebp, esp, eip ...`

```
字 
a  word = 2 bytes of data. 
a dword = 4 bytes of data.
a qword = 8 bytes of data.
```


```
寄存器有不同大小的原因是,随着技术的进步,可以在寄存器中存储更多数据

+-----------------+---------------+---------------+------------+
| 8 Byte Register | Lower 4 Bytes | Lower 2 Bytes | Lower Byte |
+-----------------+---------------+---------------+------------+
|   rbp           |     ebp       |     bp        |     bpl    |
|   rsp           |     esp       |     sp        |     spl    |
|   rip           |     eip       |               |            |
|   rax           |     eax       |     ax        |     al     |
|   rbx           |     ebx       |     bx        |     bl     |
|   rcx           |     ecx       |     cx        |     cl     |
|   rdx           |     edx       |     dx        |     dl     |
|   rsi           |     esi       |     si        |     sil    |
|   rdi           |     edi       |     di        |     dil    |
|   r8            |     r8d       |     r8w       |     r8b    |
|   r9            |     r9d       |     r9w       |     r9b    |
|   r10           |     r10d      |     r10w      |     r10b   |
|   r11           |     r11d      |     r11w      |     r11b   |
|   r12           |     r12d      |     r12w      |     r12b   |
|   r13           |     r13d      |     r13w      |     r13b   |
|   r14           |     r14d      |     r14w      |     r14b   |
|   r15           |     r15d      |     r15w      |     r15b   |
+-----------------+---------------+---------------+------------+
```


Here is a list of the `x64` registers, and what their common use cases are.

下面是`x64`寄存器的列表,以及它们的常用用例.

```
rbp: Base Pointer, points to the bottom of the stack
rsp: Stack Pointer, points to the top of the stack
rip: Instruction Pointer, points to the instruction to be executed

-------
General Purpose Registers
These can be used for a variety of different things
rax:
rbx:
rcx:
rdx:
rsi:
rdi:
r8:
r9:
r10:
r11:
r12:
r13:
r14:
r15:
```
In x64 linux arguments to a function are passed via registers. The first few args are passed by these registers:

在x64 linux中,一个函数的参数通过寄存器传递. 前几个参数是通过这些寄存器传递的:
```
rdi:    First Argument
rsi:    Second Argument
rdx:    Third Argument
rcx:    Fourth Argument
r8:     Fifth Argument
r9:     Sixth Argument
```

#### Stacks

我们要处理的最常见的内存区域之一就是"栈"(Stacks).

"局部变量"(local variables)保存在 stack.

For instance, the local variable `x` is stored in the stack:
```c
#include <stdio.h>

void main(void)
{
    int x = 5;
    puts("hi");
}
```

`x64`架构下: 可以看到变量`x`其实存储在`rbp-0x4`
```
0000000000001135 <main>:
    1135:       55                      push   rbp
    1136:       48 89 e5                mov    rbp,rsp
    1139:       48 83 ec 10             sub    rsp,0x10
    113d:       c7 45 fc 05 00 00 00    mov    DWORD PTR [rbp-0x4],0x5
    1144:       48 8d 3d b9 0e 00 00    lea    rdi,[rip+0xeb9]        # 2004 <_IO_stdin_used+0x4>
    114b:       e8 e0 fe ff ff          call   1030 <puts@plt>
    1150:       90                      nop
    1151:       c9                      leave  
    1152:       c3                      ret    
    1153:       66 2e 0f 1f 84 00 00    nop    WORD PTR cs:[rax+rax*1+0x0]
    115a:       00 00 00
    115d:       0f 1f 00                nop    DWORD PTR [rax]
```

因为stack的数据结构是LIFO(后进先出),所以从1个stack中add或remove值的唯一方式是使用`push`和`pop`.   但我们可以在stack上引用值.


Now values on the stack are moved on by either pushing them onto the stack, or popping them off. That is the only way to add or remove values from the stack (it is a LIFO data structure).  However we can reference values on the stack.


* a Stack
  * `push` - `将1个"值"(不一定保存在1个"寄存器"中) ----写入到---> 1个stack`. pushing a value (not necessarily stored in a register) means writing it to the stack.
  * `pop` - `将这个stack"栈顶"的内容 ----放回到---> 1个"寄存器"中`. popping means restoring whatever is on top of the stack into a register.
  * ...

简单介绍下`push`和`pop`
```
push 0xdeadbeef      ; push a value to the stack  将一个"值" 写入到 这个栈.  现在这个stack"栈顶"的内容 为 0xdeadbeef
pop eax              ; eax is now 0xdeadbeef   将"栈顶"的内容(whatever) 放回到 1个具体的寄存器eax中 现在寄存器eax中的内容 为 0xdeadbeef

; swap contents of registers 交换2个寄存器中的内容
push eax
mov eax, ebx
pop ebx
```

The exact bounds of the stack is recorded by two registers, `rbp` and `rsp`.  

* x64架构下 一个stack的精确边界被 寄存器`rbp` 和 寄存器`rsp`记录着:
  * `rbp` - "栈底指针"(Base Pointer)寄存器. `rbp` points to the bottom of the stack.
  * `rsp` - "栈顶指针"(Stack Pointer)寄存器. `rsp` points to the top of the stack.
  * `rip` - "指令指针"(Instruction Pointer)寄存器. `rip` points to the instruction to be executed. 总是指向下一个"指令"的内存地址.

#### Flags

https://guyinatuxedo.github.io/01-intro_assembly/assembly/index.html#flags

在包含flags的寄存器中,一个flag是这个寄存器的特定bit.设置或未设置它,都会表示某些意义.下面是flags列表.

There is one register that contains flags. A flag is a particular bit of this register. If it is set or not, will typically mean something. Here is the list of flags.

```
00:     Carry Flag
01:     always 1
02:     Parity Flag
03:     always 0
04:     Adjust Flag
05:     always 0
06:     Zero Flag
07:     Sign Flag
08:     Trap Flag
09:     Interruption Flag     
10:     Direction Flag
11:     Overflow Flag
12:     I/O Privilege Field lower bit
13:     I/O Privilege Field higher bit
14:     Nested Task Flag
15:     Resume Flag
```

There are other flags then the one listed, however we really don't deal with them too much (and out of these, there are only a few we actively deal with).

If you want to hear more about this, checkout: https://en.wikibooks.org/wiki/X86_Assembly/X86_Architecture

#### Instructions

* 常见指令(common instructions)
  * [mov](#mov)
  * [dereference](#dereference)
  * [lea](#lea)
  * [add](#add)
  * [sub](#sub)
  * ...

##### mov

`mov` - The move instruction just moves data from one register to another.

mov指令 只将数据 从一个寄存器 移动到 另一个寄存器.

```
将数据从rdx寄存器移至rax寄存器.
mov rax, rdx
```

##### dereference

If you ever see brackets like `[]`, they are meant to dereference, which deals with pointers.

如果你曾经看到过方括号(brackets)即`[]` 它的作用是"取消引用"(dereference),它处理指针.

A pointer is a value that points to a particular memory address (it is a memory address). Dereferencing a pointer means to treat a pointer like the value it points to.

一个指针是一个指向"特定内存地址"的值. 取消引用一个指针(取消指针的引用),意味着将"这个指针"当作"这个指针所指向的值".

```
mov rax, [rdx]
```

Will move the value pointed to by `rdx` into the `rax` register. 

寄存器`rdx`中保存了一个"指针". 将寄存器`rdx`中的那个指针所指向的值,移动到`rax`寄存器中. 


On the flipside:

反过来:

```
mov [rax], rdx
```

Will move the value of the `rdx` register into whatever memory is pointed to by the `rax` register. The actual value of the `rax` register does not change.

将`rdx`寄存器中的值 移动到 `rax`寄存器所指向的"任何内存"(whatever memory)中. `rax`寄存器中的值不变.


##### lea

`lea` - The lea instruction calculates the address of the second operand, and moves that address in the first.

lea指令 计算 "源"操作数(即第2个操作数)的"地址" 并将这个"地址" 赋值到 "目的"操作数(即第1个操作数).

```
lea rdi, [rbx+0x10]
```

This will move the address of `rbx+0x10` into the `rdi` register.

这会把`rbx+0x10`的"地址" 赋值到 `rdi`寄存器中.

##### add

`add` - This just adds the two values together, and stores the sum in the first argument.

这只是将2个值相加,并将sum存储在第1个参数中.

```
add rax, rdx
```

That will set `rax` equal to `rax + rdx`

这会使`rax`的值等于`rax + rdx`

##### sub

This value will subtract the second operand from the first one, and store the difference in the first argument.

第1个操作数减去第2个操作数，并将差值存储在第1个参数中.

```
sub rsp, 0x10
```

This will set the `rsp` register equal to `rsp - 0x10`

这会使`rsp`寄存器的值等于`rsp - 0x10`

#### Other

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
