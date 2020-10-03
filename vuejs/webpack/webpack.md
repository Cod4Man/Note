# webpack

## 1.定义： 前端模块化工作，并且可以做打包 

## 2. webpack依赖node

## 3. 打包项目

### 3.1 手动打包

```shell
webpack ./src/main.js ./dist/bunld.js
```

### 3.2 自动打包(配置webpack.config.js)

```js
let path = require('path')

module.exports = {
    entry: './src/main.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    }
}
```

- 然后执行命令`webpack`即可
- 可以结合node，做出更配置化的使用

```js
package.json
{
  "name": "webpack",
  "version": "1.0.0",
  "description": "",
  "main": "webpack.config.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
-- shell
npm run build
```



## 4. loader [各种loader][https://webpack.js.org/loaders/]

- 是webpack中一个非常核心的概念。webpack主要是把js文件打包，而css,image,vue,ts等转换为浏览器适用的就要用到loader。webpack提供了可拓展的loader，用来处理这些事情
- 使用
  - ①： 通过npm安装对应的loader
  - ②： 在webpack.config.js中的modules键上配置

## 4.1 css-loader

- 安装css-loader

  ```shell
  npm install --save-dev css-loader
  ```

- webpack.config.js

  ```js
  let path = require('path')
  
  module.exports = {
      entry: './src/main.js',
      output: {
          path: path.resolve(__dirname, 'dist'),
          filename: 'bundle.js'
      },
      module:{
          rules:[
              {
                  test:/\.css$/,
                  // css-loader只负责css的加载
                  // style-loader负责解析，将样式加载到DOM中
                  // 注意先后顺序
                  use:['style-loader','css-loader']
              }
          ]
      }
  }
  ```

- 入口文件中引入

  ```js
  require('./css/common.css')
  ```

## 4.2 less-loader

```json
{
    test:/\.less$/,
    use:['style-loader','css-loader', 'less-loader']
}
```

## 4.3 url-loader 处理url，比如图片

![url-loader](E:\SoftwareNote\vuejs\img\url-loader.png)

```js
{
    test: /\.(img|png|gif|jpg)$/i,
    use: [
        // 图片大小小于limit(byte)时，则会将图片转换为base64
        // 图片大小大于limit时，则需要file-loader处理，会将图片拷贝到dist
        // dist/img底下，命名为原来名字.hash8位.原来后缀
        {loader: 'url-loader', options: {limit: 90284, name: 'img/[name].[hash:8].[ext]'}}
    ]
}
```

## 4.4 file-loader 处理文件(超出limit的图片)

## 4.5 babel-loader(ES6->ES5)

```
npm install -D babel-loader @babel/core @babel/preset-env
npm install -D babel-loader @babel/core @babel/preset-es2015
```

```js
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env']
        }
      }
    }
  ]
}
```

## 4.6 打包vue

```shell
npm install vue --save 
```

```js
webpack.config.js
==================
module.exports = {
    resolve: {
        // import可省略后缀
        extensions: ['.js', '.css', '.vue'],
        // vue有两种模式：
        // 1. runtime-only （不支持template，会报错）
        // 2. runtime-compiler (支持tempalte, 因此通过resolve定制)
        // 别名
        alias: {
            'vue$': 'vue/dist/vue.esm.js'
        }
    }
}
```

```shell
====webpack解析打包vue文件所需组件
npm install --save-dev vue-loader@13.0.0 vue-template-compiler@2.5.21
```

- 终极vue

```js
Cpn.vue
=======================
<template>
    <div>
        <h2>我是子组件cpn</h2>
        <h2>传参{{cpnmes}}</h2>
    </div>
</template>

<script>
    export default {
        name: "Cpn",
        props: {
            cpnmes: {
                type: String,
                default: 'cpnmes default value',
                required: true
            }
        }
    }
</script>

<style scoped>

</style>


App.vue
===================
<template>
    <div>
        <h2>{{mes}}</h2>
        <hr>
        <h2>我是app-temp组件</h2>
        <Cpn :cpnmes="cpnvalue"></Cpn>
    </div>
</template>

<script>
    import Cpn from './Cpn'
    export default {
        name:'App',
        props:{
            mes: {
                type: String,
                default: '',
                required: true
            }
        },
        components: {
            Cpn
        },
        data() {
            return {
                cpnvalue: 'cpnvalue-app'
            }
        }
    }
</script>

<style scoped>

</style>

main.js
==================
import {add, print} from './js/mathUtil.js'

let a = 1
let b = 2
print('a+b=' + add(a+b))

import Vue from 'vue'
import App from './vue/App'

const app = new Vue({
    el: '#app',
    template: '<div><App :mes="message"></App></div>',
    components: {
      App
    },
    data: {
        message: 'hello webpack-vue'
    }
})

require('./css/common.css')
require('./css/special.less')


index.html
================
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>测试</title>
</head>
<body>

<div>
    test
</div>
<hr>
<div id="app">
</div>
</body>
<script src="dist/bundle.js"></script>
</html>


webpack.config.js
=====================
module.exports = {
   module:{
        rules:[{
            test: /\.vue$/,
            use:['vue-loader']
        }]
    }
    resolve: {
        // import可省略后缀
        extensions: ['.js', '.css', '.vue'],
        alias: {
            'vue$': 'vue/dist/vue.esm.js'
        }
    } 
}

```

