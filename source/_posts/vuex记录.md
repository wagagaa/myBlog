---
title: vuex记录
date: 2019-11-05 17:40:51
tags: 
  - vue
  - vuex
---

## vuex解决的问题
+ vue中各个组件之间传值太痛苦，在vue中我们可以使用vuex来保存我们需要管理的状态值，值一旦被修改，所有引用该值的地方就会自动更新
+ state 数据存储
+ getter state的计算属性
+ mutation 修改状态（数据层操作）（明确地追踪到状态的变化）
+ action 异步操作 （逻辑层操作）
+ module 分成一个个模块，方便各个模块的状态管理


## 使用操作

### 获取定义
方法一：
+ vue：this.$store.data1
方法二：
+ vue：
```
this.$store.getters.addCount1
```
vuex: 
```
getters:(addCount1(state1){
   return state1.data1+1
})
```
### 操作vuex的data
方法一：
+ vue方法里
```
this.$store.commit("add1")
```
+ vuex:
```
mutations:{
    add1(state1){
      state1.data1=state1.data1+1
    }
}
```
方法二：（官方建议）
+ vue方法：
```
this.$store.dispatch(addFun)
```
+ vuex:
```
mutations:{
    add1(states){
      state1.data1=state1.data1+1
    }
}
actions:addFun(context){
   context.commit('add1')
}
```
注：只有 mutation 能动 State

方法三：
```
mutations: {
    addAge (state) {
      Vue.set(state.student, 'age', 18)
      // 或者：
      // state.student = { ...state.student, age: 18 }
    }
}
```

**扩展**
+ 给vuex的方法传值进去
vue:
```
this.$store.dispatch(addFun,10)
```
vuex:
```
mutations:{
    add1(states,n){
      state1.data1=state1.data1+1+n
    }
}
actions:addFun(context,n){
    context.commit('add1',n)
}
```
+ 辅助函数 mapState、mapGetters、mapActions





