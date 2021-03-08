---
title: Vue实例$mount - 源码分析
date: 2021-01-13 15:47:31
tags: 
  - vue
  - vue源码分析
---

看到这一段代码new Vue().$mount('#app')。分析一下mount

<!-- more -->

## 创建Vue实例时的渲染过程

+ 在创建一个vue实例的时候(var vm = new Vue(options))。Vue的构造函数将自动运行 this._init（启动函数）。启动函数的最后一步为initRender(vm)

## Vue 实例挂载的实现

+ 在执行new Vue时,vue会去判断是否定义的了el参数,去执行$mount去挂载vm

## 调用$mount()方法

```js
// 在src/core/instance/init.js中
if (vm.$options.el) {
    vm.$mount(vm.$options.el)
}
```

## 执行$mount()方法

+ query()

```js
function query (el) {
  if (typeof el === 'string') {
    var selected = document.querySelector(el);
    if (!selected) { // 如果存在 el 元素，就直接返回 selected，否则如下
      process.env.NODE_ENV !== 'production' && warn(
        'Cannot find element: ' + el
      );
      return document.createElement('div') // 如果不存在el元素，就返回一个空div
    }
    return selected
  } else { // 否则直接返回dom元素
    return el
  }
}
```

+ vm的原型上本身就定义了一个$mount(如下所示)，然后通过重写$mount方法，最后返回的时候，调用这个缓存$mount方法。

```js
Vue.prototype.$mount = function ( // 这是最开始定义的$mount方法，在runtime-only版本中
  el,
  hydrating
) {
    //var inBrowser = typeof window !== 'undefined';
    el = el && inBrowser ? query(el) : undefined;
    return mountComponent(this, el, hydrating) // 然后执行 mountComponent方法
};
```

+ 直接调用的$mount代码如下

```js
Vue.prototype.$mount = function() {
    el = el && query(el); // 表示如果el存在就执行query(el)方法
    
    if (el === document.body || el === document.documentElement) {
        // Vue 不能挂载在 body、html 这样的根节点上，因为它会替换掉这些元素
        process.env.NODE_ENV !== 'production' && warn(
          "Do not mount Vue to <html> or <body> - mount to normal elements instead."
        );
        return this
    }
    
    // 如果有render方法，就直接 return 并调用原先原型原型上的 $mount 方法
    // 如果没有render方法，就判断有没有template
    // 如果没有template就会调用template = getOuterHTML(el)
    // 总之，最终还是会将template转换成render函数
    // 最后再调用 mountComponent 方法
    if (!this.$options.render) {
        if(this.$options.template){
            if (template.charAt(0) === '#') { // 我们这里的 charAt(0) 是 '<'
              template = idToTemplate(template);
              /* istanbul ignore if */
              if (process.env.NODE_ENV !== 'production' && !template) {
                warn(
                  ("Template element not found or is empty: " + (options.template)),
                  this
                );
              }
            }
        }
    }
    
    
    // 然后开始编译
    if (template) {
        // 通过compileToFunction 生成 render 函数，渲染vnode的时候会用到
        var ref = compileToFunctions(); 
        var render = ref.render;
        var render = ref.render;
        var staticRenderFns = ref.staticRenderFns; // 静态 render函数
        options.render = render; // 会在渲染 Vnode 的时候用到
        options.staticRenderFns = staticRenderFns;
    }
    
    return mount.call(this, el, hydrating)
    // 最后，调用原先原型上的 $mount 方法挂载。
    // 并执行 mountComponent() 方法
    // mountComponent 实在 core/instance/lifecycle 中   
}
```

## mountComponent()方法

```js
function mountComponent (
  vm,
  el,
  hydrating
) {
  vm.$el = el; // 对 el 进行缓存
  if (!vm.$options.render) { // 如果没有render函数，包括 template 没有正确的转换成render函数，就执行 if 语句
    vm.$options.render = createEmptyVNode; // createEmptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        // 如果使用了template而不是render函数但是使用的runtime-only版本，就报这个警告
        // 如果使用了template 但是不是用的 compile 版本，也会报警告
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        );
      } else {
        // 如果没有使用 template 或者 render 函数，就报这个警告
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        );
      }
    }
  }
  callHook(vm, 'beforeMount'); // 挂载前的一些准备工作

  var updateComponent;
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    // mark 是 util 工具函数中的perf，这里先不作深入研究，主要研究主线。
    // 性能埋点相关
    // 提供程序的运行状况，
    updateComponent = function () {
      var name = vm._name;
      var id = vm._uid;
      var startTag = "vue-perf-start:" + id;
      var endTag = "vue-perf-end:" + id;

      mark(startTag);
      var vnode = vm._render();
      mark(endTag);
      measure(("vue " + name + " render"), startTag, endTag);

      mark(startTag);
      vm._update(vnode, hydrating);
      mark(endTag);
      measure(("vue " + name + " patch"), startTag, endTag);
    };
  } else {
    updateComponent = function () {
      // vm._render() 方法渲染出来一个 VNode
      // jydrating 跟服务端渲染相关，如果没有启用的话，其为 false

      // 当收集好了依赖之后，会通过 Watcher 的 this.getter(vm, vm) 来调用 updateComponent() 方法
      vm._update(vm._render(), hydrating); // 然后执行 vm._render()方法
    };
  }
```

## updateComponent()

+ 在这个方法里有两个方法需要调用：vm._render() and vm._update(),先调用 _render 方法生成一个vnode，然后将这个vnode传入到 _update()方法中

```js
updateComponent = function () {
    // vm._render() 方法渲染出来一个 VNode
    // jydrating 跟服务端渲染相关，如果没有启用的话，其为 false
    // 当收集好了依赖之后，会通过 Watcher 的 this.getter(vm, vm) 来调用 updateComponent() 方法
    vm._update(vm._render(), hydrating); // 然后执行 vm._render()方法
};
```

## 过程总结

+ 先调用initRender。从中调用mount。验证挂载元素（有没有；是不是body）。有render方法就直接 return 并调用原先原型上的$mount。没有就通过template 调用getouterHTML和compileToFunctions 转化为render函数再执行。

+ 然后调用**mountComponent**方法。验证render函数。执行*boforeMount*钩子。定义updataComponent方法。实例化渲染**Watcher**（用来监听更新）。接着调用*monted*钩子。

+ 在Watcher里面的before钩子里执行*beforeUpdata*钩子。执行updataComponent方法。然后先调用 _render 方法生成一个vnode，然后将这个vnode传入到 _update()方法中。_update 方法把 VNode 渲染成真实的 DOM。
