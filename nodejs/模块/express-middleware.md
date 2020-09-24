# express-Middleware 中间件

## 1. 概念

- 不关心请求路径和请求方法的中间件，任何请求都能进这个方法
- 中间件本身是一个方法，该方法接受三个参数
  - Request 请求对象
  - Response 响应对象
  - next 下一个中间件

```js
app.use(function(req, res, next) {// 中间操作})
```

- 当一个请求进入一个中间件后，如果不调用 **next()**方法，则不会执行下一个 **匹配的**中间件

```js
// 只能匹配请求url为/a开头 
app.use('/a', function(req, res, next) {// 中间操作})
```

- 严格匹配get()

```js
// 严格匹配'/'才可以
app.get('/',function(req, res, next) {// 中间操作})
```

- 当调用next(err)并传参，则直接跳转app.use(function(err, req, res, next) {}), 
  - 用来做全局错误中间件