### 爬虫的分类


按类型
* 静态页面爬虫 (不能解析JavaScript) 如[1135/VulSpider](https://github.com/1135/VulSpider)
* 动态页面爬虫 (能够解析JavaScript) 如puppeteer模块调用headless chrome 如[1135/VulSpiderX](https://github.com/1135/VulSpiderX)

按架构
* 单任务版
* 并发版
* 分布式版


### 爬虫相关项目

|名称|属性|描述|
|:-------------:|--|-----|
|https://github.com/NikolaiT/GoogleScraper|python3.7 异步搜索引擎爬虫| 2k★ A Python module to scrape several search engines.|
|[chromeless](https://github.com/prisma/chromeless)|js 动态爬虫|Chrome自动化|
|[Anti-Anti-Spider](https://github.com/luyishisi/Anti-Anti-Spider)|python|反反爬虫|


### 静态爬虫 - 内容提取利器 - GraphQuery

项目地址https://github.com/storyicon/graphquery

[storyicon/graphquery-playground: The playground of graphquery, let you quickly start and test your GraphQuery statement.](https://github.com/storyicon/graphquery-playground)

参考文章https://juejin.im/entry/5bdab426f265da393d4af97e


GraphQuery 是一个文本查询语言 特点：

* 不依赖于任何后端语言，可以被任何后端语言调用
* 统一解析结果:某一段 GraphQuery 查询语句，在任何语言中有相同的解析结果
* 内置多种解析器: xpath选择器，css选择器，jsonpath 选择器 和 正则表达式 ，以及足量的文本处理函数，结构清晰易读，能够保证 数据结构、解析代码、返回结果 结构的一致性
* GraphQuery让开发者从这些重复繁琐的解析逻辑中解脱（不同的语言、不同的库的解析结果存在差异），写出高可读性、高可移植性、高可维护性的代码。

