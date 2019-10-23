### Node.js

Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine.

[V8 JavaScript engine](https://v8.dev/): Google的开源高性能JavaScript和WebAssembly引擎

[Node.js 官方API文档](https://nodejs.org/api/)

### 包管理工具 - npm

npm 是 node 的包管理器
Mac下通过Homebrew安装npm
```
brew install npm
```

npm 包安装 按文件保存位置分类
* local - 将安装包放在当前目录下的node_modules目录 (无该目录则创建)
  * 本地 安装模块 `npm install <Module Name>`
  * 本地 删除模块 `npm uninstall <Module Name>`
  * 代码中通过`require()`引入本地安装的包
* global - 将安装包放在 /usr/local 下或者你 node 的安装目录
  * 全局 安装模块 `npm install -g <Module Name>`
  * 全局 删除模块 `npm uninstall -g <Module Name>`
  * 查看全局模块 `npm list -g`
  * 命令行程序 可在命令行里直接运行
  * 运行代码找不到模块 则需指明全局模块路径 Mac下为`export NODE_PATH=/usr/local/lib/node_modules`

例1 本地安装
```
#local install - 使用npm本地安装express包 保存到"当前目录"下的 node_modules 目录中
npm install express

#local - 使用 express包 只需要直接引用 而无需指定包的路径
var express = require('express');
```


例2 指定npm仓库为 [国内镜像](https://npm.taobao.org/)

```
指定国内镜像仓库
npm install -g cnpm --registry=https://registry.npm.taobao.org

#以后都可使用cnpm代替npm  如下载安装以下常用包 (i 是 install 的缩写)
cnpm install -g vue-cli
cnpm i -g vue
cnpm i -g webpack
cnpm i -g webpack-cli
```

### node软件

例1 fkill-cli （一个不错的结束进程的命令行程序)
```
#安装
npm install --global fkill-cli

#运行
fkill
```


例2 live-server(实现文件下载服务)
```
#安装
npm i live-server -g

#运行
live-server
```


### 包管理工具 - Yarn

更强大的包管理器包管理器安装[Yarn](https://yarnpkg.com/)

Mac下通过Homebrew安装yarn
```
brew install yarn
```

如果你用了node多版本管理软件，如nvm(Node Version Manager)，则应排除安装Node.js(以便使用 nvm 的 Node.js 版本)
```
brew install yarn --without-node
```

项目-初始化:
```
yarn init
#生成package.json文件 存储了项目的有关信息:项目名称、维护者信息、代码托管地址、"项目依赖"
```

项目-添加依赖包:
```
yarn add [pkg-name] #会自动安装最新版本
yarn add jquery@2.1.4 #指定版本
```

项目-安装依赖包:
Yarn会从package.json中读取依赖并将依赖信息存储到yarn.lock中
```
yarn install
yarn install --force #重新下载
```

项目-更新依赖包:
会更新package.json和yarn.lock文件
```
yarn upgrade [package]
yarn upgrade –latest [package] #更新到最新版本
yarn upgrade [package]@[version] #指定版本
yarn upgrade [package]@[tag]
```

项目-删除依赖包:
会更新package.json和yarn.lock 文件
```
yarn remove [package]
yarn remove [pkg-name1] [pkg-name2]
```

yarn.lock 作用:记录了统一依赖信息(版本),不同环境只需运行 yarn install可安装对应依赖包。
