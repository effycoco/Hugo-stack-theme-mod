---
title: "Valid Braces"
description:
date: 2022-07-08T00:18:33+08:00
image:
math:
license:
hidden: false
comments: true
draft: false
tags:
  - codewar
  - leetcode
---

题目要求：给一个由括号构成的字符串，判断括号是否有效。

```jsx
function validBraces(braces) {
  //TODO
  const pairs = {
    "[": "]",
    "{": "}",
    "(": ")",
  };
  const stack = [];
  for (let cha of braces) {
    if (pairs[cha]) {
      // 字符是开，将其对应关放至stack尾端
      stack.push(pairs[cha]);
    } else if (stack.pop() !== cha) {
      // 字符是关，且跟stack尾端不一致
      return false;
      // 字符是关，且跟尾端一致，说明跟前面构成一对，从stack尾端去掉表示忽略这对继续看外层
    }
  }
  // 全部循环完未提前结束说明没有多余的关
  // 若有多余的开被剩下，说明invalid
  return !stack.length;
}
```

这道题 10 个月前在 leetcode 做过，这次再遇到还是没能独立想出来，怀疑我 leetcode 的提交是直接抄的讨论区答案，然后自己只是看懂了答案。这次也是直接看了自己提交过的答案，然后又默写出来。

思路：将每对括号作为 key-value pair 储存在对象里，然后对字符串的字符挨个循环，检查当前字符是开括号还是关括号。循环前建一个空数组准备储存待寻找的关。如果是开，则把对应的关储存在数组里，等后面遇到要找的关就把它从数组里移除（消消乐消掉）。如果是关，则检查数组尾端是不是这个关，因为`[(])`是无效的，有效的`()[]{}`和`([{}])` 在循环时遇到关括号数组尾端一定也是这个关括号。所以如果循环到关括号，若与数组尾端的关括号一致，则说明和前面的某个开括号可配对，应该从数组中去掉尾端，然后进行下一个检查。如果关括号与数组尾端不一致，直接返回`false`。

如果字符串全部循环完，没有提前结束，说明循环时每遇到一个关括号都能从前面找到其开括号，并从 stack 数组中删掉之前循环到这个开括号时所`push`的关括号。如果循环完数组不为空，则说明字符串中有多余的开括号，例如`(((({{` 。

遇到开括号向 stack 数组尾端添加元素，遇到关括号从尾端减少元素。

上面的代码可视作略微 refactor 后的结果，完整直白的写法是：

```jsx
function validBraces(braces) {
  //TODO
  const pairs = {
    "[": "]",
    "{": "}",
    "(": ")",
  };
  const stack = [];
  for (let cha of braces) {
    if (pairs[cha]) {
      stack.push(pairs[cha]);
    } else if (stack[stack.length - 1] !== cha) {
      return false;
    } else {
      stack.pop();
    }
  }
  if (stack.length === 0) {
    return true;
  } else {
    return false;
  }
}
```
