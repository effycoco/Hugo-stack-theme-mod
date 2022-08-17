---
title: "Object"
description: "第6期 - JavaScript中的对象"
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

# 对象与原始类型的不同点

1.  有属性。对象是由属性构成的无序的合集（对象属性没有顺序），原始数据类型没有属性。
    对象属性的值可以是对象，也可以是原始数据类型。
2.  有方法。方法是一种特殊的属性，其值是函数。函数/方法是对象的一种。只有对象有方法，原始数据类型的方法比如 String 的方法其实是先将它们转换为对象再调用方法。
3.  可改变。对象是可改变的，原始数据类型是不可改变的。

# 对象的种类

1.  内部对象。JS 语言自带的对象，17 个。 内部对象又可分为：

- 错误对象
- 常用对象（8 种：布尔、字符串、数字、数组、函数、对象、Date、正则表达式）
- 内置对象（Math, JSON, global）

2.  宿主对象。JS 语言的运行环境所产生的对象。背景：ECMA 给 JS 制定标准时有一条原则是对象和平台无关。常见的数组对象有 Windows 和 Document。
3.  自定义对象。

# 原始数据类型与对象互相转换

基础数据类型都有一个将任何数据类型转换为该类型的方法，比如 String, Number。对象也有 `Object()` 可以将任何数据类型转换为对象。

## 原始数据类型转换为对象

任何数据类型都可以作为参数传递给 `Object()` 并被转换为对象。

### 布尔值&数字

![](https://s2.loli.net/2022/08/15/kRueOsvidc1ImCW.png)

访问对象的 PrimitiveValue 属性—— 调用它的 `.valueOf()` 方法

访问对象的 Protoype —— `Object.getPrototypeOf()`

### 字符串

除了原始值属性外还有 length 属性及 0,1,2,...n-1 属性
![](https://s2.loli.net/2022/08/15/yFbZABChnVcDatg.png)

```js
const x=Object('abc');
x.valueOf(); // 'abc'
Object.getPrototypeOf(x); // String
typeof(x) // 'object'
x[1] // 'b'
x.1 // error
```

![](https://s2.loli.net/2022/08/15/3WiZRaOwtqj9rFK.png)

### `undefined` & `null`

得到空对象
![](https://s2.loli.net/2022/08/15/bJA8uVNBlw6chLC.png)

## 对象转换为原始数据类型

### 转换为布尔值 - `Boolean()`

无论传入（广义上的）任何对象，都会得到 `true`。
面试坑：

```js
Boolean(Object(false)); // => true
Boolean(new Boolean(false)); // => true
Boolean(new Boolean(false)); // => false
```

`new Boolean(false)` 创建的是一个布尔类型的对象，其 `Prototype` 为 Boolean，但 `typeof()` 为对象; `Boolean(false)` 创建的是一个真正的 `false` 原始类型

![](https://s2.loli.net/2022/08/15/jWDNUyVf16kxpwR.png)

### `.toString()`

对象的 `toString` 方法将当前对象转换为字符串，八种常用对象都有自己的 `toString` 方法，所得结果不同。开发中若想将对象转换为字符串，通常调用 `toString` 方法。

1.  `array.toString()` 将数组转换为字符串，元素之间用逗号隔开，字符串元素原来的引号会去掉，只有外部有引号

    ```js
    ["a", "b"]
      .toString() // 'a,b'
      [(1, 2)].toString(); // '1,2'
    ```

2.  `函数.toString()` 返回函数源代码
3.  `日期.toString()` 得到字符串形式的日期和时间
4.  `正则表达式.toString()` 将正则表达式用字符串表示

### `.valueOf()`

如果当前对象有原始值，则返回原始值；若没有原始值则返回对象本身。例外是 Date 对象，将得到一串数字，代表自 1970 年 1 月 1 日 0 点的毫秒数

### 转换为字符串或数字

若 JavaScript 期待的是字符串，但给的是对象，那 JavaScript 会先调用 `.toString` 方法，若该方法不能得到字符串，会再调用 `.valueOf` 方法。
若 JavaScript 需要将对象转换为数字，则顺序相反，先调用 `.valueOf` 方法，再调用 `.toString` 方法

# 创建对象

有三种方法

1.  对象字面量。当属性名含空格或保留字时应加上引号。
2.  `new Object()` 若没有参数需要传递，可省略括号，但不推荐。
3.  `Object.create()` ES5 新增方法。接收两个参数，第一个是要继承的原型，第二个是对象的描述信息。

# 点表示与方括号表示

对象的属性名是字符串，点表示时字符串不带引号，方括号表示时带引号。

点语法是首选，但有些情况只能使用方括号。

- 当对象的属性名包含可能导致语法错误的字符或包含保留字关键字时必须使用方括号，例如属性名是中间有空格的多词时必须使用方括号。
- 当我们用一个变量来使属性名是动态可变的时，必须使用方括号表示。

## JS 解释器访问对象属性的步骤

JS 解释器碰到 `.` 或 `[` 时先计算前面的表达式是否是 `undefined` & `null`，如果是，直接报错；
如果否，则判断前面的部分是否是对象，如果是，进行下一步；如果否，则先将其转换为对象。这就是为什么原始数据类型没有属性，却也能调用一些方法，因为 JS 解释器在这一步将其转换成了对象。
接下来，JS 解释器判断是 `.` 还是 `[`，如果是 `.` 则直接返回 `.` 后面字符串对应的属性值，如果是 `[`，则先计算中括号内的表达式，并将计算结果转换为字符串，然后返回字符串所对应的属性值。比如数组 `arr[1]` 其实是将括号内数字先转换为字符串。
如果属性不存在，或属性值不存在，则返回 `undefined`。

面试坑：

```js
var a = "abcd";
a.len = 4;
alert(a.len); // 输出结果是？
```

`a.len` 是 `undefined` 而不是赋值的 4。
它考察的是只有对象有属性，字符串没有。当解释器遇到 `.` 前面是字符串而不是对象时会将其先转换为对象（调用 `Object(a)`），然后给该对象的 `len` 属性赋值。该语句结束后临时的对象就被清掉了，`a` 仍然是字符串，字符串是没有属性的。同理，运行 `a.toUpperCase()` 也是先将 `a` 转换为对象，并返回运行结果，结束后若没有赋值不会保存运行结果，`a` 仍然是小写。

那为什么使用 `.length` 代替 `.len` 得到的是不同结果，访问 `.length` 时一样先转换为对象，区别在于转换后的对象自带 `length` 属性。而例子中执行赋值语句时先返回一个对象赋值后又清理掉该对象，访问时产生的新对象的 `.len` 属性还是 `undefined`。
