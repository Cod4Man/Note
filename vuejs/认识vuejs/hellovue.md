# hellovue

# 1. hellovue.demo

```html
<body>
<div id="app">{{ message}}</div>
<script src="../js/vue.js"></script>
<script>
    // ES6后使用let/const
    // let:变量
    // const: 常量， 再次赋值会报错
    // var 早期设计缺陷，比如没有作用域
    const app = new Vue({
        el :'#app',
        data : {
            message : 'Hello Vue'
        }
    })
</script>
</body>
```

