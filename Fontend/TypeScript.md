# TypeScript

JavsScript是dynamic type，而TypeScript把它变成了static type，即需要声明类型了。

似乎导包需要@type/express和express的js包都得要。前者纯粹定义type，后者才是具体实现？

然后全都换成了import写法。

## 接口

interface LabelledValue {

label: string;

}

只需要一个obj含有label：{label:xxx}，就说这个对象实现了这个接口，可以传入。

## 问题

Element类型找不到，是需要在tsconfig.json中compilerOptions中的"lib": ["es6","dom"]添加dom。

<https://github.com/DefinitelyTyped/DefinitelyTyped/issues/52383>

try to import a CommonJS module into an ES6 module （如import path from ‘path’）需要在"compilerOptions"中添加"esModuleInterop": true,

<https://bobbyhadz.com/blog/typescript-module-can-only-be-default-imported-esmoduleinterop>

# node

除了eslint，node版本和支持的js语言版本也有关系。

nvm可以控制node的多个版本切换

<https://github.com/coreybutler/nvm-windows>

nvm current: Display active version.

nvm list [available]: List the node.js installations. Type available at the end to show a list of versions available for download.

nvm use \<version\> [arch]: Switch to use the specified version. Optionally use latest, lts, or newest.

nvm install \<version\> [arch]: The version can be a specific version, "latest" for the latest current version, or "lts" for the most recent LTS version. Optionally specify whether to install the 32 or 64 bit version (defaults to system arch). Set [arch] to "all" to install 32 AND 64 bit versions. Add --insecure to the end of this command to bypass SSL validation of the remote download server.

# npm

npm install [express@4.17.1](mailto:express@4.17.1)（指定版本） --save（可保存在package.json中？）

npm install -g @vue/cli

npm uninstall xxx

npm install npm -g npm升级到最新版本

npm -v 查看当前版本

\-g 全局安装包

\-S --save 将依赖包安装到生产dependencies，后续打包dist文件夹也会打包进去。

\-D --save-dev 将依赖包安装到devDependencies，该依赖只会在开发阶段用到，而在生产阶段不会用到。

都是在package.json可以读取到的消息

devDependencies 的理解：

我们在开发一个前端项目的时候，需要使用到webpack或者gulp来构建我们的开发和本地运行环境，这时我们就要安装到devDependencies 里。webpack或者gulp是用来打包压缩代码的工具，在项目实际运行的时候用不到，所以把webpack或者gulp放到devDependencies 中就行了。

dependencies 的理解：

我们在项目中用到了element-ui或者mint-ui，在生产环境中运行项目，当然也需要element-ui或者mint-ui，所以我们把element-ui或者mint-ui安装到dependencies中。

productionSourceMap: false, //打包不生成map

npm config set proxy=http://127.0.0.1:8087 设定代理

npm config set registry=http://registry.npmjs.org 设定仓库

npm config set https-proxy <http://server:port>设置https代理

npm config delete proxy

npm config delete https-proxy取消代理

# Express

## 使用http module显示html页面

非常的繁琐：

## 使用Express

<http://expressjs.com/>

### Route Params and Query

#### Post

post需要使用req.body

但var bodyParser = require('body-parser');

app.use(bodyParser());

需要使用该中间件才可以处理

app.use(express.json({ limit: "10000kb" }));其实Express也有相应中间件。limit可以设置post内容的大小。

### Middleware

# 问题

Cannot find module 'supports-color'

if diable breakpoints type "All Exceptions" then work fine. （即在Debug——Windows中ExceptionSettings还原默认异常设置）

