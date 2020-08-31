---
title: module的使用
date: 2020-07-14 17:10:48
tags:
   - js
---

引入模块我看到用 require的方式，再联想到咱们的ES6各种export 、export default。

阿西吧，头都大了....

<!-- more -->

## 使用范围

先理理他们的使用范围

`require: node 支持的引入`

`export / import : 只有es6 支持的导出引入`

`module.exports / exports: 只有 node 支持的导出`

## node模块

Node里面的模块系统遵循的是CommonJS规范。 那问题又来了，什么是CommonJS规范呢？ 由于js以前比较混乱，各写各的代码，没有一个模块的概念，而这个规范出来其实就是对模块的一个定义。

`CommonJS定义的模块分为: 模块标识(module)、模块定义(exports) 、模块引用(require)`

先解释 exports 和 module.exports 在一个node执行一个文件时，会给这个文件内生成一个 exports和module对象， 而module又有一个exports属性。他们之间的关系如下图，都指向一块{}内存区域。

```js
exports = module.exports = {};
```

```js
//utils.js
let a = 100;

console.log(module.exports); //能打印出结果为：{}
console.log(exports); //能打印出结果为：{}

exports.a = 200; //这里辛苦劳作帮 module.exports 的内容给改成 {a : 200}

exports = '指向其他内存区'; //这里把exports的指向指走

//test.js

var a = require('/utils');
console.log(a) // 打印为 {a : 200}  
```

exports 只是 module.exports的引用，辅助后者添加内容用的。

## ES中的模块导出导入

说实话，在es中的模块，就非常清晰了。不过也有一些细节的东西需要搞清楚。
比如 export 和 export default，还有 导入的时候，import a from ..,import {a} from ..，总之也有点乱，那么下面我们就开始把它们捋清楚吧。

### export 和 export default

首先我们讲这两个导出，下面我们讲讲它们的区别

1. export与export default均可用于导出常量、函数、文件、模块等
2. 在一个文件或模块中，export、import可以有多个，export default仅有一个
3. 通过export方式导出，在导入时要加{ }，export default则不需要
4. export能直接导出变量表达式，export default不行。

```js
'use strict'
// 导出变量
export const a = '100';

// 导出方法
export const dogSay = function(){
    console.log('wang wang');
}

// 导出方法第二种
function catSay(){
   console.log('miao miao');
}
export { catSay };

// export 导出
import {userName, sayHi} from './test.js';

/* ------------ */

// export default 导出
const m = 100;
export default m;
// default导入模块
import foo from './test.js';
// export defult const m = 100; // 这里不能写这种格式。
```

## ES6 import 和 export 在浏览器当中的使用

### 1.显示声明type="module"

script 里面要加 type="module", 这样浏览器才会把相关的代码当作ES6的module 来对待

```html
  <script type="module">
      import {addTextToBody} from './utils.js'; 
      addTextToBody('Modules are pretty cool.'); 
  </script>
```

### 2.不能写“裸”路径

```html
<script type="module">
    import {addTextToBody} from 'utils.js';  // error
    addTextToBody('Modules are pretty cool.'); 
</script>
```

### 3.如何向下兼容

使用 "nomodule" 关键字来实现浏览器的向下兼容

```html
  <script type="module" src="module.js"></script>
  <script nomodule src="fallback.js"></script>
```

### 4.默认加载方式

`type=“module”的加载方式默认使用 defer的加载方式。`

关于defer 和 async ：defer和async 都是异步加载代码。但是defer要等到整个页面在内存中正常渲染结束（DOM 结构完全生成，以及其他脚本执行完成），才会执行。 async一旦下载完，渲染引擎就会中断渲染，执行这个脚本以后，再继续渲染。 一句话，defer是“渲染完再执行”，async是“下载完就执行”。另外，如果有多个defer脚本，会按照它们在页面出现的顺序加载，而多个async脚本是不能保证加载顺序的。

defer，这个布尔属性被设定用来通知浏览器该脚本将在文档完成解析后，触发 DOMContentLoaded 事件前执行

**传统的内联**\<script>是没有defer这种概念的，从不异步，大家可以直接忽略，认为什么也没设置即可

`内联的 <script> 也是采用的 defer加载模式`

### 5.支持async的加载方式

type="module" 也可以使用 async 的方式进行加载(包括其内联的 import)，等同普通 js 采用 async 进行加载的方式。
无论是内联的module \<script>还是外链的\<script>，都支持async这个异步标识属性。这个有别于传统的\<script>，也就是传统\<script>仅外链JS才支持async，内联JS直接忽略async。

```js
  <script type="module" async></script>
```

### 6.只执行一次

<script type="module"> 只执行一次同ES6的加载机制，多次import 只会被当成一次import处理

```js
<!-- 1.js 只会被加载执行一次-->
<script type="module" src="1.js"></script>
<script type="module" src="1.js"></script>
<script type="module">
  import "./1.js";
</script>

<!--  普通JS 也只会被加载一次，但是会被执行多次-->
<script src="2.js"></script>
<script src="2.js"></script>
```

### 7.关于CORS

type="module" 默认不支持跨域，这一点儿与传统js或图片完全不一样。传统js或图片默认就是支持跨域的。如果你想 type="module" 支持跨域。需要在从服务器返回的header上显示的给予有效的 CORS声明

```js
// Access-Control-Allow-Origin: *
<!-- This will not execute, as it fails a CORS check -->
<script type="module" src="https://….now.sh/no-cors"></script>

<!-- This will not execute, as one of its imports fails a CORS check -->
<script type="module">
  import 'https://….now.sh/no-cors';

  addTextToBody("This will not execute.");
</script>

<!-- This will execute as it passes CORS checks -->
<script type="module" src="https://….now.sh/cors"></script>
```

### 8.默认情况下的凭据

```js
<!-- 使用凭证（Cookie等）获取 -->
<script src="1.js"></script>

<!--使用凭据获取，但在遵循旧规范的旧浏览器中除外-->
<script type="module" src="1.mjs"></script>

<!--在遵循新旧规范的浏览器中使用凭据获取的-->
<script type="module" crossorigin src="1.js"></script>

<!--无凭证获取-->
<script type="module" crossorigin src="https://other-origin/1.mjs"></script>

<!--使用凭证获取-->
<script type="module" crossorigin="use-credentials" src="https://other-origin/1.mjs"></script>
```

### 9.Mime-types

不同于传统的 \<scripts>, \<script type="module"> 必须向浏览器提供有效的 javascript MIME types。 不然请求到的模块javascript不会执行

\<script>支持的MIME类型包括text/javascript, text/ecmascript, application/javascript, 和application/ecmascript。如果没有定义这个属性，脚本会被视作JavaScript。

### 注意（与标准脚本的不同）

1. iOS的h5，module模块不会延后执行。
2. 本地测试 —  如果你通过本地加载Html 文件 (比如一个 file:\/\/ 路径的文件), 你将会遇到 CORS 错误，因为Javascript 模块安全性需要。你需要通过一个服务器来测试。
3. 从模块内部定义的脚本部分获得不同的行为，而不是标准脚本。 这是因为模块自动使用严格模式。
4. 加载一个模块脚本时不需要使用 defer 属性 (see \<script> attributes) 模块会自动延迟加载。
5. 最后一个但不是不重要，你需要明白模块功能导入到单独的脚本文件的范围 — 他们无法在全局获得。因此，你只能在导入这些功能的脚本文件中使用他们，你也无法通过Javascript console 中获取到他们，比如，在DevTools 中你仍然能够获取到语法错误，但是你可能无法像你想的那样使用一些debug 技术
6. 上面讲的是，静态import。
