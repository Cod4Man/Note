# vuex

## 1. 定义

- 是专门为Vue.js应用程序开发的状态管理模式
- 采用***集中式存储管理*** 应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生改变(***响应式***)。
- 状态管理的定义
  - 将多个组件共享的变量全部存储到一个对象中，然后封装在顶层Vue中，这样所有的组件都可以享用。就像父类一样

![1603609871913](img\Vuex状态管理图例.png)



## 2. 使用

- 像vue-router一样

### 2.1  配置文件

```js
// store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
    state: {
        counter: 1000
    },
    mutations: {
        increment(state) {
            state.counter++;
        },
        decrement(state) {
            state.counter--;
        }
    }
})


export default store
```

### 2.2 vue实例引用

```js
new Vue({
  render: h => h(App),
  router: router,
  store
}).$mount('#app')

```

### 2.3 Vue实例中使用

```js
<template>
  <div id="app">
    <h2>{{$store.state.counter}}</h2>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  },
  methods: {
      clickAdd() {
        this.$store.commit('increment')
      },
      clickDe() {
          this.$store.commit('decrement')
      }
  }
}
</script>

<style>

</style>

```

### 2.4 store.mutations 

- mutation响应式的前提：

  - （**新版本已经可以**）数据中的参数必须是提前定义好的，才能被监听到，如果是后面加入的属性，是不可被监听的，因此不会是响应式的

    `state.info[newColumn]= 'asdasd'`

    可用Vue.set(state.info, 'newColumn', 'asdasd')

  - （**新版本已经可以**）同理，delete也是一样

    可用Vue.delete(state.info, 'newColumn')

- 通常情况，vuex要求我们的mutation里面方法必须是同步方法，因为我们使用devtools进行调试时，devtools不能很好的追踪异步操作是什么时候完成的。

- 如果非要异步操作，可以使用Action来进行中转Mutation

- mutation的类型常量：就是保证mutation定义的方法名和使用方commit调用名一致, 可以定义一个常量js

```js
import INCREMENT from './constans/mutation-type' // INCREMENT ='increment'
mutations: {
        [INCREMENT](state) {
            state.counter++;
        },
        decrement(state) {
            state.counter--;
        }
    }
```



- 定义一些变量的方法

```js
// 配置
const store = new Vuex.Store({
    state: {
        counter: 1000
    },
    mutations: {
        increment(state) {
            state.counter++;
        },
        decrement(state) {
            state.counter--;
        }
    }
})

// 调用

 methods: {
      clickAdd() {
        this.$store.commit('increment')
      },
      clickDe() {
          this.$store.commit('decrement')
      }
  }
```



- mutation的payload（负载【携带参数】）

```js
// 配置
const store = new Vuex.Store({
    state: {
        counter: 1000
    },
    mutations: {
        increment(state, count) {
            state.counter += count;
        }
    }
})

// 调用
 methods: {
      clickAdd() {
        this.$store.commit('increment', 16)
      }
  }
```

- 特殊的提交封装

```js
// 配置
const store = new Vuex.Store({
    state: {
        counter: 1000
    },
    mutations: {
        increment(state, payload) {
            // 此时拿到的是整个对象
            /*{
              type: 'increment',
              name: 'asdad',
              age: 15
         	}*/
            let name = payload.name
            let age = payload.age
        }
    }
})
// 调用
 methods: {
      clickAdd() {
          this.$store.commit({
              type: 'increment',
              name: 'asdad',
              age: 15
          })
      }
  }
```





### 2.5 store.getters

- 当共享变量需要做一些操作才能返回实际值时(如+-*/)，可以定义再getters里面(类似vue实例的computed)

```html
const store = new Vuex.Store({
    state: {
        counter: 1000,
        students: [
            {
                name: 'zhangsan',
                age: 18
            },
            {
                name: 'lisi',
                age: 28
            }
        ]
    },
    mutations: {
        increment(state) {
            state.counter++;
        },
        decrement(state) {
            state.counter--;
        }
    },
    getters: {
        large20(state) {
            return state.students.filter(stu => stu.age > 20)
        },
        large20Length(state, getters) {
            return getters.large20.length
        },
        largeCusAge(state) {
            // 返回一个函数供调用
            return age => state.students.filter(stu => stu.age > age)
        }
    }
})


<template>
  <div id="app">
    <h2>{{$store.state.counter}}</h2>
    <h2>{{$store.getters.large20}}</h2>
    <h2>{{$store.getters.large20Length}}</h2>
    <h2>{{$store.getters.largeCusAge(11)}}</h2>
  </div>
</template>
```



### 2.6 store.action

- 通常情况，vuex要求我们的mutation里面方法必须是同步方法，因为我们使用devtools进行调试时，devtools不能很好的追踪异步操作是什么时候完成的。
- 如果非要异步操作，可以使用Action来进行中转Mutation

```js
actions: {
      asyncIncrement(context, payout) {
          return new Promise((res, rej) => {
              setTimeout(() => {
                  context.commit('increment')
                  res('success')
              }, 1000)
          })
      }
    }


clickAddAsync() {
        this.$store.dispatch('asyncIncrement', '成功')
            .then((data) => {
                console.log(data);
            })
      },
```

### 2.7 store.module

- 因为所有状态都放在一起就显的很臃肿，因此vuex提供了module的概念，每个module都可以有自己的mutation/action/getters/state
- 调用的方式和原来一样，并且也不需要加上模块名，但是需要和主方法中的方法名区分开(相当于merge)
- getters有第三个参数rootState, 可以在module的getters中引用主store中的参数



### 2.8 vuex目录结构

![1603723834701](img\vuex目录结构.png)

### 2.9 mapGetters: 

- 将store.getters暴露给组件直接使用,无需再$store.getters.xxx

```js
import mapGetters from 'vuex'

new Vue({
    el: '#app',
    conputered: {
        // 方法一
        //...mapGetters(['large20Length', 'large20'])
        // 方法二 ： 重命名
        ...mapGetters({
            length: 'large20Length',
            c20: 'large20'
        })
    }
})

// 然后便可在该Vue实例中使用length等方法
```





## 3. 类型Vuex的功能：事件总线$bus

组件之间层次过多，使用父子传输很不方便，可以将变量发送到一个公共的地方进行管理

#### 3.1 在Vue原型中添加一个$bus

- Vue.prototype.$bus = new Vue(),  因为vue实例可以发送事件\$emit()

#### 3.2 共享数据方，发送事件

- this.$bus.\$emit('imgLoaded')

#### 3.3 数据获取方，监听事件

- this.$bus.\$on('imgLoaded', () => {this.$refs.scroll.refresh()})

#### 3.4 取消监听

- this.$bus.\$off('imgLoaded')