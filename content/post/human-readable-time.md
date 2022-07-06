---
title: "Human Readable Time"
description:
date: 2022-07-07T00:19:10+08:00
image:
math:
license:
hidden: false
comments: true
draft: false
tags:
  - codewar
---

题目要求：将一个秒数转换成 HH:MM:SS

---

如果是 24 小时制的时间，最大 23:59:59，有一个轻松的解决方案

```jsx
function toReadableTime(seconds) {
  return new Date(seconds * 1000).toISOString().substring(11, 19);
}

toReadableTime(86399); // => '23:59:59'
toReadableTime(86400); // => '00:00:00'
new Date(86399 * 1000).toISOString(); // => '1970-01-01T23:59:59.000Z'
new Date(86399 * 1000); // => Fri Jan 02 1970 07:59:59 GMT+0800 (China Standard Time)
```

`substring(indexStart, indexEnd)` 返回一个 String 的 substring，包括起点不包括终点。

- String 的 index 从零开始数。
- 如果只有一个传参，则该参数代表 indexStart，截取从该处至末尾的 substring。

`new Date(milliseconds)` 返回一个日期对象（以本地时区表示）。

- milliseconds 代表 UNIX 纪元 1970 年 1 月 1 日午夜（UTC）之后的毫秒数.
- 返回的日期对象实际上是根据毫秒计算的 UTC 时间，然后转换成本地时间。

`toISOString()` 将一个日期对象转换成 ISO 格式的字符串，时区总是 UTC。

---

如果是时长，最大 99:59:59，以上方法不适用。

```jsx
function toDuration(seconds) {
  let hours, minutes;
  hours = Math.floor(seconds / 3600);
  seconds %= 3600;
  minutes = Math.floor(seconds / 60);
  seconds = seconds % 60;

  // 转换成string，单位数前面加0，双位数不变
  function toDoubleDigits(number) {
    return number < 10 ? `0${number.toString()}` : number.toString();
  }

  return `${toDoubleDigits(hours)}:${toDoubleDigits(minutes)}:${toDoubleDigits(
    seconds
  )}`;
}

toDuration(86400); // => '24:00:00'
toDuration(359999); // => '99:59:59'
```
