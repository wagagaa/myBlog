---
title: nginx的使用记录
date: 2020-03-09 12:00:05
tags: 
  - 服务器
  - 跨域
---

本地跨域。通过nginx做转发.

<!-- more -->

## 使用背景

+ 本地前端开发接口跨域。后端没有测试服务器，后端不配合设置请求头信息。没办法，自己搞nginx吧

## 步骤

+ 1: 安装
`brew install nginx`

+ 2: 启动nginx服务('nginx'这个命令同样能启动)
`brew services start nginx`

+ 3: 更改配置文件
`sudo vim /usr/local/etc/nginx/nginx.conf`
    ## 数字化平台的地址
    server {
      listen       8094; # 端口号
      server_name  localhost; # 需要代理的地址
      autoindex on;
      location /api/ {
          proxy_pass http://digitizing.qa.growlib.com;
#          proxy_pass http://glschool.qa.growlib.com;
      }

      location / {
          root /Users/xiaoxiong/Downloads/web/; # 不做拦截后，取本地的文件
          index index.html; # 默认的请求的文件
      }
    }

## nginx.conf配置信息的填写

```conf
  server {
      listen       8021; # 端口号
      server_name  localhost; # 需要代理的地址
      autoindex on;
      location /outside/ { // # 什么路径拦截
          proxy_pass https://single.yuetao.group; # 最后的实际请求地址
      }
      location /mall/ {
          proxy_pass https://single.yuetao.group;
      }
      location /share/ {
          proxy_pass https://single.yuetao.group;
      }
      location / {
          root /Users/yuedian/Documents/yuelv/Single_page/; # 不做拦截后，取本地的文件
          index index.html; # 默认的请求的文件
      }
  }
```

## nginx命令

```conf
  nginx #启动nginx
  nginx -s quit #快速停止nginx
  nginx -V #查看版本，以及配置文件地址
  nginx -v #查看版本
  nginx -s reload|reopen|stop|quit #重新加载配置|重启|快速停止|安全关闭nginx
  nginx -h #帮助
  brew uninstall nginx # 卸载nginx
```

+ 需要sudo操作。更改配置文件后，需要重启。

## 问题集合

+ 全部访问403

```conf
1: ** /home/resume/www**目录下的所有文件权限改为777 # 并没什么卵用
    chmod -R 777 /Users/yuedian/Documents/yuelv/Single_page
2: 配置nginx权限和静态文件统一。（成功）
    # user 用户名 用户组  ; 这里的用户名和组就是静态文件的
    配置文件：user scott executor ;
    whoami # 查看当前用户的用户
    groups # 查看当前用户所属组（Note：用户所属组可能有多个）
```

+ 安装的时候报错，最后显示成功。

``` js
不用管，能正常启动nginx就行。
brew services start nginx 这个命令启动不了。
但是 sudo nginx 这个命令可以。能启动就行
```

+ 启动失败

```js
可能是端口被占用；也可能是数字太低（具体是哪个数字忘了~）。
```

+ 参考链接

```js
  https://juejin.im/post/5aa7704c6fb9a028bb18a993
  https://segmentfault.com/a/1190000018453157
  https://segmentfault.com/a/1190000019894251?utm_source=coffeephp.com
  https://www.cnblogs.com/PyKK2019/p/10530412.html
  https://www.jianshu.com/p/e0dadb871894
  https://www.jianshu.com/p/3d0a237ea55c
```

## 附：另一种解决办法

+ chrome插件:Allow CORS: Access-Control-Allow-Origin

安装就能解决本地跨域。不能翻墙的，先去网上安装'谷歌上网助手'或者'谷歌访问助手'
