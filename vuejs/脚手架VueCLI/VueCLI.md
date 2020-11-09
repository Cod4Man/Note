# VueCLI

## 1. 脚手架CLI: Command-Line Interface

- 在Vue.js开发大型项目，需要考虑到目录结构/项目结构/部署/热加载/单元测试等，就需要用到VueCLI
- VueCLI可以快速搭建Vue项目和对应的webpack配置

## 2. vue-cli 创建项目

- vue-cli2

```shell
vue init webpack 项目名称
```

- vue-cli3

```shell
vue create 项目名称
```



## 3. runtime模式

- VueCLI中，两种模式对比runtime-compiler/runtime-only

  `runtime-compiler: template->ast->render->virDOM->UI`

  `runtime-only:render->virDOM->UI `

		 runtime-only性能更高，代码量更少



## 4. Vue-CLI3和2的区别

- 3是基于webpack4打造的，而2是webpack3
- 3的设计原则是“0配置”，移除了配置文件目录build和config
- 3提供了vue ui命令，是可视化配置，更加的人性化
- 移除了static目录(不会打包，原封不动的拷贝过去)，增加了public目录，将index.html移入public

### 4.1 vue-cli3隐藏了配置，可以自定义配置

- 在项目根目录增加一个vue.config.js（不可改名）
- 然后`module.exports={}`

#### 4.1.1 路径别名

```js
// cli3 会和配置文件合并
module.exports = {
    configureWebpack: {
        resolve: {
            // extensions: [], // 省略后缀
            // 别名
            alias: {
                'assets': '@/assets',
                'common': '@/common',
                'components': '@/components',
                'network': '@/network'
            }
        }
    }
}
```





## 5. vue ui 可视化脚手架

