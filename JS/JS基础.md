# JS基础

## 1. 回调函数

- 由于无法拿到异步请求数据，因此可以用回调函数

```js
test(function(data) {
	console.log(data)
})

function test(callback) {

	setTimeout(function() {
		callback("1+1=2")
	}, 1000)
}
```



## 2. let和var
- let为ES6新增的
- 作用域
  + let是有块级作用域
  ```html
  <button>按钮1</button>
  <button>按钮2</button>
  <button>按钮3</button>
  <script type="text/javascript" src="./vue.js"></script>
  <script type="text/javascript">
  	var bts = document.getElementsByTagName('button')
  	for (var i = 0; i < bts.length; i++) {
  		bts[i].addEventListener('click', function() {
  				console.log(i) // 使用let
  				声明，打印出来是当时的i
  			})
  	}
  	i= 100
  </script>
  ```
  + var没有作用域
    * 不同模块之间，变量冲突可能导致问题
    ```html
    <button>按钮1</button>
    <button>按钮2</button>
    <button>按钮3</button>
    <script type="text/javascript" src="./vue.js"></script>
    <script type="text/javascript">
    	var bts = document.getElementsByTagName('button')
    	for (var i = 0; i < bts.length; i++) {
    		bts[i].addEventListener('click', function() {
    			console.log(i) // 使用var声明，打印出来全是100, 说明i不是当时传入的i，而是最终的i
    		})
    	}
    	i= 100 // 在for循环外，仍可以操作i，并且会影响到内部的参数
    </script>
    ```
    * 解决方法：闭包(原理就是ES6前，if、for是没有块级作用域的，而函数有)
    ```html
    <button>按钮1</button>
    <button>按钮2</button>
    <button>按钮3</button>
    <script type="text/javascript" src="./vue.js"></script>
    <script type="text/javascript">
    	var bts = document.getElementsByTagName('button')
    	for (var i = 0; i < bts.length; i++) {
    		(function(num) {
    			bts[num].addEventListener('click', function() {
    			console.log(num) // 使用闭包，打印出来是当时的i
    		})
    		})(i)
    	}
    	i= 100
    </script>
    ```

## 3. const 常量 (ES6新增，类似java的final，必须赋值且不可改变地址)
- `const name` // 错误，没有赋值
- `const name = 'zhangsan'; name = 'lisi;'` // 错误，不可改变地址
- `const obj = {}; obj.name = 'zhangsan;'` // 正确，地址没有发生改变

## 4. Promise函数(ES6)

- 解决回调地狱callback hell（多嵌套->链式调用）
- 语法

```js
const p1 = new Promise(function(resolve, reject) {
	// async任务
	if (true) {
		resolve('调用成功')
	} else {
		reject('调用失败')
	}
})

p1.then(function(data) {
	console.log('调用成功：' + data)
}, function(err) {
	console.log('调用失败:' + err)
})
```

- then可以链式调用，return 一个promise即可

```js
const p1 = new Promise(function(resolve, reject) {
	// async任务
	if (true) {
		resolve('调用成功p1')
	} else {
		reject('调用失败p1')
	}
})

const p2 = new Promise(function(resolve, reject) {
	// async任务
	if (true) {
		resolve('调用成功p2')
	} else {
		reject('调用失败p2')
	}
})

p1.then(function(data) {
	console.log('调用成功：' + data)
	return p2
}, function(err) {
	console.log('调用失败:' + err)
}).then(function(data) { 
	console.log('调用成功2：' + data)
}, function(err) {
	console.log('调用失败2:' + err)
})
```

- 而，如果then return一个字符串，则继续then调用，第一个function返回的就是这个return的字符串值

```js
p1.then(function(data) {
	console.log('调用成功：' + data)
	return 'hello es6'
}, function(err) {
	console.log('调用失败:' + err)
}).then(function(data) { 
	console.log('调用成功2：' + data) // data=hello es6
}, function(err) {
	console.log('调用失败2:' + err)
})
```

- 封装promise，做优雅调用

```js
const fs = require('fs')

function promiseFS(filePath) {
	return new Promise(function(resolve, reject) {
		fs.readFile(filePath, function(err, data) {
			if (err) {
				reject(err)
			} else {
				resolve(data)
			}
		})
	})
}

promiseFS('./txt/a.txt')
	.then(function(data) {
		console.log(data.toString())
		return promiseFS('./txt/b.txt')
	})
	.then(function(data) {
		console.log(data.toString())
		return promiseFS('./txt/c.txt')
	})
```

## 5. 对象字面量增强写法（ES6）

- 对象字面量`const obj = {}`, 而不是`new Object()`

- ES5写法

  ```js
  const name = "zhangsan"
  const age = 15
  const obj = {
      name : name,
      age : age,
      run: function() {
          
      }
  }
  ```

- ES6写法

  ```js
  const name = "zhangsan"
  const age = 15
  const obj = {
      name,
      age,
      run() {
          
      }
  }
  ```

## 6. 数组splice方法：增删 

- 删除：splice(m,n)删除下标m-(m+n)的元素 

  ```js
   ['1', '2', '3', '4'].splice(1,2)  ==> ['1', '4']
  ```

- 插入：splice(m,0, ele) 在下标为m的地方插入ele，m往后的元素依次后移

  ```js
   ['1', '2', '3', '4'].splice(1,0,'5')  ==> ['1','5', '2', '3', '4']
  ```

- 替换：splice(m,n,a,b,...,z) 将下标为m开始的后n个元素替换成a b ...z

  ```js
   ['1', '2', '3', '4'].splice(1,2,'5','6')  ==> ['1', '5', '6', '4']
  ```

## 7.  for-in for-of语法

- for-in ，可以拿到数组的下标

  ```js
  for (let i in arr) {
      console.log('下标为：' + i)
  }
  ```

- for-of ，可以拿到元素

  ```js
  for (let item in arr) {
      console.log('元素为：' + item)
  }
  ```

## 8. Array.join 拼接

- ['A', 'B', 'C'].join('=')  ----> A=B=C

## 9. export/import （ES6）

- export 什么变量名，则import就用什么名
- import想要自定义名称，有两种方式
  - export default ，则import可以直接命新名。但是export default只能导出一个对象
  - import * as xxx from 也可以

## 10.  箭头函数  =>（ES6）

- 和lambda大概一致，如果方法内部只有一行代码，则可以省略大括号，然后js会默认补上return

## 11. this

- 会一层一层作用域往外找到那个对象

```js
const obj = {
    func1() {
        setTimeOut(function(){// 内部的this为window，因为fuction会有个默认传参window? function是一个?
        },1000)
        setTimeOut(()=> {// 此时的this也就是func1的this，所以是obj对象
           } , 1000)
    }
}
```



## 12. 改变url不发送请求(前端路由关键)

- location.hash = 'baidu'
- history.pushState({}, '', 'baidu')
- history.back()上一个url   《===》 history.go(-1)
  - history.go(-n) 返回第前n个url，正数则前进n个
  - history.forward() 《===》history.go(1)
- history.replaceState({}, '', 'baidu')
- 