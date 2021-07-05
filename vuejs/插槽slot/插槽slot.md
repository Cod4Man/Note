# 插槽slot

## 1. 插槽

```html
<div id="app">
    <cpn ref="cpnid">
        <span>测试</span>
        <span slot="third">测试</span>
    </cpn>
</div>

</body>
<template id="comp-son">
    <div>
        <slot><button>默认button</button></slot>
        <span>固定值</span>
        <slot name="second"><span>第二个插槽</span></slot>
        <slot name="third"></slot>
    </div>
</template>
<script src="../js/vue.js" ></script>
<script>
    const cpn = {
        template: '#comp-son',
        data() {
            return {
                name: '子组件name'
            }
        }
    }
    const app = new Vue({
        el: '#app',
        components: {
            cpn
        }
    })
</script>
```

- 插槽`<slot>`里面可以写默认实现，没有被替换则展示默认内容

- 具名插槽，`<slot name='slotname'>` 在插槽中定义一个name，然后插入时，用`<span slot='slotname'>` 来找到相应的插槽进行插入

  

## 2. 作用域插槽：父组件可以对子组件的内容进行变更

