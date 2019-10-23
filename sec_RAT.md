### 远程控制 - 分类

* 传统远控
  * 针对个人电脑
  * 网络协议单一
  * 几乎无内网渗透能力

* 现代远控
  * 针对大中型网络
  * 功能更多更强，尤其是内网横向渗透方面(oneliner/网络协议/内网渗透/bypassuac...)
  * 可生成多种形式的payload 尤其是shellcode形式

### 远程控制 - 传统远程控制

2008年在台式机上使用远程控制软件，即RAT(Remote Administration Tool)。
（当时一些远控软件名字：灰鸽子远程控制(delphi)、极光远控、Ghost、白金远控等）

当时情况：
* 设备：台式电脑（主机，显示器，键盘，鼠标，音响，话筒，摄像头）
* 系统：windows XP
* 网络：宽带入户，电脑主机插入网线，直接是外网IP，但不是固定ip。

现在情况：
* 设备：台式电脑、笔记本、手机、智能设备等。
* 系统：windows、Mac、iOS、andorid等。
* 网络：多种设备通过路由器连接外网，设备默认只有内网ip。

**基本概念**

远程控制技术：使用主控端软件，通过网络，去控制 另一些电脑（被控端）。

主控端：“黑客”电脑上的主控软件，用来查看、管理、操作 被控端电脑。主控端软件 可生成 客户端程序（谁打开谁被控）。

生成客户端过程：填写“黑客”的必填信息（从而让被控端被控后能找到主控端），通过“客户端程序模板”生成属于这位“黑客”的可执行文件。

被控端：被控制的电脑。

远程控制功能：主控端可以对被控端进行很多操作，如：
* 远程屏幕(可只查看，也可进行鼠标键盘操作)
* CMD操作(MS-Dos模拟，执行windows cmd命令)
* 文件管理(下载被控端的文件、上传文件至被控端、删除被控端文件、运行、重命名等)
* 进程管理(结束进程)
* 键盘记录(记录在各个窗口下的按键记录)
* 远程音频(接受被控端麦克风传来的声音，也可以和被控端对话)
* 远程视频(开启被控端摄像头传回实时视频，可以保存截图)
* 注册表管理(操作被控端的注册表,就像操作本地注册表一样)
* 命令广播(即对批量被控端执行某些操作，常见的是：关机、重启、打开网页，筛选符合条件的机等）

**使用过程**

举例如下。
* 主控端软件：打开软件，点击“生成客户端程序”按钮，填写必要的选项，如设置动态域名为xxx.3322.org（此域名是主控端“黑客”拥有的，“黑客”可设置该域名经过dns解析后的ip为“黑客”自己的ip），最后点击生成即可生成客户端程序（1.exe）。
* 被控过程：小白打开该客户端程序，即可执行文件1.exe后，解析域名xxx.3322.org后得到了黑客当前的外网ip，即可主动连接到黑客ip的某端口，该小白的电脑被控。此时主控端语音提示“有主机上线请注意！”。
* 自动上线方式：DNS解析域名、固定IP等。（此例为DNS解析域名）

### 远程控制 - 现代远控RAT_for_Windows

以下RAT都仅针对Windows目标进行后渗透(post exploitation)、Windows域渗透

|名称|属性|攻击者环境|描述|
|:-------------:|--|--|-----|
|Cobalt Strike[非开源]|Java|/|/|
|[Empire](https://github.com/EmpireProject/Empire)|python|Kali/Debian/Ubuntu|#域渗透 #RAT 域渗透利器Empire is a post-exploitation framework|
|[QuasarRAT](https://github.com/quasar/QuasarRAT)|C#|windows|#RAT 传统远控 Remote Administration Tool for Windows|
|[NYAN-x-CAT/Lime-RAT](https://github.com/NYAN-x-CAT/Lime-RAT)|VB|windows|#RAT 额外功能:勒索(加密文件)、xmr挖矿、DDOS|
|--|--|--|-----|
|[CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)|python+powershell|Kali/Debian/Ubuntu/MacOS[Installation](https://github.com/byt3bl33d3r/CrackMapExec/wiki/Installation)| 2k★ #域渗透 域渗透利器|
|[sRDI](https://github.com/monoxgas/sRDI)|powershell|windows|Shellcode implementation of Reflective DLL Injection. Convert DLLs to position independent shellcode|

* Cobalt Strike资料
  * [渗透利器Cobalt Strike - 第1篇 功能及使用 - 先知社区](https://xz.aliyun.com/t/3975)
  * [渗透利器Cobalt Strike - 第2篇 APT级的全面免杀与企业纵深防御体系的对抗 - 先知社区](https://xz.aliyun.com/t/4191)

* Windows域渗透
  * [Windows domain - Wikipedia](https://en.wikipedia.org/wiki/Windows_domain)
  * [Domain controller - Wikipedia](https://en.wikipedia.org/wiki/Domain_controller)
