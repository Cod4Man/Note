# plugin

## 1. webpack.BannerPlugin版权说明

```js
let webpack = require('webpack')

module.exports = {
	plugins: [
        new webpack.BannerPlugin('版权归属: Cod4Man ，翻版必究')
    ]
}
```



## 2. html-webpack-plugin 打包html

```js
let htmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
	plugins: [
        // 指定摸板，会拷贝指定的摸板
        new htmlWebpackPlugin({template: './index.html'})
    ]
}
```



## 3. uglifyjs-webpack-plugin JS压缩

```js
let uglifyjsWebpackPlugin = require('uglifyjs-webpack-plugin')

module.exports = {
	plugins: [
        new uglifyjsWebpackPlugin()
    ]
}
```

