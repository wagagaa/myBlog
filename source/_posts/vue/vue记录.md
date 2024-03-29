---
title: vue记录
date: 2019-10-30 18:05:43
tags: vue
---

记录平时遇到的问题
<!-- more -->

## 动态添加input

+ model:'item'  渲染后的input不能正常输入

```
mode:'items[index]' 这么替换后就可正常输入
```

## resetFields清除表单数据

 ```
 this.$refs[formName].resetFields();
 form表单设置 -- ref="formList"
 el-form-item上设置prop字段 -- prop="xxxx"
 ```

## 视图不更新

+ 强制刷新视图

`this.$forceUpdate()`

## nextTick里面的代码会在DOM更新后执行

```js
this.$nextTick(() => {
  // 待执行程序
})
```

## 监听

```js
  watch: {
    dataSource: {
      immediate: true,    // 这句重要
      handler (val) {
          this.items=val
      }
    }
  }
```

## 父子组件传值

### 父向子

```js
// 某个子组件中：
export default {
  props: {
    title: {
      type: String,
      default: 'hello world'
    }
  }
}

// 父组件中
<public-component :title='data'></public-component>
```

### 子向父

```html
<!-- 父组件 -->
<template>
    <div class="test">
      <test-com @childFn="parentFn"></test-com> // 接受子组件定义的事件
      <br/>
      子组件传来的值 : {{message}}
    </div>
</template>

<script>
export default {
    // ...
    data: {
        message: ''
    },
    methods: {
       parentFn(payload) {
        // 父组件可以在使用子组件的地方直接用v-on来监听子组件触发的事件。
        // $on(evntName)监听事件
        this.message = payload;
      }
    }
}
</script>

<!-- 子组件 -->
<template>
<div class="testCom">
    <input type="text" v-model="message" />
    <button @click="click">Send</button>
</div>
</template>
<script>
export default {
    // ...
    data() {
        return {
          // 默认
          message: '我是来自子组件的消息'
        }
    },
    methods: {
      click() {
          // $emit(eventName,optionalPayload)触发事件。（将事件和data传给父组件）
          this.$emit('childFn', this.message);
        }
    }
}
</script>

```

## 组件间跳转

### 传参

+ router-link
+ this.$router.push() (函数里面调用)

```js
// 1.  不带参数
this.$router.push('/home')
this.$router.push({name:'home'})
this.$router.push({path:'/home'})
// 2. query传参

this.$router.push({name:'home',query: {id:'1'}})
this.$router.push({path:'/home',query: {id:'1'}})

// html 取参  $route.query.id
// script 取参  this.$route.query.id

// 3. params传参
this.$router.push({name:'home',params: {id:'1'}})  // 只能用 name

// 路由配置 path: "/home/:id" 或者 path: "/home:id" ,
// 不配置path ,第一次可请求,刷新页面id会消失
// 配置path,刷新页面id会保留

// html 取参  $route.params.id
// script 取参  this.$route.params.id

// 4. query和params区别
// query类似 get, 跳转之后页面 url后面会拼接参数,类似?id=1, 非重要性的可以这样传, 密码之类还是用params刷新页面id还在
// params类似 post, 跳转之后页面 url后面不会拼接参数 , 但是刷新页面id 会消失
```

+ this.$router.replace() (用法同上,push)
+ this.$router.go(n)

```js
this.$router.go(n)
// 向前或者向后跳转n个页面，n可为正整数或负整数

// this.$router.push
// 跳转到指定url路径，并想history栈中添加一个记录，点击后退会返回到上一个页面
// this.$router.replace
// 跳转到指定url路径，但是history栈中不会有记录，点击返回会跳转到上上个页面 (就是直接替换了当前页面)

this.$router.go(n)
// 向前或者向后跳转n个页面，n可为正整数或负整数
```

### 接受参数

```js
// /router1?id=123、/router1?id=456 通过this.$route.query.id获取参数id的值。
// /router1/:id、/router1/123 通过this.$route.params.id来获取
```

## 父子组件调用方法

### 父组件调用子组件函数

用法： 子组件上定义ref="refName",  父组件的方法中用 this.$refs.refName.method 去调用子组件方法

子组件

```vue
<template>
  <div>
    childComponent
  </div>
</template>
 
<script>
  export default {
    name: "child",
    methods: {
      childClick(e) {
        console.log(e)
      }
    }
  }
</script>
```

父组件

```vue
<template>
  <div>
    <button @click="parentClick">点击</button>
    <Child ref="mychild" />   //使用组件标签
  </div>
</template>
 
<script>
  import Child from './child';   //引入子组件Child
  export default {
    name: "parent",
    components: {
      Child    // 将组件隐射为标签
    },
    methods: {
      parentClick() {
        this.$refs.mychild.childClick("我是子组件里面的方法哦");  // 调用子组件的方法childClick
      }
    }
  }
</script>
```

### 子组件调用父组件函数

1. 第一种方法是直接在子组件中通过this.$parent.event来调用父组件的方法
2. 第二种方法是在子组件里用$emit向父组件触发一个事件，父组件监听这个事件就行了。

```vue
// 父组件
<template>
  <div>
    <child @fatherMethod="fatherMethod"></child>
  </div>
</template>
<script>
  import child from '~/components/dam/child';
  export default {
    components: {
      child
    },
    methods: {
      fatherMethod() {
        console.log('测试');
      }
    }
  };
</script>
```

```vue
// 子组件
<template>
  <div>
    <button @click="childMethod()">点击</button>
  </div>
</template>
<script>
  export default {
    methods: {
      childMethod() {
        this.$emit('fatherMethod');
      }
    }
  };
</script>
```
3. 第三种是父组件把方法传入子组件中，在子组件里直接调用这个方法

```vue
// 父组件
<template>
  <div>
    <child :fatherMethod="fatherMethod"></child>
  </div>
</template>
<script>
  import child from '~/components/dam/child';
  export default {
    components: {
      child
    },
    methods: {
      fatherMethod() {
        console.log('测试');
      }
    }
  };
</script>
```

```vue
// 子组件
<template>
  <div>
    <button @click="childMethod()">点击</button>
  </div>
</template>
<script>
  export default {
    props: {
      fatherMethod: {
        type: Function,
        default: null
      }
    },
    methods: {
      childMethod() {
        if (this.fatherMethod) {
          this.fatherMethod();
        }
      }
    }
  };
</script>
```

## 引用assets中的图片

`< img :src="school ? school.logo : require('@/assets/logo.png')" class="logo-img" v-if="school&&school.logo"/>`

## 爷孙组件传值和调用方法

```js
// index.vue 照常-父子传值的父  

//Father.vue:
<Son v-bind="$attrs" v-on="$listeners"/>

//son.vue 照常-父子传值的子

```

+ $attrs是用来将数据从爷组件传递给孙组件的。
+ $listeners是用来从孙组件中触发爷组件中事件的。
