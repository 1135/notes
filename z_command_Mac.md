# For MacOS High Sierra/Mojave

### 视频音频播放器 - IINA 
强大又极客范的播放器 https://lhc70000.github.io/iina/zh-cn/
安装命令`brew cask install iina`【自用】

### NTFS支持
* mounty 好用免费 点击下载http://enjoygineering.com/mounty/releases/Mounty.dmg
或`brew cask install mounty`
有时无法安全推出移动硬盘，用系统自带“磁盘工具”推出即可。【自用】

* Texera NTFS 2016（￥213.47)收费 据说体验较好
* Paragon NTFS 14（￥139）收费 据说体验较好

### 高效桌面

* HazeOver（专注于一个窗口，其他地方都变暗一些，可调整暗度）【自用】

* Bartender  （把状态栏上的一些应用图标收入一个“省略号图标”）

* Sepctacle （免费，比 Moom 更加简洁易操作的最好的窗口布局工具）


### 软件

* Parallels Desktop 12.1.3 虚拟机【自用】

* Navicat Premium 支持多种数据库

* dash 离线官方api文档【自用】

* paw 强大的REST API分析调试工具

* [Keka - macOS 压缩文件管理器](https://www.keka.io/zh-cn/) 【自用】 支持常见压缩/解压文件格式 zip 7z rar ...

* sublime text3 编辑器【自用】

* MWeb 编辑器 强大的markdown编辑器 尤其支持hexo／wordpress等博客

* Quiver 编辑器 程序员的笔记应用 很小4.4mb 功能强大

* CheatSheet 在任何软件界面 长按command弹出快捷键的说明

* Automator （系统自带）自动复制文件等自动化操作

* winery 运行exe程序

* eudic字典

* RescueTime

* XtraFinder (Finder的增强工具 )

* PDF Expert   最好的pdf阅读工具【自用】

* NearLock 这款Mac + iPhone 结合使用 利用蓝牙距离对 Mac 自动锁屏/开锁


### 常用快捷键

官网 - Mac 键盘快捷键 https://support.apple.com/zh-cn/HT201236

`option + command + L` 打开下载文件夹

`command+alt+D` 显示／隐藏Dock

`Cmd +Opt + Esc` 活动监视器  强制结束正运行的程序


### Finder下使用的快捷键

`Command+Shift+G` 前往任何文件夹(包括隐藏文件夹)

`Command+Shift+.` 显示/隐藏 文件、文件夹

`Command+Shift+G` 跳转到输入的文件夹

### 截图快捷键

+表示同时操作   -表示分步操作   在快捷键基础上同时按住Control 键可将截图复制到剪切板。

```
Cmd+Shift+3
全屏截图保存至桌面。

Cmd+Shift+4
区域截图。鼠标光标变成带坐标的小十字，通过拖拽截取特定区域，保存至桌面。

Cmd+Shift+4 - 按一下空格键 - 鼠标单击指定窗口
窗口截图。出现小十字光标后对指定窗口按空格键，鼠标光标变成照相机，鼠标单击截取指定窗口，保存截图至桌面。

Cmd+Shift+4 - 按住空格键拖动鼠标
区域截图。选取区域范围后，按住空格键并拖到鼠标可移动选取范围，释放按键后保存截图至桌面文件夹。
Cmd+Shift+4 - 按住Shift - 上下/左右移动鼠标
区域截图。选取区域范围后，按住 Shift 并上下/左右移动鼠标即可固定选取范围宽度/高度改变高度/宽度，释放按键后保存截图至桌面文件夹。
Cmd+Shift+4 - 按住Shift和Option
区域截图。选取区域范围后，按住 Shift 和 Option 键在上一快捷键功能的基础上自由切换高度宽度，释放按键后保存截图至桌面文件夹。
Cmd+Shift+4 - 按住Option
区域截图。选取区域范围后，按住 Option 键可按比例缩放选取范围，释放按键后保存截图至桌面文件夹。
```

### macOS所有命令
http://ss64.com/osx/

```
last | grep shutdown #查看关机时间记录
last | grep reboot #查看开机时间记录
```

注意 unix系统 文件名区别大小写.

根目录标志 `/` 有意义:
`cd /System` 表示转到根目录下的System中
`cd System`表示转到当前目录下的 System中

`／`
根目录

`ls /Volumes`
列出所有驱动器

`/Systme/Library/Extensions`
驱动所在位置

`/User/用户名`
用户文件夹位置

`/User/用户名/Desktop`
桌面的位置

`sudo －s`
取得root权限

 `command + k` 或 `clear`或 `Ctrl + l`
（终端）清屏

### bash - 编辑快捷键

`Ctrl + a`移到命令行首

`Ctrl + e`移到命令行尾

`Ctrl + f`按字符前移（右向）

`Ctrl + b`按字符后移（左向）

`Alt + f`按单词前移（右向）

`Alt + b`按单词后移（左向）

`Ctrl + x`在命令行首和光标之间移动

`Ctrl + u`从光标处删除至命令行首

`Ctrl + k`从光标处删除至命令行尾

`Ctrl + w`从光标处删除至字首

`Alt + d`从光标处删除至字尾

`Ctrl + d`删除光标处的字符

`Ctrl + h`删除光标前的字符

`Ctrl + y`粘贴至光标后

`Alt + c`从光标处更改为首字母大写的单词

`Alt + u`从光标处更改为全部大写的单词

`Alt + l`从光标处更改为全部小写的单词

`Ctrl + t`交换光标处和之前的字符

`Alt + t`交换光标处和之前的单词
`Alt + Backspace` 和Ctrl + w ~~相同~~类似，分隔符有些差别

重新执行命令

`Ctrl + r`逆向搜索命令历史
`Ctrl + g`从历史搜索模式退出
`Ctrl + p`历史中的上一条命令
`Ctrl + n`历史中的下一条命令
`Alt + .`使用上一条命令的最后一个参数

### bash - 控制命令

`Ctrl + o`执行当前命令，并选择上一条命令
`Ctrl + s`阻止屏幕输出
`Ctrl + q`允许屏幕输出
`Ctrl + c`终止命令
`Ctrl + z`挂起命令

### bash - Bang (!) 命令
`!!`执行上一条命令

`!blah`执行最近的以 blah 开头的命令，如 !ls

`!blah:p`仅打印输出，而不执行

`!$`上一条命令的最后一个参数，与 Alt + . 相同

`!$:p`打印输出 !$ 的内容

`!*`上一条命令的所有参数

`!*:p`打印输出 !* 的内容

`^blah`删除上一条命令中的 blah

`^blah^foo`将上一条命令中的 blah 替换为 foo

`^blah^foo^`将上一条命令中所有的 blah 都替换为 foo


### 程序员相关

必备软件包管理工具Homebrew
* 可安装程序多 并且一直更新  官网 https://brew.sh/
* 干净  通过Homebrew安装的软件全都在目录`/usr/local/Cellar` 保持系统内置的依赖库和自带软件不变
* 彻底删除Homebrew 应当用卸载脚本 https://gist.github.com/mxcl/1173223而不要直接删除 /usr/local/

```
#终端安装命令
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

权限问题请参考https://stackoverflow.com/questions/16432071/how-to-fix-homebrew-permissions

cd /usr/local/Cellar #所有的软件包 都统一在该目录下

brew update #更新Homebrew自身

brew outdated #查看已经过时的软件包名(有更新的版本)

brew upgrade  #直接升级所有过期软件包

brew upgrade sqlmap #升级指定的软件包（brew 升级某软件包的版本后，并不会删除旧版本软件包）

brew cleanup -n # 查看哪些软件包可被清除

brew cleanup #清除所有软件包的所有老版本

brew cleanup sqlmap #删除指定软件包的所有老版本

brew uninstall formula_name --force  #彻底卸载某个软件包（本机不再需要该软件）
```

* 安装软件命令
如安装you-get  （可下载世界各大网站视频／音乐  并保存到当前目录`pwd` 
  https://github.com/soimort/you-get ）
`brew install you-get`

* sqlite数据库GUI管理工具
`brew cask install db-browser-for-sqlite`

* shadowsocks-ng 科学上网
地址https://github.com/shadowsocks/ShadowsocksX-NG

* proxychains 终端下的代理链工具（关闭mac安全机制SIP才能使用！）

```bash
$ proxychains4 help
[proxychains] config file found: /usr/local/etc/proxychains.conf
[proxychains] preloading /usr/local/Cellar/proxychains-ng/4.12_1/lib/libproxychains4.dylib
proxychains can't load process....: No such file or directory
$ vim /usr/local/etc/proxychains.conf
```

* 配置git的ssh（macOS自带git）

* proxifier 全局 终端代理

* 改变网络mac地址(mac地址是全球唯一的，不过是被操作系统识别并提供其他程序调用，故可被修改)
 https://github.com/feross/SpoofMAC

* iterm2 取代原生终端   官方文档http://www.iterm2.com/documentation.html
 * 修改配色
Perferences - Profiles 选项卡 - Colors
最右下角有几种内置的配色方案：黑色风格，白色风格，深蓝风格...
 * 双击复制文本
比如输入ping wiki.com  双击ip即可复制
 * cmd + f
输入一个开头，tab可自动“智能”补全
点击放大镜 可支持 正则表达式
 * 一键隐藏／显示
Perferences - Keys 选项卡
左下角有 `show/hide iTerm2 with a system-wide hotkey`
勾选 推荐为`cmd\\`
 * 标记/跳转
标记 ：在某一行按下 Cmd + Shift + M
跳回行：刚才设置的那一行 Cmd + Shift + J
 * 剪贴板历史纪录
Cmd + Shift + H
显示（使用命令行过程中）所有的复制粘贴的内容列表
 *  曾经执行的命令
Cmd + Option + B
界面上显示一个时间轴，按下键盘的左右箭头控制时间轴上的点，查看当时的命令
 * 多窗口搜索
Cmd + Option + E
 搜索，最匹配关键词的窗口会高亮显示，按回车键切换到高亮窗口。





Command+L 浏览器下,定位到地址栏
Command +/-  放大或缩小字体



Command+~ 同一应用程序多窗口间切换


Command+Tab 任意情况下切换应用程序-向前循环
Shift+Command+Tab 切换应用程序-向后循环

Command+Delete 把选中的资源移到废纸篓
Shift+Command+Delete 清倒废纸篓（有确认）
Shift+Option+Command+Delete 直接清倒废纸篓


fn+delete可以往前删

fn+上下左右方向键可以实现PageUP/PageDown/Home/End的功能



### zip命令

* zip命令
  * -e 指定解密zip包的口令
  * -r 对文件夹进行压缩
* zip实操
  * 文件  -压缩->  普通zip包 `zip backup.zip 1.png`
  * 文件  -压缩->  加密zip包 `zip -e backup.zip 1.png`
  * 文件夹 -压缩->  普通zip包 `ip -r aaa.zip notes`

### open命令

直接打开文件夹
open /Users

直接打开文件
open a.txt

直接打开application
open -a Safari

用 文本编辑器打开本目录下的文件
open -a TextEdit a.txt

打开网站
open http://news.sina.com.cn


### time命令

如果你想知道在终端执行的某个程序耗时多久，对CPU等的使用情况，可以输入：
time python fib.py

输出结果：
程序结果..
real 0m0.005s
user 0m0.001s
sys 0m0.003s



### 聚焦搜索Spotlight 高级


输入节日：
春节

通过文件类型搜索：

kind:文件类型搜索关键字

kind:bookmark
搜索书签和历史纪录

kind:document
搜索各类文档

kind:folder
搜索文件夹

kind:pic
搜索图片

kind:movies
搜索视频

kind:music
搜索音乐

kind:pdf
搜索pdf文件


kind:word
搜索word



kind:contact
搜索联系人

kind:app
搜索应用程序


kind:email
搜索email



label:红
就可以找到红色标签的文件和文件夹。



* 通过日期搜索

date:today
查看今天创建或修改的文件

date:yesterday
查看昨天创建或修改的文件


date:2018-05-0
查看2018年5月1日创建或修改的文件



* 条件表达式
想搜索包含Mac不包含Windows的Keynote，可以这样写：

kind:app -Telgram

也可以这样写：

kind:app NOT Telgram



可以使用+/- 条件表达式
也可以通过NOT AND OR 【一定要大写，否则会被当做搜索内容处理】





### 输入符号

有的字符仅在英文输入法下可行

```
美元，shift+4
$

美分，option+4
¢


英镑，option+3
£


人民币，option+y
¥

欧元，option+shift+2
€


省略号，option+;
……


约等于，option+x
≈

度，option+shift+8
°


除号，option+/
÷


无穷大，option+5
∞


小于等于，option+,
≤


大于等于，option+.
≥


不等于，option+=
≠


圆周率Pi，option+p
π


正负，option+shift+=
±


平方根，option+v
√


总和，option+w
∑


注册，option+r
®


版权，option+g
©

商标Trademark  option+2
™
```

# MacOS有用的技巧

### 查看iphone连接的wifi密码

前提：苹果设备都开启了icloud 钥匙串访问

先用万能钥匙类app 或 一键连微信小程序 连接到wifi，然后用苹果电脑查看keychain中的wifi密码即可


### MacBook提供热点

如果你的macbook通过网卡接网线 才可提供WIFI热点：

（如果你的macbook通过wifi来上网 因为被占用 所以不能提供WIFI热点）

搜索：互联网共享

在设置窗口
左侧 选中“互联网共享”
右侧 共享以下来源的连接  选择【WIFI】

点击下方的 wifi选项 可设置 wifi密码。


### Mac电脑 （插网线）开启热点

如果你的mac是通过蓝牙连接上网的 可做热点：


搜索：互联网共享

在设置窗口
左侧 选中“互联网共享”
右侧 共享以下来源的连接  选择【蓝牙PAN】

### Mac版 微信小助手

[TKkk-iOSer/WeChatPlugin-MacOS: 一款功能强大的 macOS 版微信小助手](https://github.com/TKkk-iOSer/WeChatPlugin-MacOS)
```
git clone https://github.com/TKkk-iOSer/WeChatPlugin-MacOS.git
./Other/Install.sh

./Other/Uninstall.sh
```
