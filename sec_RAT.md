### 远程控制 - 分类

* 传统远控
  * 针对个人电脑
  * 网络协议单一
  * 几乎无内网渗透能力

* 现代远控
  * 针对大中型网络
  * 功能更多更强，尤其是内网横向渗透方面(oneliner/网络协议/内网渗透/bypassuac...)
  * 可生成多种形式的payload 尤其是shellcode形式


### 现代远控 RAT_for_Windows

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
