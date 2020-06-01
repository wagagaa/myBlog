---
title: Vue3的虚拟dom
date: 2020-05-21 16:33:44
---

静态标记，upadte性能提升1.3\~2倍，ssr提升2\~3倍，怎么做到的呢？

<!-- more -->

## Vue3的虚拟dom

### 编译模板的静态标记

我们来看一段很常见的代码

```html
<div id="app">
    <h1>技术摸鱼</h1>
    <p>今天天气真不错</p>
    <div>{{name}}</div>
</div>
```

vue2中会解析

```js
function render() {
  with(this) {
    return _c('div', {
      attrs: {
        "id": "app"
      }
    }, [_c('h1', [_v("技术摸鱼")]), _c('p', [_v("今天天气真不错")]), _c('div', [_v(
      _s(name))])])
  }
}
```

其中前面两个标签是完全静态的，后续的渲染中不会产生任何变化，Vue2中依然使用_c新建成vdom，在diff的时候需要对比，有一些额外的性能损耗

我们看下vue3中的解析结果

```js
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", { id: "app" }, [
    _createVNode("h1", null, "技术摸鱼"),
    _createVNode("p", null, "今天天气真不错"),
    _createVNode("div", null, _toDisplayString(_ctx.name), 1 /* TEXT */)
  ]))
}
// Check the console for the AST
```

最后一个_createVNode第四个参数1，只有带这个参数的，才会被真正的追踪，静态节点不需要遍历，这个就是vue3优秀性能的主要来源，再看复杂一点的

```html
<div id="app">
    <h1>技术摸鱼</h1>
    <p>今天天气真不错</p>

    <div>{{name}}</div>
    <div :class="{red:isRed}">摸鱼符</div>
    <button @click="handleClick">戳我</button>
    <input type="text" v-model="name">
</div>
```

解析的结果

```js
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", { id: "app" }, [
    _createVNode("h1", null, "技术摸鱼"),
    _createVNode("p", null, "今天天气真不错"),
    _createVNode("div", null, _toDisplayString(_ctx.name), 1 /* TEXT */),
    _createVNode("div", {
      class: {red:_ctx.isRed}
    }, "摸鱼符", 2 /* CLASS */),
    _createVNode("button", { onClick: _ctx.handleClick }, "戳我", 8 /* PROPS */, ["onClick"])
  ]))
}

// Check the console for the AST
```

_createVNode出第四个参数出现了别的数字，根据后面注释也很容易猜出，根据text，props等不同的标记，这样再diff的时候，只需要对比text或者props，不用再做无畏的props遍历, 优秀！ 借鉴一下劝退大兄弟的注释

```js
export const enum PatchFlags {
  
  TEXT = 1,// 表示具有动态textContent的元素
  CLASS = 1 << 1,  // 表示有动态Class的元素
  STYLE = 1 << 2,  // 表示动态样式（静态如style="color: red"，也会提升至动态）
  PROPS = 1 << 3,  // 表示具有非类/样式动态道具的元素。
  FULL_PROPS = 1 << 4,  // 表示带有动态键的道具的元素，与上面三种相斥
  HYDRATE_EVENTS = 1 << 5,  // 表示带有事件监听器的元素
  STABLE_FRAGMENT = 1 << 6,   // 表示其子顺序不变的片段（没懂）。 
  KEYED_FRAGMENT = 1 << 7, // 表示带有键控或部分键控子元素的片段。
  UNKEYED_FRAGMENT = 1 << 8, // 表示带有无key绑定的片段
  NEED_PATCH = 1 << 9,   // 表示只需要非属性补丁的元素，例如ref或hooks
  DYNAMIC_SLOTS = 1 << 10,  // 表示具有动态插槽的元素
}
```

如果同时有props和text的绑定呢， 位运算组合即可

```html
<div id="app">
    <h1>技术摸鱼</h1>
    <p>今天天气真不错</p>
    <div :id="userid"">{{name}}</div>
</div>
```

```js
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", { id: "app" }, [
    _createVNode("h1", null, "技术摸鱼"),
    _createVNode("p", null, "今天天气真不错"),
    _createVNode("div", {
      id: _ctx.userid,
      "\"": ""
    }, _toDisplayString(_ctx.name), 9 /* TEXT, PROPS */, ["id"])
  ]))
}

// Check the console for the AST
```

text是1，props是8，组合在一起就是9，我们可以简单的通过位运算来判定需要做text和props的判断, 按位与即可，只要不是0就是需要比较

### 事件缓存

绑定的@click会存在缓存里

```html
<div id="app">
  <button @click="handleClick">戳我</button>
</div>
```

```js
export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", { id: "app" }, [
    _createVNode("button", {
      onClick: _cache[1] || (_cache[1] = $event => (_ctx.handleClick($event)))
    }, "戳我")
  ]))
}
```

传入的事件会自动生成并缓存一个内联函数再cache里，变为一个静态节点。这样就算我们自己写内联函数，也不会导致多余的重复渲染 真是优秀啊

### 静态提升

```html
<div id="app">
    <h1>技术摸鱼</h1>
    <p>今天天气真不错</p>
    <div>{{name}}</div>
    <div :class="{red:isRed}">摸鱼符</div>
</div>
```

```js
const _hoisted_1 = { id: "app" }
const _hoisted_2 = _createVNode("h1", null, "技术摸鱼", -1 /* HOISTED */)
const _hoisted_3 = _createVNode("p", null, "今天天气真不错", -1 /* HOISTED */)

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", _hoisted_1, [
    _hoisted_2,
    _hoisted_3,
    _createVNode("div", null, _toDisplayString(_ctx.name), 1 /* TEXT */),
    _createVNode("div", {
      class: {red:_ctx.isRed}
    }, "摸鱼符", 2 /* CLASS */)
  ]))
}
```

## vue3和react fiber的vdom

很多人吐槽越来越像React，其实越来越像的api，代表着前端的两个方向

### Vue1.x

没有vdom，完全的响应式，每个数据变化，都通过响应式通知机制来新建Watcher干活，就像独立团规模小的时候，每个战士入伍和升职，都主动通知咱老李，管理方便

项目规模变大后，过多的Watcher，会导致性能的瓶颈

### React15x

而React15时代，没有响应式，数据变了，整个新数据和老的数据做diff，算出差异 就知道怎么去修改dom了，就像老李指挥室有一个模型，每次人事变更，通过对比所有人前后差异，就知道了变化，  看起来有很多计算量，但是这种immutable的数据结构对大型项目比较友好，而且Vdom抽象成功后，换成别的平台render成为了可能，无论是打鬼子还是打国军，都用一个vdom模式
碰到的问题一样，如果dom节点持续变多，每次diff的时间超过了16ms，就可能会造成卡顿（60fps）

### Vue2.x

引入vdom，控制了颗粒度，组件层面走watcher通知， 组件内部走vdom做diff，既不会有太多watcher，也不会让vdom的规模过大，diff超过16ms，真是优秀啊
就像独立团大了以后，只有营长排长级别的变动，才会通知老李，内部的自己diff管理了

### React 16 Fiber

React走了另外一条路，既然主要问题是diff导致卡顿，于是React走了类似cpu调度的逻辑，把vdom这棵树，微观变成了链表，利用浏览器的空闲时间来做diff，如果超过了16ms，有动画或者用户交互的任务，就把主进程控制权还给浏览器，等空闲了继续，特别像等待女神的备胎

diff的逻辑，变成了单向的链表，任何时候主线程女神有空了，我们在继续蹭上去接盘做diff，大家研究下requestIdleCallback就知道，从浏览器角度看 是这样的

```js
requestIdelCallback(myNonEssentialWork);
// 等待女神空闲
function myNonEssentialWork (deadline) {
    // deadline.timeRemaining()>0 主线程女神还有事件
    // 还有diff任务没算玩
    while (deadline.timeRemaining() > 0 && tasks.length > 0) {
    doWorkIfNeeded();
    }
    // 女神没时间了，把女神还回去🤣
    if (tasks.length > 0){
        requestIdleCallback(myNonEssentialWork);
    }
}
```

### Vue3

这里的静态提升和事件缓存刚才说过了，就不说了，其实我也很纳闷，这些静态标记和事件缓存，React本身也可以做，为啥就不实现了，连shouldComponentUpdate都得自己定义，为啥不把默认的组件都变成pure或者memo呢，唉，也许这就是人生把

React给你自由，Vue让你持久，可能也是现在国内Vue和React都如此受欢迎的原因把
Vue3通过Proxy响应式+组件内部vdom+静态标记，把任务颗粒度控制的足够细致，所以也不太需要time-slice了

人生啊，小孩才天天研究利弊， 成年人选择我都要，也期待React17的新特性
