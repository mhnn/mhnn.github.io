---
title: Vue组件通信方式
date: 2022-01-16 08:28:04
categories: 
- Vue
tags:
- Vue
- 学习
- 前端
---
对Vue组件通信方式进行的一个总结。摘录自微信公众号`web前端知识点`
<!-- more -->
## props/$emit
**props:** 可以是数组或者对象，用于接收父组件传递而来的属性；当props为对象时，可以通过type、default、required、validator等配置来设置属性的类型、默认值、是否必传和校验规则。
**$emit:** 用于触发父组件在子组件上绑定了v-on事件相应的监听。
**实例**
- 父向子传值：父组件通过`:messageFromParent="message"`将父组件message值传递给子组件，当父组件的input标签输入时，子组件p标签的内容就会改变。
- 子向父传值：父组件通过`@on-receive="receive"`在子组件上绑定了receive事件的监听，子组件input标签输入时，会触发receive回调函数，通过`this.$emit('on-receive',this.message)`将子组件message的值赋给父组件messageFromChild，改变父组件p标签的内容。
**子组件代码**
```html
<template>
  <div>
    <div class="child">
      <h4>this is child component</h4>
      <input type="text" v-model="message" @keyup="send" />
      <p>收到来自父组件的消息：{{ messageFromParent }}</p>
    </div>
  </div>
</template>
<script>
export default {
  name: "child",
  //通过props向父组件暴露属性
  props: ["messageFromParent"],
  data() {
    return {
      message: "",
    };
  },
  methods: {
    send() {
      //通过$emit触发on-receive事件，调用父组件中的
      //receive回调，并将this.message作为参数
      this.$emit("on-receive", this.message);
    },
  },
};
</script>
```
**父组件代码**
```html
<template>
  <div class="parent">
    <h3>this is parent component</h3>
    <input type="text" v-model="message" />
    <p>收到来自子组件的消息：{{ messageFromChild }}</p>
    <Child :messageFromParent="message" @on-receive="receive" />
  </div>
</template>
<script>
import Child from "./child";
export default {
  data() {
    return {
      //传递给子组件的消息
      message: "",
      messageFromChild: "",
    };
  },
  components: {
    Child,
  },
  methods: {
    //接受子组件的信息，并将其赋值给messageFromChild
    receive(msg) {
      this.messageFromChild = msg;
    },
  },
};
</script>
```
![父子组件传递](./父子组件传递.webp)
## v-slot
> v-slot是Vue2.6新增的用于统一实现插槽和具名插槽的Api，用于替代`slot`、`slot-scope`、`scope`等api。
> v-slot在template标签中用于提供具名插槽或需要接收prop的插槽，如果不指定v-slot，则取默认值default
**实例**
- 父向子传值：父组件通过`<template v-slot="child">{{ message }}</template>`将父组件的message传递给子组件，子组件使用`<slot name="child"></slot>`接收到相应内容。
**子组件**
```html
<template>
  <div class="child">
    <h4>this is child component</h4>
    <p>
      收到来自父组件的消息：
      <!-- 展示父组件通过插槽传递的{{ message }} -->
      <slot name="child"></slot>
    </p>
  </div>
</template>
```
**父组件**
```html
<template>
  <div class="parent">
    <h3>this is parent component</h3>
    <input type="text" v-model="message" />
    <Child>
      <template v-slot:child>
        <!-- 插槽要展示的内容 -->
        {{ message }}
      </template>
    </Child>
  </div>
</template>

<script>
import Child from "./slot-child";
export default {
  name: "Parent",
  data() {
    return {
      message: "",
    };
  },
  components: {
    Child,
  },
};
</script>
```
![v-slot应用](./v-slot应用.webp)
## \$refs/\$parent/\$children/\$root
> 我们也同样可以通过 `$refs/$parent/$children/$root` 等方式获取 Vue 组件实例，得到实例上绑定的属性及方法等，来实现组件之间的通信。
**$refs：**我们通常会将 $refs绑定在DOM元素上，来获取DOM元素的 attributes。在实现组件通信上，我们也可以将 $refs 绑定在子组件上，从而获取子组件实例。
**$parent:**我们可以在Vue中通过`this.\$parent`来获取当前组件的父组件实例（若有）。
**$children：**与parent同理。但需注意的是，`this.\$children`数组中的元素下标不一定就与父组件引用子组件的顺序一一对应，异步加载可能会影响其顺序，故使用时一定要根据一定条件（如name）去找相应组件。
**$root：**获取当前组件树的根Vue实例。若当前实例没有父实例，此实例将会是其自己。通过这个，我们可以实现组件之间的跨级通信。
**实例：**
`$parent`与`$children`
- 父向子传值：子组件通过`$parent.message`获取到父组件中message的值。
- 子向父传值：父组件通过`$children`获取子组件实例的数组，通过遍历找到name=Child1的子组件实例并将其赋值给child1，而后使用`child1.message`获取到child1子组件的message
**子组件**
```html
<template>
  <div class="child">
    <h4>this is child component</h4>
    <input type="text" v-model="message" />
    <p>来自于父组件的消息：{{ $parent.message }}</p>
  </div>
</template>

<script>
export default {
  name: "Child1",
  data() {
    return {
      message: "",
    };
  },
};
</script>
```
**父组件**
```html
<template>
  <div class="parent">
    <h3>this is parent component</h3>
    <input type="text" v-model="message" />
    <p>来自子组件的消息：{{ child1.message }}</p>
    <Child />
  </div>
</template>

<script>
import Child from "@/pages/test/$child";
export default {
  data() {
    return {
      message: "",
      child1: {},
    };
  },
  components: {
    Child,
  },
  mounted() {
    this.child1 = this.$children.find((child) => {
      //$options用于获取自定义属性，此处的name即为自定义属性
      return child.$options.name == "Child1";
    });
  },
};
</script>
```
![$children与$parent](./$child与$parent.webp)
## $attrs/$listener
> 属于Vue2.4新加入的属性。主要用来开发高级组件的。
**\$attrs：**用来接收父作用域中不作为props被识别的attribute属性，并可以通过`v-bind="$attrs"`传入内部组件。
> 试想一下，当你创建了一个组件，你要接收 param1 、param2、param3 …… 等数十个参数，如果通过 props，那你需要通过props:
> ['param1', 'param2', 'param3', ……]等声明一大堆。如果这些 props 还有一些需要往更深层次的子组件传递，那将会更加麻烦。
> 而使用 $attrs ，你不需要任何声明，直接通过$attrs.param1、$attrs.param2……就可以使用，而且向深层子组件传递上面也给了示例，十分方便。
**\$listeners：**包含了父作用域中的 v-on 事件监听器。它可以通过`v-on="$listeners" `传入内部组件——在创建更高层次的组件时非常有用，这里在传递时的使用方法和 $attrs 十分类似。
**实例**
在这个实例中，有三个组件：A、B、C。他们的关系是：
```js
A:{
  B:{
    C
  }
}
```
即A是B的父组件，B是C的父组件，C是最后一级（3级组件）。我们实现了：
- 父向子传值：A通过`:messageFromA="message"`将message属性传递给B，B通过`$attrs.messageFromA`获取到A的message。
- 跨级向下传值：A通过`:messageFromA="message"`将message传给B，B通过`v-bind="$attrs"`将其传给C，C通过`$attrs.messageFromA`获取到A的message值。
- 子向父传值：A通过`@keyup="receive"`在子孙组件上绑定keyup事件的监听，B通过`v-on="$listeners"`来将keyup事件绑定在其input标签上。当B的input输入内容时，便会触发A的receive回调——将B的input框内的值赋值给A的messageFromComp ，从而展示了子向父传值。
- 跨级向上传值：A通过`@keyup="receive"`在子孙组件上绑定keyup事件的监听，B通过`<CompC v-on="$listeners" />`将其继续传递给C。C通过`v-on="$listeners"`来将keyup事件绑定在其input标签上。当C的input输入内容时，便会触发A的receive回调——将B的input框内的值赋值给A的messageFromComp ，从而展示了子向父跨级传值。
**三级组件-C**
```html
<template>
  <div class="C">
    <h5>this is C component</h5>
    <input type="text" name="compC" id="" v-model="message" v-on="$listeners" />
    <p>收到来自A的消息：{{ $attrs.messageFromA }}</p>
  </div>
</template>

<script>
export default {
  name: "Compc",
  data() {
    return {
      message: "",
    };
  },
};
</script>
```
**二级组件-B**
```html
<template>
  <div class="B">
    <h4>this is B component</h4>
    <!--将A组件keyup的监听回调绑在该input上-->
    <input type="text" name="compB" v-model="message" v-on="$listeners" />
    <p>收到来自A组件的消息：{{ $attrs.messageFromA }}</p>
    <!--将A组件keyup的监听回调继续传递给C组件，将A组件传递的attrs继续传递给C组件-->
    <CompC v-bind="$attrs" v-on="$listeners" />
  </div>
</template>

<script>
import CompC from "./C";
export default {
  name: "CompB",
  components: {
    CompC,
  },
  data() {
    return {
      message: "",
    };
  },
};
</script>
```
**A组件**
```html
<template>
  <div class="A">
    <h3>this is A component</h3>
    <input type="text" v-model="message" />
    <p>收到来自{{ comp }}的消息：{{ messageFromComp }}</p>
    <!--监听子孙组件的keyup事件，将message传递给子孙组件-->
    <CompB :messageFromA="message" @keyup="receive" />
  </div>
</template>

<script>
import CompB from "./B";
export default {
  name: "CompA",
  data() {
    return {
      message: "",
      messageFromComp: "",
      comp: "",
    };
  },
  components: {
    CompB,
  },
  methods: {
    // 监听子孙组件keyup事件的回调，并将keyup所在input输入框的值赋值给messageFromComp
    receive(e) {
      this.comp = e.target.name;
      this.messageFromComp = e.target.value;
    },
  },
};
</script>
```
![](./跨级传递.webp)
## provide/inject
> 这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论层次有多深。并在其上下游关系成立的时间内始终生效。
> 这个类似于React的Context Api
**provide：**一个对象，或是一个返回对象的函数。包含可注入其子孙的property，即要传递给子孙的属性和属性值。
**inject：**一个字符串数组，或者是一个对象。当其为字符串数组时，使用方式和props十分相似，只不过接收的属性由data变成了provide中的属性。当其为对象时，也和props类似，可以通过配置default和from等属性来设置默认值，在子组件中使用新的命名属性等。
**实例**
此实例中有三个组件的关系和上一个例子相同。此例实现了：
- 父向子传值：A通过provide将message注入给子孙组件，B通过`inject:['messageFromA']`来接收A中的message，并通过`messageFromA.content`获取A的message的content值
- 跨级向下传值：A通过provide将message注入给子孙组件，C通过`inject:['messageFromA']`直接接收A的message即可
**组件A代码**
```html
// 1级组件A
<template>
  <div class="compa">
    <h3>this is A component</h3>
    <input type="text" v-model="message.content" />
    <CompB />
  </div>
</template>
<script>
import CompB from './compB'
export default {
  name: 'CompA',
  provide() {
    return {
      messageFromA: this.message,  // 将message通过provide传递给子孙组件
    }
  },
  data() {
    return {
      message: {
        content: '',
      },
    }
  },
  components: {
    CompB,
  },
}
</script>
```
**组件B代码**
```html
// 2级组件B
<template>
  <div class="compb">
    <h4>this is B component</h4>
    <p>收到来自A组件的消息：{{ messageFromA && messageFromA.content }}</p>
    <CompC />
  </div>
</template>
<script>
import CompC from './compC'
export default {
  name: 'CompB',
  inject: ['messageFromA'], // 通过inject接受A中provide传递过来的message
  components: {
    CompC,
  },
}
</script>
```
**组件C代码**
```html
// 3级组件C
<template>
  <div class="compc">
    <h5>this is C component</h5>
    <p>收到来自A组件的消息：{{ messageFromA && messageFromA.content }}</p>
  </div>
</template>
<script>
export default {
  name: 'Compc',
  inject: ['messageFromA'], // 通过inject接受A中provide传递过来的message
}
</script>
```
<div class="warning">

> 注意：
> 1. message使用object而不是string，是因为provide与inject并不是可响应的
> 2. inject遵循就近原则，即如果上例中B也通过provide向C传入messageFromA的值，则C会优先接收B的

</div>

![](./provideAndinject.webp)
## eventBus
> eventBus又称事件总线。通过注册一个新的Vue实例，通过调用这个实例的\$emit和\$on等来监听和触发这个实例的事件，通过传入参数来实现事件的全局通信。它是一个不具备 DOM 的组件，有的仅仅只是它实例方法而已，因此非常的轻便。
我们可以通过在全局Vue实例上注册:
```js
// main.js
Vue.prototype.$Bus = new Vue()
```
但是当项目过大时，我们最好将事件总线抽象为单个文件,将其导入到需要使用的每个组件文件中。这样,它不会污染全局命名空间：
```js
//bus.js，使用时通过import引入
import Vue from 'vue'
export const Bus = new Vue()
```
**原理分析**
eventBus使用的是订阅-发布模式：

## Vuex
待续。。。
## 总结
待续。。。