# Express框架

## 1. start

```js
var express = require('express');

// 创建应用程序 http.createServer
var app = express();

app.get('/', function(req, res) {
	res.send('hello express');
})
app.get('/about', function(req, res) {
	res.send('hello express, nihao');
})

app.listen(3000,function() {
	console.log('express start...');
})
// 指定静态资源路径
// 静态资源需要在开头加/public
app.use('/public/', express.static('./public/'))
// 静态资源直接/就能访问
app.use(express.static('./public/'))
app.use('/node_modules/', express.static('./node_modules/'))
```

- get('/') get请求
- post('/') post请求
- res.send 替代原来的end
- req.query 相当于原先的url.parse(,true).query

## 2. 在Express中是用art-template

- 安装

```shell
npm i --save art-template
npm i --save express-art-template
```

- 使用

```js
// 这样就可以在express框架中开启art-template模板
// 默认是art结尾的文件
// app.engine('art', require('express-art-template'))
// 然后就可以用res.render('')来使用摸板了

app.engine('html', require('express-art-template'))

app.get('/error', function(req, res) {
	// express 默认会去views路径下面找文件，因此文件不用加views路径
	// 当然也可以修改默认路径
	// app.set('view','路径')
	res.render('404.html',{
		title:'404'
	})
})

```

- 重定向

```js
res.redirect('/')
```

## 3. POST 请求

- 方法

```js
app.post('/')
```

- 获取表单数据：需要第三方插件 body-parse

  - 安装

  ```shell
  npm i --save body-parse
  ```

  - 配置

  ```js
  app.use(bodyParser.urlencoded({extended:false}))
  app.use(bodyParser.json())
  ```

  - 获取

  ```js
  req.body
  ```

## 4.router

-  router.js

```js
var express = require('express')
var router = express.Router()

router.get('/about', function(req, res) {
	res.send('hello express, nihao')
})
module.exports = router

```

- index.js挂载router

```js
app.use(require('./router'))
```



