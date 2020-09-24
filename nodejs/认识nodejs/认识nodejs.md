## Node.js是什么

- Node.js 是JavsScript运行时环境(类似java的JVM)
- 简单来说就是可以解析和执行JS代码，而以前是只有浏览器可以解析

### 浏览器中的JavaScript

- EcmaScript
  - 基本的语法
  - if
  - var
  - function
  - Object
  - Array

- BOM
- DOM


### Node.js中的JavaScript
- 没有BOM/DOM

- EcmaScript

- 在Node这个JavaScript执行环境中，为JS提供了一些服务器级别的API
  - 文件读写
  - 网络服务的构建
  - 网络通信
  - http服务器

- 构建于Chrome的V8引擎之上

  - 代码只是具有特定歌是的字符串而已

  - 引擎可以帮你去解析和执行

  - Google Chrome的V8引擎是目前公认的解析执行JS代码最快的

  - Node.js的作者把Google Chrome上的V8引擎移植了出来，开发了一个独立的JS运行时环境

    

## Node.js特性

- event-driven 事件驱动
- non-blocking I/O 非阻塞IO模型(异步）
- lightweight and efficient 轻量和高效

## npm：node package manager

- npm是世界上最大的开源库生态系统

- npm是node开发的

- 绝大多数js相关的包，都存放在npm上面

- `npm install jquery`

- `npm i jquery@1.1.1`

- `npmjs.com` 可发布自己的包，也可查找包

- npm升级 `npm install --global npm`

- npm 常用命令

  - npm init
    - 初始化package.json
    - 跳过向导，全选默认值： npm init -y
  - npm install
  - npm install 包
  - npm install --save 包
    - 简写 `npm -S 包`
  - npm uninstall 包
    - 只删除，如果有依赖会保留
    - 简写: npm un 包
  - npm uninstall --save 包
    - 删除的同时会包依赖一起删除
    - 简写： npm un -S 包
  - npm help
  - npm  命令 help
    - 命令解释

- 淘宝镜像

  - 安装淘宝 cnpm

  ```js
  npm install --global cnpm
  ```

  - 镜像

  ```js
  npm config set registry https://registry.npm.taobao.org
  ```

  - 查看配置

   ```js
  npm config list
   ```

  



## 模块化

- 使用require来加载模块

- 使用exports接口对象来导出模块中的成员（导出的是整个对象）

- 想导出模块中某个对象，用 `module.exports`

- 原理：exports 是module.exports 的一个引用

  ```js
  实际上所有的node模块，都有这么一些默认代码(隐藏)、
  module.exports = {};
  exports = module.exports;
  return module.exports;
  ```

  

## require加载原则

- 主要是第三方模块
  - 第三方模块必须通过npm来下载
  - 必须通过require('包名')来加载
  - 不可能和核心模块重名
  - 导包是无需路径是怎么找到唯一的入口的呢
    - 先找当前文件所处目录中的node_modules目录
    - node_modules/art-template
    - node_modules/art-template/package.json
    - node_modules/art-template/package.json中的main属性记录了art-template模块的入口模块(index.js)
    - **如果package.json不存在或者main没有，则node会找index.js（方式2）**
    - 以上都不符合，则继续查找，一层一层往上查找，直至找到磁盘根路径(F:/)，再次不存在就报错
- 模块查找机制(优先级)
  - 优先找缓存
  - 核心模块
  - 路径形式的文件模块
  - 第三方模块 （参考上方）
  - 

## package.json 项目依赖，npm i --save xxx

- 初始化package.json

  ```js
  npm init
  ```

  ```js
  {
    "name": "day02",
    "version": "0.0.1",
    "description": "测试package.json",
    "main": "main.js",
    "dependencies": {
      "art-template": "^4.13.2",
      "jquery": "^3.5.1"
    },
    "devDependencies": {},
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "codeman",
    "license": "ISC"
  }
  
  ```

- 有了package.json后，可以直接通过命令 `npm install` 下载package.json中的依赖(类似Maven)

## package-lock.json

- 可以锁定版本(避免依赖升级)
- 可以记录下载地址，下次下载会更快

## 热重启nodemon

- 安装

  ```shell
  npm i --global nodemon
  ```

- 使用

```shell
nodemon 替换node 
```

