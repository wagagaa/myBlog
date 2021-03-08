---
title: js
date: 2019-10-29 22:39:46
tags: js
---

js的一些重要或易忘的记录

<!-- more -->

## instanceof

### 判断是否是数组

若为true，是数组
`arr instanceof Array`
或
`Array.isArray(arr)`

扩展：如果arr上面为true，则下面也为true。
原因：原型链。从 instanceof 能够判断出 [].proto 指向 Array.prototype，而 Array.prototype.proto 又指向了Object.prototype，最终 Object.prototype.proto 指向了 null，标志着原型链的结束。
`arr instanceof Array`

## 深拷贝

### Object.create

严格不算是深拷贝。使用现有的对象来提供新创建的对象的__proto__。通过原型链实现类似深拷贝。

`const objNew = Object.create(objOld);`

+ 附：该函数还可以实现类式继承

### Object.assign

只能进行首层深拷贝

`const objNew = Object.assign({}, objOld);`

### 递归/JSON转化

大部分可拷贝。除function等。

## enumerable:可枚举性

如果一个属性的enumerable为false，下面四个操作不会取到该属性。

+ for..in循环
+ Object.keys方法
+ JSON.stringify方法
+ Object.assign({}, objOld)方法

```js
var o = {a:1, b:2};
o.c = 3;
Object.defineProperty(o, 'd', {
  value: 4,
  enumerable: false
});
o.d
// 4
for( var key in o ) console.log( o[key] ); 
// 1
// 2
// 3
Object.keys(o)  // ["a", "b", "c"]
JSON.stringify(o // => "{a:1,b:2,c:3}"
```

+ 附：获取所有属性:Object.getOwnPropertyNames

## 其他值转化为字符串

+ 使用 String() 方法将其它对象转化为字符串可以被认为是一种更加安全的做法，虽然该方法底层使用的也是 toString() 方法，但是针对 null/undefined/symbols，String() 方法会有特殊的处理

## 其他

+ 创建对象：您可以看到，person2是基于person1创建的。缺点是比起构造函数，浏览器在更晚的时候才支持create()方法（IE9,  IE8 或甚至以前相比），您只需要一个对象的一些副本， 那么创建一个构造函数可能会让您的代码显得过度繁杂。有些人发现create() 更容易理解和使用。

`var person2 = Object.create(person1);

+ js findIndex()方法_返回满足条件的数组下标
+ splice() 方法向/从数组中添加/删除项目，然后返回被删除的项目。

## 删除事件

```js
// 错误做法
document.body.addEventListener('touchmove', function (event) {
    event.preventDefault();
},false);
document.body.removeEventListener('touchmove', function (event) {
    event.preventDefault();
},false);
```

```js
// 正确做法
// 相同事件绑定和解除，需要使用共用函数;共用函数不能带参数
function bodyScroll(event){
    event.preventDefault();
}
document.body.addEventListener('touchmove',bodyScroll,false);
document.body.removeEventListener('touchmove',bodyScroll,false)
```
