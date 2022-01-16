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
对Vue组件通信方式进行的一个总结。
<!-- more -->
1. props/$emit
**简介**
**props:** 可以是数组或者对象，用于接收父组件传递而来的属性；当props为对象时，可以通过type、default、required、validator等配置来设置属性的类型、默认值、是否必传和校验规则。
**$emit:** 用于触发父组件在子组件上绑定了v-on事件相应的监听。
**实例**
* 父向子传值：父组件通过`:messageFromParent="message"`将父组件message值传递给子组件，当父组件的input标签输入时，子组件p标签的内容就会改变。
* 子向父传值：父组件通过`@on-receive="receive"`在子组件上绑定了receive事件的监听，子组件input标签输入时，会触发receive回调函数，通过`this.$emit('on-receive',this.message)`将子组件message的值赋给父组件messageFromChild，改变父组件p标签的内容。

