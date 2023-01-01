---
title: "Data Type"
description: "第4&5期 - 数据类型"
date: 2022-08-17T13:24:35+08:00
image:
math:
license:
hidden: false
comments: true
draft: false
categories:
  - 陪你读书JavaScript高级程序设计
---

# 什么是数据类型？

计算机的本质是什么？是计算。计算的本质是什么？是对值进行操作。无论是参与计算的值还是计算得到的值，在计算机中都称为数据。计算机语言要处理大量数据，想要更好地处理数据，就要把数据进行归纳分类。把数据归纳分出来的类别就叫数据类型。

# JS 中的数据类型（7 种）

- 原始数据类型（6 种）：数字、字符串、Boolean、undefined、null、Symbol
- 复杂数据类型（1 种）：[[对象]]

另一种分类方式是按是否可变分类，原始数据类型均不可变，只有对象是可变的。比较原始数据类型时是比较值，而比较对象时，通常比较的是对象的引用，也就是指针，看它们是否指向同一位置。就对象本身而言，任何两个独立的对象均不相等。

# JavaScript 变量的动态类型

所有变量实际上都是指针，所指向的值有类型之分，而变量本身没有。

复制变量就相当于复制指针，而不是复制它们指向的数据。

# 查看数据类型 - `typeof()`

返回字符串，有 7 种，与数据类型分类相比少了 `null`，多了 `function`。

`typeof(null)` 返回 `'object'`。非函数的对象传递给 `typeof()` 也返回 `'object'`

typeof 是操作符而不是函数，使用时可以不带括号。推荐使用时将操作符与操作数整个括起来，以免产生歧义。

```js
var a = true;
var b = true;
alert(typeof a == b);
```

查询 [操作符优先级](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#table)，`typeof` 高于 `==`，因此先计算 `typeof a` 得到 `'boolean'`，再判断字符串与 `true` 是否相等，得到 `false` 。这种情况下虽然是否加括号不影响结果，但可避免阅读时产生理解偏差。

# `undefined` vs. `null`

相同点：

1. 这两种数据类型都只有一个值
2. 这两个值都是 falsy value，参与判断时都先被转换为 false。所以 `null==undefined` 返回 `true`，因为相当于 `false==false`
3. 没有方法。虽然严格来说，只有对象有方法，但 String, Number 这些数据类型也能调用方法（尽管 under the hood 是先转换为对象），只有 undefined 和 null 是无法调用方法的。

不同点：

1. `null` 是关键字，`undefined` 不是关键字
2. `null` 是一个特殊的对象，空对象。`undefined` 本质是 `Window` 对象的一个属性，属性的值为 `undefined`。
3. `undefined` 代表未初始化变量，定义变量为 `null` 时则相当于已经初始化了，初始化为空对象。
4. typeof 得到的结果不同

   ```js
   typeof undefined; // 'undefined'
   typeof null; // 'object'
   ```

5. 转换为数字返回结果不同
   ```js
   Number(undefined); // NaN
   Number(null); // 0
   ```

## 用法

1. 声明变量时可以不赋值，如果一定要赋值，可以先赋为 `null`
2. 如果非要用 `===` 检测某个变量值是否存在时，用 `undefined`
3. 如果非用 `===` 检测某个变量值是否为空时，用 `null`
   空和不存在的区别？

给初学者的建议

1. 声明变量并赋值时不用 `undefined`，用 `null`。前者没有意义，后者初始化了变量，且明确了该变量将要保存对象型数据。
2. 判断变量值是否为空或不存在时，建议用 `==null`。因为从后台返回的数据要么没有这个值，如果有但是是空的，会写 `null`。（我的理解，server 返回的数据可能有 null，不会有 undefined）[[隐式类型转换]]
3. 如果明确知道返回的是 null 还是 undefined，判断时可以用 `===`

# Boolean

逻辑运算又叫布尔运算，布尔运算得到的值为布尔类型。

其他类型数据在需要时都能转换为布尔类型，其中会转换为 `false` 的有 6 个：

- undefined
- null
- 0
- -0
- ""
- NaN

其他数据都会转换为 `true`

转换方法：

1. 数据传递给 `Boolean()`
2. 数据前面加 `!!`

对于代码中出现未声明的变量，解释器遇到都会报错，只有前面加 typeof 时不会报错，而是返回 `undefined`
