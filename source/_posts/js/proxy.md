---
title: Proxy内置对象
date: 2020-03-18 16:38:07
---

作为vue3的数据更新方法，值得去看一下。

<!-- more -->

## 基本定义

+ Proxy 对象用于定义基本操作的自定义行为（如属性查找，赋值，枚举，函数调用等）。

## 语法

```js
let p = new Proxy(target, handler);
// target:用Proxy包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。
// handler:一个对象，其属性是当执行一个操作时定义代理的行为的函数。
```

## vue中的使用

+ vue2 基于的Object.defineProperty, 我们需要使用Vue.set,Vue.delete 来设值属性值，触发属性值的变化。
+ vue3 使用JS Proxy, 直接对属性进行赋值，就可以触发属性值变化。并且vue3的这一改进，可以更快触发属性值变化，性能比vue2 大约快2倍。
+ Proxy，可以检测到数组内部数据的变化。defineProperty不行。由于 js 的限制，Vue 不能检测一些数组的变动。需要使用Vue.set、vm.$forceUpdate()等去更新视图
+ 是

## 示例

### 基础示例

```js
  let handler = {
      get: function(target, name){
          return name in target ? target[name] : 37;
      }
  };

  let p = new Proxy({}, handler);

  p.a = 1;
  p.b = undefined;

  console.log(p.a, p.b);    // 1, undefined

  console.log('c' in p, p.c);    // false, 37
```

### 无操作转发代理

```js
  let target = {};
  let p = new Proxy(target, {});

  p.a = 37;   // 操作转发到目标

  console.log(target.a);    // 37. 操作已经被正确地转发
```

### 验证

```js
let validator = {
  set: function(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }

    // The default behavior to store the value
    obj[prop] = value;

    // 表示成功
    return true;
  }
};

let person = new Proxy({}, validator);

person.age = 100;

console.log(person.age);
// 100

person.age = 'young';
// 抛出异常: Uncaught TypeError: The age is not an integer

person.age = 300;
// 抛出异常: Uncaught RangeError: The age seems invalid
```
