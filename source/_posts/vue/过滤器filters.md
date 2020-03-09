---
title: 过滤器filters
date: 2020-01-30 18:44:21
tags: vue
---

过滤器filters
<!-- more -->

## 传参

+ 一个参数

```
<!-- 在双花括号中 -->
{{item.paytypeid | filtersChange}} // item.paytypeid-参数1;
```

```
<!-- 在 `v-bind` 中 -->
<div v-bind:id="item.paytypeid | filtersChange"></div>
```

后两种为网上搜集
```
<p v-html="$options.filters.filter(this.msg,arg1,arg2)"></p>
```

```
// methods中使用，并传参
methods:{
    fn(){
        let filter = this.$options.filters['filter']
        let data = filter(this.msg,arg1,arg2)
    }
}
```

+ 多个参数

```
{{item.paytypeid | filtersChange(item.state)}} // item.paytypeid-参数1；item.state-参数2；
```

## 组件使用的实例

```
<template>
  <div class="filters">
    <h1 v-text="filtersTitle"></h1>
    <input v-model="filtersText"/>
    <div>{{filtersText | filtersTextChange}}</div>
  </div>
</template>
<script>
  let vm = {};
  export default {
    data() {
      vm = this;
      return {
        filtersTitle: 'vue过滤器学习，filters',
        arrayList: [
          {
            "code": "1",
            "value": "北京市"
          },
          {
            "code": "2",
            "value": "上海市"
          },
        ],
        filtersText: '1',
      }
    },
    filters: {
      filtersTextChange: function (dataStr) {
        let arrayList = vm.arrayList;
        let value = '没有这个城市 ';
        for (let b of arrayList) {
          if (b.code == dataStr) {
            value = b.value;
            break;
          }
        }
        return value;
      }
    }
  }
</script>
```

## 全局使用

在创建 Vue 实例之前全局定义过滤器
```
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0).toUpperCase() + value.slice(1)
})

new Vue({
  // ...
})
```

## 过滤器串联

```
{{ message | filterA | filterB }}
```
filterA 被定义为接收单个参数的过滤器函数，表达式 message 的值将作为参数传入到函数中。然后继续调用同样被定义为接收单个参数的过滤器函数 filterB，将 filterA 的结果传递到 filterB 中。

+ 附官网链接：https://cn.vuejs.org/v2/guide/filters.html
