# art-template

## 1. 初体验

 ```js
<script type="text/javascript" src="node_modules/art-template/lib/template-web.js"></script>
	<!-- 声明摸板text/template -->
	<script type="text/template" id="tpl-cus">
		hello {{ name }}
		我喜欢： {{each hobbies}} {{ $value }} {{/each}}
	</script>

	<script>
		var tmp = template('tpl-cus', {
			name : 'World',
			hobbies: [
				'music',
				'game'
			]
		});
		console.log(tmp);

	</script>
 ```

## 2. 在js中使用art-template

```js
var art_temp = require('art-template');

var res = art_temp.render('hello {{ name }}',{
	name : 'World'
});

console.log(res);
```

## 3. 地址

- [github地址](https://github.com/aui/art-template)

- [官方网站]()

  