## IO模块

### 1. fs模块(所有操作都是异步的)

```
require('fs');
```

- 读取 

  ```js
  fs.readFile('路径', function(error, data) {})
  ```

- 写入

  ```
  fs.writeFile('路径','内容',function (error) {})
  ```

  

