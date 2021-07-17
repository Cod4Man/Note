# mysql

## 1. 安装

- ```shell
  npm i mysql
  ```

## 2. 实例

```js
const mysql = require('mysql')

// 创建连接
const connection = mysql.createConnection({
	host: 'localhost',
	user: 'root',
	password: '123',
	database: 'smbms'
})

// 连接
connection.connect()

// 查询
// 增删改查都用query
connection.query('select * from smbms_user', function(err, results, fields) {
	if (err) throw err;
	for (var i = 0; i < results.length; i++) {
		console.log('results[' + i + ']:' + JSON.stringify(results[i]) + '\n')
	}
	
})

// 关闭连结
connection.end()
```

