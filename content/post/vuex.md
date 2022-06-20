---
title: "Vuex"
description: Udemy课程和官方文档学习笔记
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
  - 学习笔记
---

# 关于

材料：

- 官方文档（中/英）：[https://vuex.vuejs.org/guide/](https://vuex.vuejs.org/guide/#the-simplest-store) （以下简称官网）
- udemy 课第 15 节

用法：先看视频标题，在官网中找到对应部分，读，读完或读到看不懂的部分后看视频。一个视频看完后再看下一个视频标题，读下一部分的对应文档……；这次没有一比一机械复刻视频里的代码，看懂了他这样设计是为了用到什么知识点后，我可以自己用不完全一致的方式写代码以用到这个知识点。

评价：是篇合格的笔记，一开始只是想把在官网的高亮和批注整合在一起，只是复制粘贴，后来加入了自己的理解。这是我学这个课以来最透彻的一节，记笔记的功劳。

# 小结

商店是全局可用，任何一个组件内都可访问。

getter、mutation、action 都是由一个 string type 和一个 handler 组成的

getter 是根据 state 数据计算新数据，使相同逻辑集中在一处，多个组件使用时不必重复代码

mutation 直接更改 state 数据，action 通过提交 mutation 间接更改 state 数据

mutation 必须是同步函数，action 可以包含异步操作

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

## 使用 mutation 改变 state

> Again, the reason we are committing a mutation instead of changing `store.state.count`
>  directly, is because we want to explicitly track it. This simple convention makes your intention more explicit, so that you can reason about state changes in your app better when reading the code. In addition, this gives us the opportunity to implement tools that can log every mutation, take state snapshots, or even perform time travel debugging. 在 vuex 中，更改状态通过提交 mutation 的方式，而非直接改变  `store.state.count`，这样可以更明确地追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样你在阅读代码的时候能更容易理解为什么改变状态。此外，这样也让我们能利用那些能记录每次状态改变，保存状态快照的调试工具。甚至可以实现如时间穿梭般的调试体验。

在组件里通过方法直接更改 state 是可以做到的，但不符合 the Vuex philosophy, 不推荐。通过 mutation 更改的好处：不易出错；可追踪状态的变化；可读性更强；方便实现调试工具。另外，将更改状态的代码集中在一处，而不是分散在各组件的方法中，有利于后续维护。

## 在组件中使用 state 和 mutation

> Using store state in a component simply involves returning the state within a computed property, because the store state is reactive. Triggering changes simply means committing mutations in component methods. 由于 store 中的状态是响应式的，在组件中调用 store 中的状态简单到仅需要在计算属性中返回即可。触发变化也仅仅是在组件的 methods 中提交 mutation。

```jsx
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

# State

## Access

in main.js, 在传递给 createStore 的对象内部：`state.name` （在 action handler 中是`context.state.name`）

in main.js, 在 store 创建后，`store.state.name`

Inside script of component file: `this.$store.state.name`

Inside template of component file: `$store.state.name` (不推荐，按照 convention，应该在计算属性中返回)

## The mapState Helper

当一个组件需要用到 store 的多个 state 和 getter 时，挨个声明为计算属性有些繁琐，使用 mapState 可简化步骤。

```jsx
// 在Component.vue中
computed: mapState([
  // map this.count to store.state.count 不改名
  "count",
]);

// or
computed: mapState({
  // map this.number to store.state.count 改名
  number: "count",
});
```

```jsx
// 代替
computed: {
	count () {
      return this.$store.state.count
  }
}
```

mapState 返回的是对象，如果还有其他要在组件中声明的计算属性，要用\***\*Object Spread Operator\*\***将 mapState 返回的对象和包括本地计算属性的对象合为一个(前者的 key-value pairs spread 在后者里面)：

```jsx
computed: {
  localComputed () { /* ... */ },
  // mix this into the outer object with the object spread operator
  ...mapState({
    // ...
  })
}
```

## Components Can Still Have Local State

依然可以在组件中用`data(){return {}}`单独声明本地 state，在适合的情况下，比如 a piece of state strictly belongs to a single component.

# Getters

使用场景：need to compute derived state based on store state

可以看作是 computed properties for stores.

如果没有 getters 呢，假设一个在 state 基础上的计算逻辑需要在多个组件用到，就需要复制函数，或者提取成一个共享函数然后在多处导入它，总之会很麻烦

## Property-Style Access

- In main.js, 在传递给 createStore 的对象内部：`getters.name`
  - 如果定义 getter 时需要用其他 getter，则将`getters`作为第二个参数（`state`是 getter 的第一个参数）
  - 在 action handler 内部是`context.getters.<name>`
- In main.js, 在 store 创建后，`store.getters.name`
- Inside script of component file: `this.$store.getters.name`
- Inside template of component file: `$store.getters.name` (和 state 一样，最好是通过在计算属性中返回来访问，而不是在 template 里直接访问)

## `mapGetters`

用法与`mapState`类似

## Method-Style Access

pass arguments to getters by returning a function

```jsx
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find((todo) => todo.id === id);
  };
}
```

```jsx
store.getters.getTodoById(2); // -> { id: 2, text: '...', done: false }
```

```jsx
// 完整写法是这样的
getters: {
  // ...
  getTodoById：function(state) {
		return  (id) => {
	    return state.todos.find(todo => todo.id === id)
	  }
	}
}
// 或
getters: {
  // ...
  getTodoById(state) {
		return  (id) => {
	    return state.todos.find(todo => todo.id === id)
	  }
	}
}

```

通过 vuex 内部语法，让传递给 getter 的 argument 传递给 getter 所返回的函数，而 getter 自身接受的 argument 只有 state?

## The `mapGetters` Helper

用法与`mapState`类似

# Mutations

## 注册 mutation

永远不要在组件内部直接更改 state，而是提交 mutation，调用 mutation 的 handler

```jsx
const store = createStore({
  state: {
    count: 1,
  },
  mutations: {
    increment(state) {
      state.count++;
    },
  },
});
```

## 在组件中使用 mutation

```jsx
methods:{
	add(n){
		this.$store.commit('increment',n) // payload format commit
	}
}
// or
methods:{
	add(n){
		this.$store.commit({type:'increment', n:n}) // object-style commit
	}
}
```

### `mapMutations`

```jsx
methods:{
	...mapMutation({
		add: 'increment'
	})
}
// 需要传递的载荷应在template中与事件绑定时传入 @click="add({n:10})"
```

不懂的地方：文档里在提到组件中用 mapMutations 或 mapActions 时都注明“需要在根节点注入  `store`”（requires root `store` injection），这是什么意思

## Mutations must be synchronous

原因，否则 mutation 对状态的改变就是不可追踪的了。假设两个 mutation 都对一个状态进行更改，其中一个 mutation 是异步函数，如过先调用那个异步的，再调用另一个，无法确定另一个被调用时 state 是否已经过更改。有可能改了，也可能没改，这样无法保证 mutation 被调用时获取的是最新的 state，会出错。

如果你非要在 mutation 里写异步函数，也会 work，但是 bad practice，正确做法是写在 actions 里。

# Actions

类似 mutations, 区别在于：

- actions 是提交 mutation 而不是直接更改状态
- actions 可以含有异步操作

最好是在组件里一律用`this.$store.dispatch('xxx')`间接提交 mutation（put actions in between components and mutations 即在组件里只触发 action, 由 action 提交 mutation），即使不涉及异步操作。（udemy 课这么说的，官网没要求）

触发 mutation 是`store.commit`, 触发 action 是`store.dispatch` ; 向二者传递参数的方式是一样的，payload or object-style，详见文档。

# Modules

可以由**模块（module）**组成 store。每个模块拥有自己的 state、mutation、action、getter.

可以由模块和非模块组成， merge modules into main store

```jsx
// ... 定义moduleA, moduleB

const store = createStore({
  modules: {
    a: moduleA,
    b: moduleB,
  },
  state() {
    return {};
  },
  mutations: {},
  actions: {},
});
```

在各模块内部，传递给 mutations 和 getters 的第一个参数`state`是 local state。传递给 actions 的第一个参数是`context`，`context.state`是 local state，`context.rootState`是 root state。传递给 getters 的参数有三个`(state, getters, rootState)`。

总之，模块内的`state`是 local state。

## NameSpacing 命名空间

默认 getters, actions, mutations 都是注册在全局命名空间的。所以不能在不同模块中定义重名的 getters, actions, mutations。

若在定义 module 时，使用了`namespaced: true` 则该模块具有独立的命名空间，该模块内的 getters, actions, mutations 名可以与其他模块或主 store 中定义的 getters, actions, mutations 名相同。

对于 state，模块内的 state 始终是独立的，不管是默认还是添加了`namespaced: true`。访问模块内的 state 要用`this.$store.state.<模块名>.<state名>`

在组件中访问 namespaced 模块时，要加上路径，比如原来的`this.$store.getters.counter` 换成`this.$store.getters['numbers/counter']` , `numbers`是指创建 store 时分配给该 namespaces module 的 key name , 而不是定义模块时用的变量名

```jsx
const counterModule = {
  namespaced: true,
  // ...
};
const store = createStore({
  modules: {
    numbers: counterModule,
  },
});
```

或者可以说`numbers`是`counterModule`的 namespace 名称（udemy 课这么说的，官网没说）。

如果是用 mapper helpers，比如 mapGetters， 则把 `...mapGetters(['finalCounter'])` 换成`...mapGetters(numbers,['finalCounter'])` 。mapActions 同理。如果 mapHelper 接受的参数是的对象而不是数组也同理，只要添加 module 名作为第一个参数。

## 改用模块后的变动

如果一开始未用模块，整个 app 写完后要把一部分逻辑提取出来放到一个模块里，在组件文件中，用 getters, mutations, actions 的地方无需改变；用到被提出的 state 之处，要将`this.$store.state.<state名>`替换为`this.$store.state.<模块名>.<state名>`。

如果在定义模块时添加了`namespaced: true`，则用 getters, mutations, actions 的地方也需改变：

- mutation: `this.$store.commit('<mutation名>')` 替换为 `this.$store.commit('<模块名>/<mutation名>')` (mutation 名即 mutation type)
- mapMutations: `...mapMutations(['<mutation名>'])` 换成`...mapMutations(<模块名>,['<mutation名'])`
- 其他两个同理

# Application Structure

## Restructuring

如果一开始把全部逻辑都写在了 main.js 里，现在要 restructrue，首先在 src 下面新建一个 store 文件夹，其中的 index.js 文件使用 createStore 并导出创建的 store，由 main.js 导入。若要将 store 进一步细分，比如将 module 提取出来，在 store 文件夹下创建 modules 文件夹，创建 module 同名的 js 文件，在该文件中创建 module 对象并导出。创建 store 时用的 actions, getters, mutations 也都可以存放在单独的文件里。

# 学英语时间

whether to … is largely a preference 做不做……取决于你/看个人喜好

it's totally optional if you don't like them 如果你不喜欢，你完全可以不这样做。

as our application grows in scale, the store can get really **bloated**.

- **bloated** adj. excessive in size or amount
