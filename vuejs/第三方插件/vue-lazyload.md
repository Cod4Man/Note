# vue-lazyload

## 1. vue-lazyload 介绍

- 可用于图片懒加载

## 2. 用法

### 2.1 安装 

- `npm i vue-lazyload --S`

### 2.2 导入

- `import VueLazyLoad from 'vue-lazyload'`

### 2.3 使用

```vue
Vue.use(VueLazyLoad, [options])

/*
options = {
	error: '图片加载失败'，
	loading: required('图片url'),
	...
}
*/

<img :src="www.baidu.com/asd.jpg"/>
更换为
<img vue-lazy="www.baidu.com/asd.jpg"/>
```

