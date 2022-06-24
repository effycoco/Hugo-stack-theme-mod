---
title: "Var, Let, Const 区别"
description:
date: 2022-06-24T18:36:04+08:00
image:
math:
license:
hidden: false
comments: true
draft: false
categories:
  - notes
tags:
  - JavaScript
---

# 参考资料

_Secrets of the JavaScript Ninja_ Section 5.5

# 小结

这三种定义变量的方式的区别体现在两方面：

1. 可变性
2. 与词法环境的关系

# 变量可变性

如果按是否可变分类，const 是一种，var 和 let 是另一种。

用 const 声明的变量必须得在定义时给它一个初始值，且后续不能再赋值。但如果初始值是对象类型，后续可修改增删其属性。比如对于 const 定义的 array，后续可更改其元素。即可对其已有的值进行修改，只是不能再次赋全新的值来覆盖旧值。

用 var 和 let 声明的变量可以不给它初始值，也可以在后续赋值任意次。

# 所定义变量的关键字与词法环境的关系

如果按与词法环境的关系（即其作用域 scope）分类，var 是一种，let 和 const 是另一种。
var 定义的变量以函数为界，let 和 const 定义的变量以 block 为界。
对于在 block 里定义的变量，二者有关键区别，前者定义的变量注册和储存在最近的函数中，后者定义的变量注册和储存在最近的 block 中。

## 使用 `var` 定义 function-scoped 变量

使用 `var`，变量定义在最近的函数或全局词法环境里面，而 block（如`loops`, `try-catch`, `switch`等）被忽略。 也就是说变量的作用域仅由函数决定，block 不起作用。只要是在同一函数里，在 block 外面可以访问到 block 里面用`var`定义的变量。
Variables declared with the keyword `var` are always registered in the closest function or global lexical environment, without paying any attention to blocks. 即使是在 block 里面定义的`var`变量, 一开始就是在最近的函数词法环境里注册的，而非在 block 词法环境里。

## 使用 `let` 和 `const` 定义 block-scoped 变量

使用 `let` 或 `const`，变量定义在最近的词法环境(可以是 block environment, function environment, 或 global environment)中。 所以 let 和 const 定义的变量作用域可以是 block, function 和 global。在 block 里面定义的变量作用域是该 block，在 block 外面无法访问。
