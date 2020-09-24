# 、http模块

## 1. http模块

```js
	var http = require('http');
```

- 创建server实例

  ```js
  var server = http.createServer();
  ```

- 请求事件处理并响应

  ```js
  server.on('request',function(request, response){
      console.log('get request, url:' + request.url);	
  	response.setHeader('Content-type', 'text/plain;charset=utf-8')
      // response 最后需要调用end()结束响应
  	response.write('收到了收到了'.toString());
  	response.write('over'.toString());
  	response.end();
  })
  ```

  - response响应需要以response.end();结尾

  - 其中end，支持两种数据格式：二进制和字符串

  - 也可二合一，发送数据后结束

    ```js
    response.end('response over');
    ```

  - 响应内容Content-type

    ```js
    response.setHeader('Content-type','text/plain;charset=utf-8')
    ```

    - text/plain : 普通文本 <a>点我</a>
    - tetx/html : 渲染页面 <a>点我</a>

- 设置端口号

  ```js
  server.listen(3000, function() {
  	console.log('server started! into http://localhost:3000/ to visitor web');
  });
  ```

  

## 2. 重定向
### 2.1 临时重定向

- 步骤1： 状态码改为302

 ```js
  response.statusCode=302
 ```

- 步骤2： 重定向路径：Location

```js
response.setHeader('Location','/url')
```

### 2.2 永久重定向
- 状态码为301
- 和临时重定向的区别
	+ 永久重定向浏览器会己住
	+ 如:a.com->b.com
	  * 临时重定向： 每次都会请求a
	  * 永久重定向： 直接跳转b
	  

