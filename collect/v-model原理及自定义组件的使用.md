# v-model原理及自定义组件的使用

### v-model 是什么

1. v-model 即可以作用在普通表单元素上，又可以作用在组件上。

2. vuejs隐式添加 value 的 prop，子组件通过 props.value 接收值。

3. 子组件通过 this.$emit('input')，改变父组件 v-model 绑定的值。

4. v-model 可以实现双向绑定，无需定义接收事件。

### v-model原理

v-model实际上时语法糖，下面就是语法糖的构造过程。 

而v-model自定义指令下包裹的语法是input的value属性、input事件，整个过程是： 

```
<input v-modle="inputV" />
// 等同于
<input :value="inputV" @input="inputV = $event.target.value"/>
```

