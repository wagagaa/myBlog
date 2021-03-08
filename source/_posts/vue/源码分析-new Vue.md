---
title: new Vue() - 源码分析
date: 2021-01-15 11:21:31
tags: 
  - vue
  - vue源码分析
---

对 vue 的源码分析的第一篇

<!-- more -->

+ import Vue from 'vue'过程中，Vue 初始化主要就干了几件事情，合并配置，初始化生命周期，初始化事件中心，初始化渲染，初始化 data、props、computed、watcher 等等。目前主要注意一下initMixin方法

## initMixin()

```js
function initMixin (Vue) {
  Vue.prototype._init = function (options) { // 在Vue的原型上定义 _init 方法
    var vm = this;
    vm._self = vm;
    initLifecycle(vm); // 初始化生命周期
    initEvents(vm); // 事件
    initRender(vm); // render 方法
    callHook(vm, 'beforeCreate');
    initInjections(vm); // resolve injections before data/props
    initState(vm); // 初始化状态
    initProvide(vm); // resolve provide after data/props
    callHook(vm, 'created');

    if (vm.$options.el) { // 挂载dom元素
      vm.$mount(vm.$options.el);
    }
  };
}
```

## new Vue()

+ 通过initMixin方法之后，执行 new Vue() 操作，在 Vue构造函数内部，调用了vm原型上的 _init()方法，在this._init()方法中我们目前主要关注的是 initState()方法

```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword');
  }
  this._init(options); // 这是在Vue原型上定义了的方法(initMixin方法中)，使构建出来的Vue实例调用
}

new Vue({
  el: '#app',
  render: h => h(App)
})
```

## initState()

```js
function initState (vm) {
  vm._watchers = [];
  var opts = vm.$options;
  if (opts.props) { initProps(vm, opts.props); }
  if (opts.methods) { initMethods(vm, opts.methods); }
  if (opts.data) {
    initData(vm); // 初始化数据，并且做数据代理
  } else {
    observe(vm._data = {}, true /* asRootData */);
  }
  if (opts.computed) { initComputed(vm, opts.computed); }
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch);
  }
}
```

## initData()

+ 在这个方法中，需要注意的是 proxy() 方法

```js
function initData (vm) {
  var data = vm.$options.data;
  data = vm._data = typeof data === 'function' // 在vm 上定义一个 '_data' 属性
    ? getData(data, vm) // getData()方法返回的就是我们在vue data()方法中定义了的属性和方法
    : data || {};
  if (!isPlainObject(data)) {
    data = {};
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    );
  }
  // proxy data on instance
  var keys = Object.keys(data);
  var props = vm.$options.props;
  var methods = vm.$options.methods;
  var i = keys.length;
  while (i--) {
    var key = keys[i];
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(
          ("Method \"" + key + "\" has already been defined as a data property."),
          vm
        );
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' && warn(
        "The data property \"" + key + "\" is already declared as a prop. " +
        "Use prop default value instead.",
        vm
      );
    } else if (!isReserved(key)) {
      proxy(vm, "_data", key); // 数据代理
    }
  }
  // observe data
  observe(data, true /* asRootData */);
}
```

## proxy()

+ 在这个方法中，通过Object.defineProperty()来实现数据劫持，当我们通过 this.xxx 访问我们定义的数据时，其实就是访问的 this._data.xxx，从而达到数据代理的目的。

```js
var sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
};

function proxy (target, sourceKey, key) {
// target是vm, sourceKey是'_data', key 是我们定义的data中的键
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  };
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val;
  };
  Object.defineProperty(target, key, sharedPropertyDefinition);
}
```

## 总结

+ 在初始化的最后，检测到如果有 el 属性，则调用 vm.$mount 方法挂载 vm，挂载的目标就是把模板渲染成最终的 DOM，在下个系列中将分析Vue实例挂载的实现
