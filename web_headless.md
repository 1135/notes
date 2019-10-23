### Headless Chrome

Mac系统安装Chrome即可使用headless模式.

* 使用alias设置临时别名
  * `alias google-chrome="/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome"`
* 使用Headless模式发起HTTP请求 并打印页面DOM
  * `google-chrome --headless --dump-dom http://xx.net`
* 设置User-Agent 参考[User Agents](https://developers.whatismybrowser.com/)
  * `--user-agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.113 Safari/537.36"`


Headless模式 默认的HTTP请求(可以看到User-Agent标明了Headless模式 浏览器版本 ...)
```
GET / HTTP/1.1
Host: xx.net
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_4) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/73.0.3683.86 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
```


### Puppeteer

Puppeteer is a Node library which provides a high-level API to control headless Chrome or Chromium over the DevTools Protocol.

Puppeteer是一个提供高级API[(查看官方API文档)](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md)的node库，能够通过DevTools协议控制headless Chrome/Chromium

Puppeteer由Chrome DevTools team维护

* Puppeteer能在headless模式下模拟大部分的人为操作 应用非常广泛
  * Web自动化操作  包括 登录 注销...
  * Web自动化测试  包括 QA测试 安全测试...
  * 爬虫 模拟正常浏览器访问 可以解析javascript
  
在线体验[Try Puppeteer](https://try-puppeteer.appspot.com/)
