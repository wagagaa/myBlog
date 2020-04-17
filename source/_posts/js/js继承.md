---
title: 继承与原型链
date: 2020-03-11 18:05:22
tags:
   - js
   - 继承
---

js继承、原型链
<!-- more -->

## 继承属性

+ 继承属性:

```js
对象继承：someObject.[[Prototype]] 可以通过 Object.getPrototypeOf() 和 Object.setPrototypeOf() 访问器来访问;
函数继承：f.prototype
```

+ 继承方法:

```js
// 不要在 f 函数的原型上直接定义。这样会直接打破原型链。

// 当继承的函数被调用时，this 指向的是当前继承的对象，而不是继承的函数所在的原型对象。

// 函数（function）是允许拥有属性的。所有的函数会有一个特别的属性 —— prototype
doSomething.prototype.foo = "bar"; // doSomething函数的原型对象添加新属性,实例化的对象原型属性对应更新。
```

## 使用不同的方法来创建对象和生成原型链

### 使用语法结构创建的对象

```js
var o = {a: 1};

// o 这个对象继承了 Object.prototype 上面的所有属性
// o 自身没有名为 hasOwnProperty 的属性
// hasOwnProperty 是 Object.prototype 的属性
// 因此 o 继承了 Object.prototype 的 hasOwnProperty
// Object.prototype 的原型为 null
// 原型链如下:
// o ---> Object.prototype ---> null

var a = ["yo", "whadup", "?"];

// 数组都继承于 Array.prototype 
// (Array.prototype 中包含 indexOf, forEach 等方法)
// 原型链如下:
// a ---> Array.prototype ---> Object.prototype ---> null

function f(){
  return 2;
}

// 函数都继承于 Function.prototype
// (Function.prototype 中包含 call, bind等方法)
// 原型链如下:
// f ---> Function.prototype ---> Object.prototype ---> null
```

### 使用构造器创建的对象

```js
// 在 JavaScript 中，构造器其实就是一个普通的函数。当使用 new 操作符 来作用这个函数时，它就可以被称为构造方法（构造函数）。

function Graph() {
  this.vertices = [];
  this.edges = [];
}

Graph.prototype = {
  addVertex: function(v){
    this.vertices.push(v);
  }
};

var g = new Graph();
// g 是生成的对象，他的自身属性有 'vertices' 和 'edges'。
// 在 g 被实例化时，g.[[Prototype]] 指向了 Graph.prototype

```

### 使用 Object.create 创建的对象

```js
var a = {a: 1};
// a ---> Object.prototype ---> null

var b = Object.create(a);
// b ---> a ---> Object.prototype ---> null
console.log(b.a); // 1 (继承而来)

var c = Object.create(b);
// c ---> b ---> a ---> Object.prototype ---> null

var d = Object.create(null);
// d ---> null
console.log(d.hasOwnProperty); // undefined, 因为d没有继承Object.prototype

```

### 使用 class 关键字创建的对象

```js
// 这些新的关键字包括 class, constructor，static，extends 和 super。

"use strict";

class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}

class Square extends Polygon {
  constructor(sideLength) {
    super(sideLength, sideLength);
  }
  get area() {
    return this.height * this.width;
  }
  set sideLength(newLength) {
    this.height = newLength;
    this.width = newLength;
  }
}

var square = new Square(2);
```

+ hasOwnProperty 是 JavaScript 中唯一一个处理属性并且不会遍历原型链的方法。（其实还有另一种这样的方法：Object.keys()）
+ 经常使用的一个错误实践是扩展 Object.prototype 或其他内置原型。

## 4 个用于拓展原型链的方法

### 函数原型的设置

`函数.prototype=对象`

### 对象原型的设置

1. 对象=new 函数
2. 对象=Object.create(函数.prototype)
3. Object.setPrototypeOf(对象, 函数.prototype);
4. 对象.\_\_proto__=foo.prototype

+ 上面->foo.prototype可换为对象

### 优缺点

| 名称                  | 优点                                                                 | 缺点                                                                                                       |
|-----------------------|----------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|
| new 函数              | 所有浏览器适用。这种方法非常快，非常符合标准，并且充分利用JIST优化。 | 函数必须要被初始化。构造函数的初始化，可能会给生成对象带来并不想要的方法                                   |
| Object.create         | IE9+                                                                 | 第二个参数的时候有可能成为一个性能黑洞                                                                     |
| Object.setPrototypeOf | IE9+。允许动态操作对象的原型。                                       | 应该被弃用。如果你在生产环境中使用这个方法，那么快速运行 Javascript 就是不可能的，因为许多浏览器优化了原型 |
| 对象.\_\_proto__      | IE11+。将 \_\_proto__ 设置为非对象的值会静默失败，并不会抛出错误。   | 应该完全将其抛弃。因为这个行为完全不具备性能可言                                                           |

+ 其他

```js
var o = new Foo();
// 等于
var o = new Object();
o.__proto__ = Foo.prototype;
Foo.call(o);
```
