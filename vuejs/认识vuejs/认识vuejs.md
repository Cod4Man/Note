# 认识vuejs

## 1. 前端三大框架

- Vue.js
- Angular.js
- React

## 2. vue全家桶

- Core
- Vue-router
- Vuex

## 3. 编程范式

- js：命令式编程，一行一行执行
- vue：声明式编程

## 4. MVVM模型

- ![MVVM](E:\softwareNote\vuejs\img\MVVM模型.png)

## 5. 响应式原理 

![1606632843798](E:\SoftwareNote\vuejs\img\vue响应式原理.png)

- app.message修改数据，Vue内部是如何监听message数据的改变

  Object.defineProperty 监听对象属性的改变(可以监听getter/setter)

- 当数据发生改变，Vue是如何知道要通知哪些人，界面发生刷新

  发布订阅者模式（解析摸板时，订阅。发生改变时，通知订阅者）