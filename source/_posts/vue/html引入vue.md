---
title: html引入vue组件
date: 2020-02-18 17:14:02
tags: 
    - vue
---

在html中运用vue组件
<!-- more -->

## 1.引用js

```
<script src="https://unpkg.com/http-vue-loader"></script>
```

## 2.使用httpVueLoader

```
Vue.use(httpVueLoader);
```

## 3.引用组件 

```
  components: {
      // 将组件加入组建库
      'my-component': 'url:./component/my-component.vue'
  }
```

## 4.调用组件

```
  <div id="app">
      <my-component></my-component>
  </div>
```

## 注意

+ 不能直接点击打开。这样不能加载组件。需要通过nginx转发。错：file:///Users/。对：http://localhost

## 附实例

+ my-component.vue

```
  <template>
      <div class="hello">Hello {{who}}</div>
  </template>
  
  <script>
  module.exports = {
      data: function() {
          return {
              who: 'world'
          }
      }
  }
  </script>
  
  <style>
  .hello {
      background-color: #ffe;
  }
</style>
```

+ index.html

```
  <!DOCTYPE html>
  <html>

  <head>
      <meta charset="UTF-8">
      <!-- 引入样式 -->
      <link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
      <!-- 先引入 Vue -->
      <script src="https://unpkg.com/vue/dist/vue.js"></script>
      <!-- 引入 http-vue-loader -->
      <script src="https://unpkg.com/http-vue-loader"></script>
  </head>

  <body>
      <div id="app">
          <my-component></my-component>
      </div>
  </body>

  <!-- 引入组件库 -->
  <script src="https://unpkg.com/element-ui/lib/index.js"></script>
  <script>
      // 使用httpVueLoader
      Vue.use(httpVueLoader);

      new Vue({
          el: '#app',
          data: function () {
              return { visible: false }
          },
          components: {
              // 将组建加入组建库
              'my-component': 'url:./component/my-component.vue'
          }
      })
  </script>

  </html>
```

+ httpVueLoader的其他组件载入方式可查看：https://www.npmjs.com/package/http-vue-loader
