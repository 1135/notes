### 移动应用安全

移动安全测试指南([OWASP-MSTG](https://github.com/OWASP/owasp-mstg))是"移动APP安全测试"和"逆向工程"的综合手册

### android-应用常见漏洞

[android漏洞列表 安全知识库](http://appscan.360.cn/vulner/list/)

* 加密 - AES/DES硬编码密钥
  * 风险描述: 使用AES或DES加解密时，采用硬编码在程序中的密钥
  * 危害描述: 通过反编译拿到密钥可以轻易解密APP"通信数据"
  * 修复建议: 密钥加密存储 或 用过变形后进行加解密运算
* 加密 - AES弱加密 [参考](https://developer.android.com/reference/javax/crypto/Cipher.html)
  * 风险描述: 在AES加密时，使用了安全性不高的`AES/ECB/NoPadding`或`AES/ECB/PKCS5padding`的模式
  * 危害描述: ECB是将文件分块后对文件块做同一加密，所以只需要针对一个文件块进行解密即可破解加密，降低了破解难度和文件安全性
  * 修复建议: 禁止使用AES加密的`ECB`模式，显式指定加密算法为CBC或CFB模式，可带上PKCS5Padding填充。AES密钥长度推荐使用256位(最少是128位)
* 网络 - SSL通信客户端检测信任任意证书 [参考](https://developer.android.com/reference/javax/net/ssl/X509TrustManager.html)
  * 风险描述: 自定义SSL x509 TrustManager，重写checkClientTrusted方法，方法内不做任何服务端的证书校验
  * 危害描述: 攻击者可以使用中间人攻击获取加密内容
  * 修复建议: 严格判断服务端和客户端证书校验，对于异常事件禁止return空或null
* 网络 - HTTPS关闭主机名验证 [参考](https://developer.android.com/reference/javax/net/ssl/HostnameVerifier.html)
  * 风险描述: 构造HttpClient时，设置HostnameVerifier时参数使用ALLOW_ALL_HOSTNAME_VERIFIER或空的HostnameVerifier
  * 危害描述: 关闭主机名校验可以导致黑客使用中间人攻击获取加密内容
  * 修复建议: setHostnameVerifier时使用STRICT_HOSTNAME_VERIFIER替代ALLOW_ALL_HOSTNAME_VERIFIER来进行证书严格校验，自定义实现HostnameVerifier时，在实现的verify方法中对Hostname进行严格校验
* 网络 - WebView忽略SSL证书错误 [参考](https://developer.android.com/reference/android/webkit/WebViewClient.html)
  * 风险描述: WebView调用onReceivedSslError方法时，直接执行handler.proceed()来忽略该证书错误
  * 危害描述: 忽略SSL证书错误可能引起中间人攻击
  * 修复建议: 不要重写onReceivedSslError方法， 或者对于SSL证书错误问题按照业务场景判断，避免造成数据明文传输情况
* 网络 - 开放socket端口
  * 风险描述: 开放网络端口带来的风险，APP绑定端口进行监听，建立连接后可接收外部发送的数据。
  * 危害描述: 攻击者可构造恶意数据对端口进行测试，对绑定了`0.0.0.0`的APP可发起远程攻击
  * 修复建议: 如无必要，只绑定本地IP`127.0.0.1` 并且对接收的数据进行过滤、验证
* 广播 - 动态注册广播
  * 风险描述: 使用`registerReceiver`动态注册的广播在组件的生命周期里是默认导出的
  * 危害描述: 导出的广播可以导致拒绝服务、数据泄漏或是"越权调用"
  * 修复建议: 使用带权限检验的`registerReceiver`API进行动态广播的注册
* 广播 - BroadcastReceiver组件暴露 [参考](https://developer.android.com/guide/topics/manifest/receiver-element.html)
  * 风险描述: BroadcastReceiver组件的属性exported被设置为true或是未设置exported值但IntentFilter不为空时，BroadcastReceiver被认为是导出的
  * 危害描述: 导出的广播可以导致数据泄漏或者是越权
  * 修复建议: 如果组件不需要与其他app共享数据或交互，请将AndroidManifest.xml配置文件中设置该组件为`exported = "False"` 如果组件需要与其他app共享数据或交互， 请对组件进行权限控制和参数校验
* 程序可被任意调试
  * 风险描述: 安卓AndroidManifest.xml文件中`android:debuggable`为`true`
  * 危害描述: app可以被任意调试。
  * 修复建议: AndroidManifest.xml文件中设置为`android:Debugable="false"`
* 程序数据任意备份
  * 风险描述: 安卓AndroidManifest.xml文件中`android: allowBackup`为`true`
  * 危害描述: app数据可以被备份导出
  * 修复建议: AndroidManifest.xml文件中设置为`android:allowBackup="false"`
* 随机数不安全使用
  * 风险描述: 调用SecureRandom类中的setSeed方法
  * 危害描述: 生成的随机数具有确定性，存在被破解的可能性
  * 修复建议: 使用`/dev/urandom`或者`/dev/random`来初始化伪随机数生成器
* 动态链接库中包含执行命令函数
  * 风险描述: 在native程序中，有时需要执行系统命令，在接收外部传入的参数执行命令时没有严格过滤/检验
  * 危害描述: 攻击者传入任意命令，导致恶意命令的执行
  * 修复建议: 对传入的参数进行严格的过滤 (建议从设计上彻底禁止使用系统命令执行函数 改为其他成熟的实现)
* 未使用地址空间随机化技术
  * 风险描述: PIE(Position Independent Executables)是一种地址空间随机化技术，当so被加载时，在内存里的地址是随机分配的
  * 危害描述: 不使用PIE，将会使得shellcode的执行难度降低，攻击成功率增加
  * 修复建议: NDK编译so时 加入`LOCAL_CFLAGS := -fpie -pie` 开启对PIE的支持
* unzip解压缩 （ZipperDown)
  * 风险描述: 解压 zip文件，使用`getName()`获取压缩文件名后未对名称进行校验。
  * 危害描述: 攻击者可构造恶意zip文件，被解压的文件将会进行目录跳转被解压到其他目录，覆盖相应文件导致任意代码执行
  * 修复建议: 解压文件时，判断文件名是否有`../`特殊字符
* 未使用[编译器堆栈保护技术 Stack_canaries](https://en.wikipedia.org/wiki/Stack_buffer_overflow#Stack_canaries)
  * 风险描述: 为了检测栈中的溢出，引入了Stack Canaries缓解技术。`原理：在所有函数调用发生时，向栈帧内压入一个额外的被称作canary的随机数，当栈中发生溢出时，canary将被首先覆盖，之后才是EBP和返回地址。在函数返回之前，系统将执行一个额外的安全验证操作，将栈帧中原先存放的canary和.data中副本的值进行比较，如果两者不吻合，说明发生了栈溢出。`
  * 危害描述: 不使用Stack Canaries栈保护技术，发生栈溢出时系统并不会对程序进行保护。
  * 修复建议: 使用NDK编译so时，在Android.mk文件中添加`LOCAL_CFLAGS := -Wall -O2 -U_FORTIFY_SOURCE -fstack-protector-all`
* Fragment注入 [参考](https://developer.android.com/reference/android/preference/PreferenceActivity.html#isValidFragment(java.lang.String))
  * 风险描述: 通过导出的PreferenceActivity的子类，没有正确处理Intent的extra值
  * 危害描述: 攻击者可绕过限制访问未授权的界面
  * 修复建议: 当targetSdk >= 19时，强制实现了isValidFragment方法；当targetSdk<19时，在PreferenceActivity的子类中都要加入isValidFragment,两种情况下在isValidFragment方法中进行fragment名的合法性校验
* webview启用访问文件数据 [参考](https://developer.android.com/reference/android/webkit/WebSettings.html#setAllowFileAccess(boolean))
  * 风险描述: Webview中使用`setAllowFileAccess(true)` App可通过webview访问私有目录下的文件数据
  * 危害描述: 在Android中`mWebView.setAllowFileAccess(true)`为默认设置。此时在File域下，可执行任意的JavaScript代码（如果绕过同源策略能够对私有目录文件进行访问，导致用户隐私泄漏）
  * 修复建议: 使用WebView.getSettings().setAllowFileAccess(false)来禁止访问私有文件数据
* activity绑定browserable与自定义协议 [参考](https://developer.android.com/reference/android/content/Intent.html#CATEGORY_BROWSABLE)
  * 风险描述: activity设置`android.intent.category.BROWSABLE`属性并同时设置了自定义的协议`android:scheme` 即意味着可以通过浏览器使用自定义协议打开此activity
  * 危害描述: 可能通过浏览器对APP进行越权调用
  * 修复建议: APP对外部调用过程和传输数据进行安全检查或检验
* Provider文件目录遍历
  * 风险描述: 当Provider被导出且覆写了openFile方法时，没有对Content Query Uri进行有效判断或过滤
  * 危害描述: 攻击者可以利用`openFile()`接口进行文件目录遍历以达到访问任意可读文件的目的
  * 修复建议: 一般情况下无需覆写openFile方法，如果必要，对提交的参数进行`../`目录跳转符或其他安全校验
* DEX文件动态加载 [参考](https://developer.android.com/reference/dalvik/system/DexClassLoader.html)
  * 风险描述: 使用DexClassLoader加载外部的文件如apk/jar/dex 当外部文件的来源无法控制时或是被篡改，此时无法保证加载的文件是否安全
  * 危害描述: 加载恶意的dex文件将会导致任意命令的执行
  * 修复建议: 加载外部文件前，必须使用校验签名或MD5等方式确认外部文件的安全性
* Intent Scheme URLs攻击 [参考](https://developer.android.com/guide/components/intents-filters.html)
  * 风险描述: 在AndroidManifast.xml设置Scheme协议之后，可以通过浏览器打开对应的Activity
  * 危害描述: 攻击者通过访问浏览器构造Intent语法唤起app相应组件，轻则引起拒绝服务，重则可能演变为提权漏洞
  * 修复建议: 配置category filter, 添加android.intent.category.BROWSABLE方式规避风险
* 隐式意图调用 [参考](https://developer.android.com/guide/components/intents-filters.html)
  * 风险描述: 封装Intent时采用隐式设置，只设定action，未限定具体的接收对象，导致Intent可被其他应用获取并读取其中数据
  * 危害描述: Intent隐式调用发送的意图可能被第三方劫持，可能导致内部隐私数据泄露
  * 修复建议: 将隐式调用改为显式调用
* ContentProvider组件暴露 [参考](https://developer.android.com/guide/topics/providers/content-providers.html)
  * 风险描述: Content Provider组件的属性exported被设置为true或是Android API<=16时，Content Provider被认为是导出的
  * 危害描述: 黑客可能访问到应用本身不想共享的数据或文件
  * 修复建议: 如果组件不需要与其他app共享数据或交互，请将AndroidManifest.xml 配置文件中设置该组件为exported = “False”。如果组件需要与其他app共享数据或交互，请对组件进行权限控制和参数校验
* Webview存在本地Java接口 [参考](https://developer.android.com/reference/android/webkit/WebView.html)
  * 风险描述: android的webView组件有一个非常特殊的接口函数addJavascriptInterface，能实现本地java与js之间交互
  * 危害描述: 在targetSdkVersion小于17时，攻击者利用 addJavascriptInterface这个接口添加的函数，可以远程执行任意代码
  * 修复建议: 建议开发者不要使用addJavascriptInterface，使用注入javascript和第三方协议的替代方案
* Activity组件暴露 (参考)[https://developer.android.com/guide/components/activities.html]
  * 风险描述: Activity组件的属性exported被设置为true或是未设置exported值但IntentFilter不为空时，activity被认为是导出的，可通过设置相应的Intent唤起activity
  * 危害描述: 黑客可能构造恶意数据针对导出activity组件实施越权攻击
  * 修复建议: 如果组件不需要与其他app共享数据或交互，请将AndroidManifest.xml 配置文件中设置该组件为exported = “False”。如果组件需要与其他app共享数据或交互， 请对组件进行权限控制和参数校验
* Service组件暴露 [参考](https://developer.android.com/guide/components/services.html)
  * 风险描述: Service组件的属性exported被设置为true或是未设置exported值但IntentFilter不为空时，Service被认为是导出的，可通过设置相应的Intent唤起Service
  * 危害描述: 攻击者可能构造恶意数据针对导出Service组件实施越权攻击
  * 修复建议: 如果组件不需要与其他app共享数据或交互，请将AndroidManifest.xml配置文件中设置该组件为`exported = "False"` 如果组件需要与其他app共享数据或交互，请对组件进行权限控制和参数校验
* ContentProvider组件暴露 [参考](https://developer.android.com/guide/topics/providers/content-providers.html)
  * 风险描述: Content Provider组件的属性exported被设置为true或是Android API<=16时，Content Provider被认为是导出的
  * 危害描述: 攻击者可能访问到应用本身不想共享的数据或文件
  * 修复建议: 如果组件不需要与其他app共享数据或交互，请将AndroidManifest.xml 配置文件中设置该组件为exported = “False”。如果组件需要与其他app共享数据或交互， 请对组件进行权限控制和参数校验
* 全局文件可读可写 [参考](https://developer.android.com/reference/android/content/Context.html#MODE_WORLD_WRITEABLE)
  * 风险描述: APP在创建内部存储文件时，将文件设置了全局的可读可写权限
  * 危害描述: 攻击者恶意写文件内容或者，破坏APP的完整性，或者是攻击者恶意读取文件内容，获取敏感信息
  * 修复建议: 请开发者确认该文件是否存储敏感数据，如存在相关数据，请去掉文件全局可写、写属性
* 1
  * 风险描述: 
  * 危害描述: 
  * 修复建议: 

#### 有影响力的漏洞

* [ZipperDown漏洞](https://zipperdown.org/)
  * 风险描述: Zip文件解压缩过程中，由文件名和符号链接等引起的"路径穿越"进而导致"文件覆盖"是一类非常经典的安全问题
  * 危害描述
    * iOS - 常见于"热补丁"升级功能处 `SSZipArchive和ZipArchive解压缩过程中没有考虑到文件名中包含 ../ 的情况，造成了文件释放过程中路径穿越，导致恶意Zip文件可以在App沙盒范围内，覆盖任意可写文件`
    * Android - 同样发现了类似漏洞
  * 修复建议: 严格使用https下载Zip文件，且对下载的Zip文件进行安全校验以防止篡改
* SQLite漏洞 - [Magellan](https://blade.tencent.com/magellan/)
  * 风险描述: Magellan（麦哲伦）是Tencent Blade Team发现的一个存在于SQLite中的远程执行代码漏洞。作为知名开源数据库SQLite已经被应用于所有主流操作系统和软件中，因此该漏洞影响面十分广泛。经测试Chromium浏览器也受此漏洞影响，目前Google与SQLite官方已确认并修复了此漏洞。本着负责任的漏洞披露原则 ，我们暂时不会透露此漏洞的任何详细信息，同时我们正在努力推动所有受影响厂商尽快修复此漏洞。
  * 危害描述: SQLite高危漏洞，该漏洞可远程利用达到任意代码执行
  * 修复建议: 升级SQLite至最新版本

#### android-反编译工具

|名称|属性|描述|
|:-------------:|--|-----|
|[jadx](https://github.com/skylot/jadx)|Java|17k★ Dex to Java decompiler [参考文章posts.specterops.io](https://posts.specterops.io/dont-you-forget-about-re-e2c92d67c641)|
|[Luyten](https://github.com/deathmarine/Luyten)|Java|2k★ Java Decompiler [Gui] |


#### android-应用自动化检测平台

>参考文章[移动APP漏洞自动化检测平台建设](https://security.tencent.com/index.php/blog/msg/109)

|名称|属性|描述|
|:-------------:|--|-----|
|[360显危镜](http://appscan.360.cn/)|web|自动扫描多种漏洞类型 完全免费|
