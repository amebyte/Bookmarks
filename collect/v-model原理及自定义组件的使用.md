# v-model原理及自定义组件的使用

### v-model 是什么

1. v-model 即可以作用在普通表单元素上，又可以作用在组件上。

2. vuejs隐式添加 value 的 prop，子组件通过 props.value 接收值。

3. 子组件通过 this.$emit('input')，改变父组件 v-model 绑定的值。

4. v-model 可以实现双向绑定，无需定义接收事件。

### v-model原理

v-model实际上时语法糖，下面就是语法糖的构造过程。 

而v-model自定义指令下包裹的语法是input的value属性、input事件，整个过程是： 

```javascript
<input v-modle="inputV" />
// 等同于
<input :value="inputV" @input="inputV = $event.target.value"/>
```

正常情况下当你在父组件里给子组件绑定 v-model属性时，子组件中会默认的将 v-model绑定的数据，付给子组件 名为 value的props属性，value依然需要提前在子组件props中声明，否则接收不到。 当 value修改后，并不会立即双向回传给父组件。如果想回传实现同步更新父组件的v-model需要如下操作：

```javascript
 this.$emit('input', value) 
```

当未声明双向绑定回传的事件时，默认通过input事件回传，为什么说 “当未声明双向绑定回传的事件”,这个便是第二种方式，下面会讲到。 简单来说，第一种方式的实现，首先是v-model绑定父组件数据 ，然后子组件value 的props属性自动接收，最后当数据更改后调用this.$emit('input', value) 回传父组件，这样父组件不需要额外实现子组件的回传就可以实现双向绑定 第二种方式： 前面提到 “当未声明双向绑定回传的事件” 默认使用 input回传，既然这样说了那就存在，如果我不用input呢？这就需要了解vue的一个特殊的属性：model， 这个属性可以用来声明 子组件用哪个字段去接收双向绑定的数据，以及用哪个方法回调更新父组件v-model的数据，写法如下：

```javascript
export default {
  name: 'CommonCkeckBox',
  model: {
    prop: 'checked',
    event: 'changeCheck'
  },
    props: {
    checked: {
      type: Boolean,
      default: false,
    }, // 选中状态
  }
  }

```

这种写法就意味着，父组件 双向绑定的数据会绑定到子组件名为checked的props属性，并且，当子组件调用this.$emit('changeCheck', value)时，会同步的更新父组件的数据，实现双向绑定。 

一个组件上的v-model默认会利用名为value的 prop 和名为input的事件，但是像单选框、复选框等类型的输入控件可能会将valueattribute 用于[不同的目的](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FHTML%2FElement%2Finput%2Fcheckbox%23Value)。model选项可以用来避免这样的冲突： ———— 摘自 vue 官网 

```javascript
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
}) // ————  摘自 vue 官网
```



默认情况下，一个组件上的 v-model 会把 value 用作 prop 且把 input 用作 event。

但是像单选框、复选框等类型的输入控件可能会将 value attribute 用于不同的目的。model 选项可以用 来避免这样的冲突。

```javascript
new Vue({
 components: {
   CompOne,
 },
 data () {
   return {
     value: 'vue'
   }
 },
 template: `
   <div>
     <comp-one v-model="value"></comp-one> // 使用v-model 满足语法糖规则：属性必须为value，方法名必须为：input
   </div> 
 `,
}).$mount(root)

```

```javascript
// 修改子组件 
const CompOne = {
  props: ['value1'],
  model: {
      prop: "value1", // 接收的数据 value => value1
      event: "change" // $emit 需要绑定的事件 input => change
  },
  template: `
    <div>
      <input type="text" @input="handleInput" :value="value1"></input>
    </div>
  `,
  methods: {
    handleInput (e) {
      this.$emit('change', e.target.value)
    }
  }
}
```

### 总结

v-model 时一个语法糖，它做了：

1. 绑定数据value
2. 触发输入事件input
3. data 更新触发重新渲染