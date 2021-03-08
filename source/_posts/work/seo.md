---
title: seo优化记录
date: 2021-03-05 11:07:45
tags: 
  - seo
---

近期对官网进行了seo优化。从seo的这个整体优化和前端的具体实施记录

<!-- more -->

## 整体seo分析

+ 见效慢。
+ 需要：关键字要精练不是越多越好。不可频繁更换。title精练。
+ 前端优化：整站3秒内加载(重要)、语义化标签、img添加alt。
+ 布局优化：面包屑导航、底部有公共链接、有筛选和分类
+ 少用iframe、提高网站速度、流量

## 基于vue的seo优化

### vue不利于seo的原因

这与VUE的本质相关，VUE是一个SPA应用，就是只有一个Web页面的应用，是加载单个HTML页面，并在用户与应用程序交互时动态更新该页面的Web应用程序。

为什么说VUE不利于SEO，SEO本质是一个服务器向另一个服务器发起请求，解析请求内容。但一般来说搜索引擎是不会去执行请求到的js的。也就是说，如果一个单页应用，html在服务器端还没有渲染部分数据数据，在浏览器才渲染出数据，而搜索引擎请求到的html是没有渲染数据的， 所以就很不利于内容被搜索引擎搜索到。

现在主要采用的有以下四种方式：

SSR服务器渲染；
静态化；
预渲染prerender-spa-plugin；
使用Phantomjs针对爬虫做处理。

### 1.SSR服务器渲染

### 2.静态化

Nuxt.js框架

### 3.预渲染prerender-spa-plugin

改善少数页面的 SEO，那么你可能需要预渲染。无需使用 web 服务器实时动态编译 HTML，而是使用预渲染方式，在构建时 (build time) 简单地生成针对特定路由的静态 HTML 文件。优点是设置预渲染更简单，并可以将你的前端作为一个完全静态的站点。

这里就详细的讲解下如何在VUE-CLI3的项目中使用prerender-spa-plugin。

+ 安装
`cnpm install prerender-spa-plugin --save`

在VUE-CLI3的项目中调整 webpack 配置最简单的方式就是在 vue.config.js 中的 configureWebpack 选项提供一个对象。官方文档：https://cli.vuejs.org/zh/guide/webpack.html

```js
// vue.config.js
const PrerenderSPAPlugin = require('prerender-spa-plugin')
const Renderer = PrerenderSPAPlugin.PuppeteerRenderer
const path = require('path')
module.exports = {
    configureWebpack: config => {
        // 为开发环境修改配置...
        if (process.env.NODE_ENV !== 'production') return;
        // 为生产环境修改配置...
        return {
            plugins: [
                new PrerenderSPAPlugin({
                   // 网页包的路径将应用程序输出到prerender
                    staticDir: path.join(__dirname,'dist'),
                    // Routes to render 对应自己router
                    routes: ['/', '/home','/blog','/aboutMe','/message'],
                    renderer: new Renderer({
                        inject: {
                            foo: 'bar'
                        },
                        // 渲染时显示浏览器窗口。对调试有用。
                        headless: false,
                        // 在 main.js 中 document.dispatchEvent(new Event('render-event'))，两者的事件名称要对应上。
                        renderAfterDocumentEvent: 'render-event'
                    })
                }),
            ],
        };
    }
}
```

```js
// main.js
new Vue({
  router,
  store,
  render: h => h(App),
  //这里与vue.config.js中的事件名相对应
  mounted () {
    document.dispatchEvent(new Event('render-event'))
  }
}).$mount('#app')
```

最后有点一定要注意，router中必须是history模式。

好了，到这里就已经配置完了，只需要和传统的打包方式一样打包就可以，打包的过程中会看到浏览器自动打开许多页面然后自动关闭，这就是打包的过程。
`npm run build`

打开打包后的dist文件夹,现在你看到的会比之前多几个文件夹，多的文件夹正是你配置的那几个路由，每个文件夹中都是一个index.html文件。原本只有一个index.html的单页应用现在变成了多页应用，这也就提升了你的网页被抓取的概率。

到这里我们还可以继续优化，虽然现在已经成了多个页面，但是每个页面的title，description，keywords都是一样的，这也往往不符合实际，毕竟每个页面都有自己的主题内容。

那么我们可以针对每个页面对他们的meta-info分别设置，这里可以使用vue-meta-info插件。

+ 安装
`npm install vue-meta-info --save`

引用，在main.js中

```js
import MetaInfo from 'vue-meta-info'
Vue.use(MetaInfo)
```

在你需要的页面中使用

```js
export default {
  metaInfo:{
    title: 'message',
    meta: [
      {
        name: 'description',
        content: '这是Message页面',
      },
      {
        name: 'keywords',
        content: 'message'
      }
    ]
  },
  data(){
    return {
    }
  },
}
```

linux服务器部署注意：
服务器上部署是，出现Error: Failed to launch chrome!(因为打包的时候会去调用浏览器，去生成静态页)
在服务器上分别执行下面两行命令即可

`yum install pango.x86_64 libXcomposite.x86_64 libXcursor.x86_64 libXdamage.x86_64 libXext.x86_64 libXi.x86_64 libXtst.x86_64 cups-libs.x86_64 libXScrnSaver.x86_64 libXrandr.x86_64 GConf2.x86_64 alsa-lib.x86_64 atk.x86_64 gtk3.x86_64 -y`

`yum install ipa-gothic-fonts xorg-x11-fonts-100dpi xorg-x11-fonts-75dpi xorg-x11-utils xorg-x11-fonts-cyrillic xorg-x11-fonts-Type1 xorg-x11-fonts-misc -y`

再次打包时如果出现如下错误，更新glib2
`chrome-linux/chrome: symbol lookup error: /lib64/libpango-1.0.so.0:undefined symbol: g_log_structured_standard`

`yum update glib2`

### 4.使用Phantomjs针对爬虫做处理

或在后端做http拦截，检查请求代理User-Agent是不是爬虫，如果检测到是爬虫类型，动态生产html片段，仅为爬虫服务。

## 扩展sem

sem：其实就是花钱做的推广

## 打包后的执行

vue打包后的执行顺序：preload加载两个js（一个是：vue、vue-router、vuex等等的集合；一个是：router的配置信息和处理逻辑），通过跳转逻辑去请求对应页面的js和css(build的时候对vue的scrtip和style进行提取压缩)，生成当前页面。跳转的时候获取地址栏信息，加载对应页面的js和css生成内容。

prerender-spa-plugin部分页面静态化打包后的执行顺序：未静态化的逻辑和之前一样。
静态化的执行顺序：打包的时候会加载对应目录生成index.html文件。解析index文件时候会先加载3个+的js（一个是：vue、vue-router、vuex等等的集合；一个是：router的配置信息和处理和处理逻辑；一个是当前页面的信息集合；可能会有一个公共模块信息，比如公共头部尾部，会生成这个文件）（这3个+的js都是build写好的加载顺序）。一开始dom会去加载build的时候生成的dem和css。区别于正常的只有#app。通过之前build生成的结构就可以更好的利于seo。跳转的话和正常的跳转处理一致。
