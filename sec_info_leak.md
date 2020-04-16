### 处置流程

* 信息发现
* 信息删除
  * 某个git仓库包含了敏感信息 - 可以删除敏感信息后commit 再删除相关的commit记录 这样可以保持代码不变的情况下删除敏感信息. 也可以删除git仓库的所有commit历史记录[git - how to delete all commit history in github? - Stack Overflow](https://stackoverflow.com/questions/13716658/how-to-delete-all-commit-history-in-github)
* 信息变更 尽量消除已泄露数据的影响
  * 泄露的配置 - 域名/ip...
  * 泄露的凭证 - username/password/token...

### 企业信息泄露的源

* 外部-泄露源
  * 代码仓库 - 如github
  * 网盘 - 如百度网盘
  * 云笔记 - 如Pastebin
  * ...
* 内部-查看权限不严
  * 内部wiki系统
  * ...

#### 自动化-github信息泄露发现系统

|名称|属性|运行环境|描述|
|:-------------:|--|--|-----|
|[MiSecurity/x-patrol](https://github.com/MiSecurity/x-patrol/)|Golang|all|MiSecurity|
|[michenriksen/gitrob](https://github.com/michenriksen/gitrob)|Golang|allReconnaissance tool for GitHub organizations|
