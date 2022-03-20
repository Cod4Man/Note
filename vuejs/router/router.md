# router

## 1. 简单应用

### 1.1 配置路由

```js
import VueRouter from 'vue-router'
import Vue from 'vue'
import Home from '../components/Home'
import About from '../components/About'


// 注入组件
Vue.use(VueRouter)
// 配置router
const routes = [
    {
        path: '', // 缺省router
        redirect: '/home' // 重定向
    },
    {
        path: '/home',
        component: Home
    },
    {
        path: '/about',
        component: About
    }

]
// 创建router实例
const router = new VueRouter({
    routes // 注意名字，因为是ES6的增强写法，所以变量名需要一致
})
// 导出路由
export default router
```

### 1.2 挂载路由

```js
new Vue({
  render: h => h(App),
  router: router
}).$mount('#app')

```

### 1.3 使用

```html
<template>
  <div id="app">
    <!--<img alt="Vue logo" src="./assets/logo.png">-->
    <HelloWorld msg="Welcome to Your Vue.js App"/>
    <router-link to="/home" tag="button">首页</router-link>
    <router-link to="/about">关于</router-link>
    <router-view></router-view> <!--组件展示位置-->
    <hr>
  </div>
</template>
```



## 2. route-link

### 2.1 属性

- tag: 传标签名。可以指定route-link渲染成什么标签(默认`<a/>`)

  ```html
  <router-link to="/home" tag="button"></router-link>
  <router-link to="/home" tag="li></router-link>
  ```

- replace：不会留下history痕迹，也就是浏览器无法“上一步“，”下一步“找到记录

  原理是：使用replace做的跳转是`history.replaceState({}, '', 'baidu')`方法，而不是`history.pushState({}, '', 'baidu')`

- active-class: 顾名思义，就是点击后多了一个active的属性。默认不做配置，vue会自动给组件增加名为router-link-active的属性，这样开发就可以对这个属性进行操作(如点击的高亮显示等等)

  ```js
  <router-link to="/home" tag="button" replace active-class="active">首页</router-link>
  <router-link to="/about" replace>关于</router-link>
  ```

  - 也可显式的增加active-class属性来修改属性名
  - 对于多router组件，也可在router统一配置来修改active属性名

  ```js
  // 创建router实例
  const router = new VueRouter({
      routes,
      mode: 'history', // 默认是hash模式，url会带#
      linkActiveClass: 'active2'
  })
  ```

  

## 3. 不使用router-link，在函数中动态的跳转

- vue实例的$router对象方法：

```js
methods: {
      goHome() {
          console.log('goHome');
          this.$router.push('/home')
      },
      goAbout() {
          console.log('goAbout');
          this.$router.replace('/about')
      }
  }
```



## 4. 动态路由

### 4.1 

### - router配置：用`:` ＋变量名

 ```js
  {
          path: '/home/:id',
          component: Home
      }
 ```

### - 动态传参: $route对象为当前活跃的路由(active)
```html
<router-link to="/home/zhangsan" tag="button">首页带id, id为：{{$route.params.id}}</router-link>
```
### 4.2 query: 参数会拼接到url上，就像get请求一样

```html
<router-link :to="{path:'/profile', query:{name: 'zhangsan', age: '15'}}" tag="button" >我的</router-link>

<template>
    <div>
        <h2>这是Profile组件</h2>
        <h2>{{$route.query.name}}</h2>
        <h2>{{$route.query.age}}</h2>
    </div>
</template>


```

- 动态传参

```js
goProfile() {
    this.$router.push({
        path: '/profile',
        query: {
            name: 'zhangsan',
            age: 15
        }
    })
}
```



## 5. 路由懒加载

- 打包构建项目，所有东西都放在一个js文件中，每次请求都需要加载(大部分是不想管的东西)，用户体验差

- 路由懒加载可以把不同路由打包成不同的js文件，一个请求只加载对象的js

- 懒加载的三种方式

  ![1602408161709](img\路由懒加载.png)

```js
// import Home from '../components/Home'
// import About from '../components/About'
const Home = () => import('../components/Home.vue')
const About = () => import('../components/About.vue')
```

## 6. 路由的嵌套

- router配置：children

```js
// 配置router
const routes = [
    {
        path: '',
        redirect: '/home' // 重定向
    },
    {
        path: '/home',
        component: Home,
        children: [
            {
                path: '',
                redirect: 'news' // 不加/
            },
            {
                path: 'news',
                component: HomeNews
            },
            {
                path: 'music',
                component: HomeMusic
            }
        ]
    },
    {
        path: '/about',
        component: About
    },
    {
        path: '/home/:id',
        component: Home
    }

]
```

- 在路由组件中引入路由

```js
<template>
    <div>
        <h2>这个是Home组件</h2>
        <router-link to="/home/news">新闻</router-link>
        <router-link to="/home/music">音乐</router-link>
        <router-view></router-view>
    </div>
</template>

<script>
    export default {
        name: "Home"
    }
</script>
```

## 7. 导航守卫

- 在路由中定义meta元数据，则可以通过router.beforeEach函数拿到，

  这里就可以用来动态修改各页面的参数，如title

```js
{
    path: '/about',
        component: About,
            meta: {
                title: '关于About'
            }
},

router.beforeEach((to, from, next) => {
    console.log(to);
    document.title = to.matched[0].meta.title
    next() // 注意next(), 和node那个类似
})
```

- 前置守卫：router.beforeEach((to, from, next)=>}, 需主动调用next()
- 后置勾子：router.afterEach((to,from)=>) , 无需主动调用next()
- 导航守卫也称作全局首位，都可以用。对应的还有路由独享的守卫，组件内的首位

## 8. Keep-Alive: 保持活着(以免被频繁销毁/创建)

- 用法

```html
// 包裹着router-view
<keep-alive>
	<router-view></router-view>
</keep-alive>
/*  这些方法都写在router-component里面*/
// 使用声明周期函数：
activated() {
	// 该路由活跃
	// 读取缓存
	this.$router.path = this.path
},
deactivated() {
	// 该路由不活跃
},
beforeRouteLeave((to, from ,next) => {
	// 缓存path
	this.path = this.$router.path
	next()
})
```

- activated和deactivated生命周期函数在有`<keep-alive/>`标签时才生效

- beforeRouteLeave路由不活跃时调用

- 排除router-view里面的某些路由keepAlive：exclude，支持正则匹配组件

  `<keep-alive exclude="ComponentName1,ComponentName2"></keep-alive>`

- 包含router-view里面的某些路由keepAlive：include，支持正则匹配组件

  `<keep-alive include="ComponentName1,ComponentName2"></keep-alive>`