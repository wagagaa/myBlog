---
title: vue-cli3部署到非根目录
date: 2020-07-30 14:47:31
tags: 
  - vue
  - vue-cli
---

history模式部署到非域名根路径下，遇到的问题及解决方法。

<!-- more -->

## 场景

+ 本地正常的vue项目打包成dist文件，部署到测试环境服务器上，页面空白，无报错也无请求；
+ 部署到服务器上第一页有页面，刷新后页面空白或404；
+ 引入css的type被拦截装换为“text/plain”；

## 思路

+ 前端部署路径publicPath是否正确;
+ 前端路由模式是否配置正确；
+ 后端配置是否正确；

## 解决方案

假设打包后的dist文件内容需要部署到非根目录http.xxx.com/m子路径下，解决步骤如下：

+ 修改vue.config.js中的publicPath

```js
  module.exports = {
    publicPath: "/m/", //打包后部署在一个子路径上http:xxx/m/
    productionSourceMap: false,
    devServer: {
      proxy: "http://xxxx.com", //测试或正式环境域名和端口号
    },
  };
```

+ 修改路由router的/index.js

```js
const router = new VueRouter({
  mode: "history",//路由模式
  base: "/m/",//部署的子路径
  routes,
});

export default router;
```

+ 后端同学帮忙配置
nginx配置：

```nginx
location /m {
  try_files $uri $uri/ /index.html;
}

# 实际中还用到了last;
location /m {
  index index.html;
  alias /com/daimawenjianjia;
  try_files $uri $uri/ /index.html last;
}
```

Apache配置：

```apa
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

原生Node.js:

```js
const http = require('http')
const fs = require('fs')
const httpPort = 80

http.createServer((req, res) => {
  fs.readFile('index.htm', 'utf-8', (err, content) => {
    if (err) {
      console.log('We cannot open "index.htm" file.')
    }

    res.writeHead(200, {
      'Content-Type': 'text/html; charset=utf-8'
    })

    res.end(content)
  })
}).listen(httpPort, () => {
  console.log('Server listening on: http://localhost:%s', httpPort)
})
```

## 深入理解为什么

### publicPath publicPath：部署应用包时的基本URL,默认是根目录./

```text
默认情况下，Vue CLI打包后的dist会被部署到域名的根目录下，例如http:xxxx.com。
```

### vue的两种Router（路由）模式

```text
Hash模式URL：http://www.xxxx.com/index.html#test，修改#后边的参数不会重新加载页面，不需要后台配置支持
History模式URL：http://www.xxxx.com/index.html，需要后台配置支持
```

### Vue-Router构建选项配置

```js
const router = new VueRouter({
  mode: "history",//路由模式,hash | history | abstract
  base: "/m/",//部署的子路径,如果打包部署在/m/下，base应该设置为'/m/'，所有的请求都会在url之后加上/m/
  routes,
});
```

