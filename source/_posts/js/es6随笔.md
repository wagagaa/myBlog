---
title: es6随笔
date: 2019-11-19 17:05:30
tags: 
  - es6
  - js
---

es6
<!-- more -->

## for of 和 Iterator

+ es6中for of是for、forEach的集合。
+ for of依赖于es6的Iterator迭代器方法。
+ 可迭代类型：string、array、map、set、arguments类数组
+ 默认调用迭代器：解构赋值、扩展用算符
+ 对象默认没有迭代器。为对象添加Symbol.iterator属性可以有。
+ 当 for of执行的时候，循环过程中引擎就会自动调用这个对象上的迭代器方法。
依次执行迭代器对象的 next 方法,将 next 返回值赋值给 for of 内的变量，从而得到具体的值。

```js
ar arr=[100,200,300];
var iteratorObj=  arr[Symbol.iterator]();//得到迭代器方法，返回迭代器对象
console.log(iteratorObj.next());
console.log(iteratorObj.next());
console.log(iteratorObj.next());
console.log(iteratorObj.next());
```

## 循环

+ 推荐在循环对象属性的时候，使用for...in,在遍历数组的时候的时候使用for...of。
+ for...in循环出的是key，for...of循环出的是value
+ 注意，for...of是ES6新引入的特性。修复了ES5引入的for...in的不足
+ for...of不能循环普通的对象，需要通过和Object.keys()搭配使用

## 检测对象有几个属性

`Object.keys(obj)`

arr=Object.keys(obj)
