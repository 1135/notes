### 处置流程

* 信息发现
* 信息删除(尽量删除)
* 消除影响(尽量更改)
  * 泄露的配置 - 域名/ip...
  * 泄露的凭证 - username/password/token...

### 企业信息泄露的源

* 外部-泄露源
  * 代码仓库 - 如github
  * 网盘 - 如百度网盘
  * 云笔记 - 如Pastebin
* 内部-查看权限不严
  * 内部wiki系统
  * 内部IM系统
  * ...

### 自动化-github信息泄露发现系统

|名称|属性|运行环境|描述|
|:-------------:|--|--|-----|
|[MiSecurity/x-patrol](https://github.com/MiSecurity/x-patrol/)|Golang|all|MiSecurity|
|[michenriksen/gitrob](https://github.com/michenriksen/gitrob)|Golang|allReconnaissance tool for GitHub organizations|

**x-patrol 搭建过程**

进入 https://github.com/MiSecurity/x-patrol/releases

下载 二进制版本

MAC的二进制文件名为x-patrol_darwin_amd64

还需要下载仓库中的3个文件夹：conf、public、templates

现在文件夹结构如下：
```
.
├── conf
│   ├── app.ini
│   └── gitrob.json
├── public
│   ├── css
│   ├── js
│   └── lib
├── templates
└── x-patrol_darwin_amd64
```

配置数据库命令：
```
mysqld #启动mysql服务端
create database xiaomi; #登录mysql并创建一个让该系统使用的数据库 我这里命名为xiaomi
```

修改conf/app.ini文件：
```
HTTP_HOST = 127.0.0.1
HTTP_PORT = 8000
MAX_INDEXERS = 2
DEBUG_MODE = true
REPO_PATH = repos
MAX_Concurrency_REPOS = 1

[database]
;support sqlite3, mysql, postgres
DB_TYPE = mysql
HOST = 127.0.0.1
PORT = 3306
NAME = xiaomi
USER = root
PASSWD = root
SSL_MODE = disable
PATH = data
```

运行二进制文件：
```
./x-patrol_darwin_amd64 web
```

登录系统：
```
http://127.0.0.1:8000

#默认账号密码 建议登录后修改
xsec
x@xsec.io
```

![效果图.png (2862×1234)](https://camo.githubusercontent.com/6f66528be6b1a80300ba103995f71a69fb629335/68747470733a2f2f696d61676573322e696d67626f782e636f6d2f39372f65342f5676776f364450555f6f2e706e67)


* web安全测试结果
  * 有CSRF漏洞 实际意义不大
  * 无sql注入漏洞 所有sql查询均使用了orm框架 [go-xorm/xorm](https://github.com/go-xorm/xorm) 支持 mysql,postgres,tidb,sqlite3,mssql,oracle
  * 无XSS漏洞
