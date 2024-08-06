---
title: qiankun乾坤微前端的应用
date: 2024-08-05 11:40:28
---

## 应用背景

项目中需要有一部分功能外包给另一个公司，两个项目开发要保持相对独立，但是用户是无感知的。有考虑过使用iframe，但是用户体验和开发效率都很低。

iframe的缺点。两个应用隔离。

 1. 地址栏的前进后退刷新和实际用户使用有偏差
 2. 跨应用通信麻烦。
 3. 样式的全局性无法保持一致。
 4. 不同网址下带来的跨域等安全性问题

## qiankun的原理

1. 监听路由的变化
2. 通过import-html-entry加载对应的子应用

## qiankun的使用。vue2主应用+vue3、vite子应用

使用前建议先看一遍[官网](https://qiankun.umijs.org/zh/guide)

### 主应用

```js
// layout.vue
const entryMap = {
  development: '//localhost:9522',
  qa: 'qa.**.com',
  production: '**.com'
}
const apps = [
  {
    name: 'vue3',
    entry: entryMap[process.env.NODE_ENV], // 默认会加载这个html 解析里面的js 动态的执行 （子应用必须支持跨域）fetch
    container: '#vue3',
    activeRule: '/vue3'
  }
]
registerMicroApps(apps) // 注册应用
start({
    prefetch: false, // 取消预加载
    sandbox: {
      // experimentalStyleIsolation: true
    }
  })// 开启

// 如果子应用的dome节点在app.vue中，可以使用loadMicroApp注册
```

### 子应用

```js
// main.ts
const initQianKun = () => {
  renderWithQiankun({
    bootstrap() {
      // console.log('微应用：bootstrap')
    },
    mount(props) {
      // 获取主应用传入数据
      // console.log('微应用：mount', props)
      render(props)
    },
    unmount(props) {
      // console.log('微应用：unmount', props)
    },
    update(props) {
      // console.log('微应用：update', props)
    },
  })
}

qiankunWindow.__POWERED_BY_QIANKUN__ ? initQianKun() : render() // 判断是否使用 qiankun ，保证项目可以独立运行
// vite.config.ts
import qiankun from 'vite-plugin-qiankun' // qiankun官方不支持vite，需要引入这个插件(https://github.com/tengmaoqing/vite-plugin-qiankun)

base: env.VITE_APP_BASE_URL, // 生产环境需要指定运行域名作为base
plugins: [
  vue(),
  vueJsx(),
  qiankun(`vue3`, {
    useDevMode: true
  }),
],
server: {
  headers: {
    'Access-Control-Allow-Origin': '*' // 开发阶段需要配置跨域，防止主应用在开发阶段拿不到子应用的资源
  },
},
```

### 生产环境的nginx

```nginx
## 子应用有单独的域名，否则需要单独的路由在主应用进行转发
## 子应用需要配置跨域
server {
  location / {
    # 添加 CORS 头
    add_header 'Access-Control-Allow-Origin' '*';
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
    add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
  }
}
```

## 遇到的问题

### 使用antd版本的样式冲突

+ 解决办法

```js
// 1. 子应用全局更改类名 a-config-provider prefixCls="vue3"
// 2. 主应用信息更改类名
message.config({
  prefixCls: 'vue3',
});
```

### 子应用内路由跳转后，后退会出现404

+ 解决办法：子应用路由切换的时候更改浏览器的history记录。参考这个[issues](https://github.com/umijs/qiankun/issues/2254)

```js
const base = window.__POWERED_BY_QIANKUN__ ? '/vue3' : ''
router.afterEach((to) => {
    const state = {
        ...history.state,
        current: base + to.fullPath
    }
    history.replaceState(state, '', window.location.href)
    nProgress.done(true)
})
```

### 作为子应用部分资源夹加载不到

```js
1. js中通过window.open打开的需要添加baseUrl
2. 记得配置base: env.VITE_APP_BASE_URL
```

其他参考的文章也可以看看的：https://juejin.cn/post/7205401322111385656 https://juejin.cn/post/7242623208841592869 https://juejin.cn/post/6854573214564089864
