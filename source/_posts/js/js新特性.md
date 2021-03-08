---
title: js新特性
date: 2020-12-21 10:33:59
tag: js
---

js新特性

<!-- more -->

## 空位合并操作符

?? 。我习惯叫他为空值默认赋值符。之前习惯用 '||' 来替换

```js
12 ?? 22  // 12
null ?? 22 // 22
undefined ?? 22 // 22

'' ?? 22 // ''
0 ?? 22 // 0 // ''和0这两个假值要注意。和之前||默认赋值有区别

'' || 22 // 22
0 || 22 // 22

null || undefined ?? "foo"; // 抛出 SyntaxError。将 ?? 直接与 AND（&&）和 OR（||）操作符组合使用是不可取的。应当是因为空值合并操作符和其他逻辑操作符之间的运算优先级/运算顺序是未定义的
(null || undefined ) ?? "foo"; // 返回 "foo"
```

+ es2020

## 可选链式操作符

?. 这个我叫他空属性访问兼容符（undefined.method // Uncaught TypeError）

```js
let foo = { someFooProp: "hi" };

console.log(foo.someFooProp?.toUpperCase()); // "HI"
console.log(foo.someBarProp?.toUpperCase()); // undefined
```

+ es2020

## Promise.allSettled

如果要执行更多的异步请求，可以使用 Promise.all 来收集它们。但是如果其中任何一个请求失败的话，将会引发异常。如果我们希望每个请求都能够完成，无论其请求是否失败，那该怎么办。这时可以用 Promise.allSettled，当所有请求都被解决或拒绝时，它将返回。

```js
const promises = [Promise.resolve(1), Promise.reject(2)];
const [result1, result2] = await Promise.allSettled(promises);
```

## BigInt

BigInt 是 JavaScript 中新的原始数据类型，与 Boolean、Number、String、Symbol和 undefined 的地位相同。 BigInt 可以安全的处理大于 Number 限制的整数数字。也就是说如果要处理大于 9007199254740991 的数字时应该用 BigInt。 BigInt 在数字末尾用 n 表示。

```js
9_007_199_254_740_991 + 2; // 9007199254740992
BigInt(9_007_199_254_740_991) + BigInt(2) // 9007199254740993n
```

## 动态导入

以前只能在文件开头静态导入模块。现在有了动态导入，可以按需在代码中的任何位置进行这种操作。import() 会与模块一起返回一个 Promise。

```js
import ( './calculate.js' )
  .then( module => {
    // use a method exported by the module
  })
  .catch( err => {
    // handle any errors loading the module or any subsequent errors
  })
```

在ES2020之前，在JavaScript中使用import语句意味着在请求父文件时，导入的文件会自动包含在父文件中。

诸如webpack的捆绑软件已经普及了“代码拆分”的概念，该功能使您可以将JavaScript捆绑软件拆分为多个文件，然后按需加载。 React还通过其React.lazy()方法实现了此功能。

代码拆分对于单页应用程序（SPA）极为有用。 您可以将代码分为每个页面单独的捆绑包，因此仅下载当前视图所需的代码。 这显着加快了初始页面加载时间，从而使最终用户不必预先下载整个应用程序。

```js
// old
import { exportPdf } from './pdf-download.js'
 
const exportPdfButton = document .querySelector( '.exportPdfButton' )
exportPdfButton.addEventListener( 'click' , exportPdf)
 
// this code is short, but the 'pdf-download.js' module is loaded on page load rather than when the button is clicked


// es2020
const exportPdfButton = document .querySelector( '.exportPdfButton' )
 
exportPdfButton.addEventListener( 'click' , () => {
  import ( './pdf-download.js' )
    .then( module => {
      // call some exported method in the module
      module .exportPdf()
    })
    .catch( err => {
      // handle the error if the module fails to load
    })
})
 
// the 'pdf-download.js' module is only imported once the user click the "Export PDF" button
```

## 导出模块的命名空间

我们很早就能在导入时重新命名

```js
import * as utils from './utils.js'
```

现在我们可以这样进行导出了

```js
export * as utils from './utils.js'

===================================
上面的结果等同于现在的用法

import * as utils from './utils.js'
export { utils }
```

## globalThis

如果你的代码需要在多个环境（例如浏览器和 Node 服务器）下运行，那么它们所使用全局对象名称并不一致。在浏览器中用的是 window，Node 则用的是 global，而 web worker 用的是self。现在，无论代码在哪种环境中运行，globalThis 都能够为你提供正确的全局对象。

下面是一个例子，我们需要检查是否可以向用户提示 alert。如果代码在浏览器中运行，则 globalThis 将会引用 window，并且 alert 也可以使用。

```js
if (typeof globalThis.alert === 'function'){
  globalThis.alert('hi');
}
```
