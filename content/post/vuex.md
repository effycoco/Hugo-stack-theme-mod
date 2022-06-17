---
title: "Vuex"
description:
date: 2022-06-17T23:14:47+08:00
image:
math:
license:
hidden: false
comments: true
draft: false
categories:
  - Notes
tags:
  - 前端
---

#

[https://vuex.vuejs.org/guide/](https://vuex.vuejs.org/guide/#the-simplest-store)

商店是全局可用，任何一个组件内都可访问。

# Get Started

在`main.js`里引入 vuex，并创建 store

```jsx
import { createApp } from "vue";
import { createStore } from "vuex";

// 创建一个新的 store 实例
const store = createStore({
  state() {
    return {
      count: 0,
    };
  },
  mutations: {
    increment(state) {
      state.count++;
    },
  },
});

const app = createApp({
  /* 根组件 */
});

// 将 store 实例作为插件安装
app.use(store);
```

> Again, the reason we are committing a mutation instead of changing `store.state.count`
>  directly, is because we want to explicitly track it. This simple convention makes your intention more explicit, so that you can reason about state changes in your app better when reading the code. In addition, this gives us the opportunity to implement tools that can log every mutation, take state snapshots, or even perform time travel debugging. 再次强调，我们通过提交 mutation 的方式，而非直接改变  `store.state.count`，是因为我们想要更明确地追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样你在阅读代码的时候能更容易地解读应用内部的状态改变。此外，这样也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。有了它，我们甚至可以实现如时间穿梭般的调试体验。

在组件里通过方法直接更改 state 是可以做到的，但不符合 the Vuex philosophy, 不推荐。通过 mutation 更改的好处：不易出错；可追踪状态的变化；可读性更强；方便实现调试工具。另外，将更改状态的代码集中在一处，而不是分散在各组件的方法中，有利于后续维护。

> Using store state in a component simply involves returning the state within a computed property, because the store state is reactive. Triggering changes simply means committing mutations in component methods. 由于 store 中的状态是响应式的，在组件中调用 store 中的状态简单到仅需要在计算属性中返回即可。触发变化也仅仅是在组件的 methods 中提交 mutation。

```jsx
// in Component.vue
// returning the state within a computed property
computed:{
	counter(){
		return this.$store.state.counter
	}
},

// commiting mutations in component methods
methods:{
	addOne() {
	  this.$store.commit('increment');
  },
}

```

# Mutation
