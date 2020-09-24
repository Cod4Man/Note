# mongoose

## 1. mongoose是第三方模块，官方的是node-mongodb-navite

## 2.使用

```js
var mongoose = require('mongoose')

// 创建数据库连接
mongoose.connect('mongodb://localhost/test', {useMongoClient: true})

mongoose.Promise = global.Promise


var  Cat = mongoose.model('Cat', {name: String})

var kitty = new Cat({name:'xiaoMiao'})

kitty.save(function(err) {
	if (err) {
		console.log(err)
	} else {
		console.log('插入成功')
	}
})
```

## 3. 官方指南

### 3.1 设计Scheme发布Model

```js
var mongoose = require('mongoose')

var Schema = mongoose.Schema

// 1. 连接数据库

mongoose.connect('mongoose://localhostL/test')

// 2. 设计文档结构（表结构）
var userSchema = new Schema({
	username: {
		type: String,
		required: true
	},
	password: {
		type: String,
		required: true
	},
	email: {
		type: String
	}
})

// 3. 将文档结构发布为模型
var User = mongoose.model('User', userSchema)

```

## 3.2 CRUD

- C

  ```js
  var user = new User({
  	username: '张桑',
  	password: 'zhj123',
  	email: 'asd@163.com'
  })
  
  user.save(function(err) {
  	if (err) {
  		console.log(err)
  	} else {
  		console.log('插入成功')
  	}
  })
  ```

- R

  ```js
  // 查询全部
  User.find(function(err, data) {
  	if (err) {
  		console.log(err)
  	} else {
  		console.log(data)
  	}
  }) 
     
   // 条件查询
   
   User.find({
   	username: '张桑'
   }, function(err, data) {
  	if (err) {
  		console.log(err)
  	} else {
  		console.log(data)
  	}
  })
  
   // 查找一个
   User.findOne({
   	username: '张桑'
   }, function(err, data) {
  	if (err) {
  		console.log(err)
  	} else {
  		console.log(data)
  	}
  })
  ```

- U

  ```js
  // 更新全部
  User.update(conditions, doc, [options], [callback])
  
  // 根据条件更新一个
  User.findOneAndUpdate([conditions], [update], [options], [callback])
  
  // 根据ID更新一个：
  User.findByIdAndUpdate(id, [update], [options], [callback])
  ```

- D

  ```js
  // 根据条件删除全部
  User.remove({
   	username: '张桑'
   }, function(err, data) {
  	if (err) {
  		console.log(err)
  	} else {
  		console.log(data)
  	}
  })
  
  // 根据条件删除一个
  User.findOneAndRemove(conditions, [options], [callback])
  
  // 根据ID删除一个
  User.findByIdAndRemove(id, [options], [callabck])
  ```

  