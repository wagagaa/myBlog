---
title: vue深度分析
date: 2019-11-13 16:06:35
tags: 
  - vue
  - vue深度
---

vue一些原理性的东西分析

<!-- more -->

## 响应式原理

## router路由原理

+ HTML5引入了 history.pushState() 和 history.replaceState() 方法，它们分别可以添加和修改历史记录条目。这些方法通常与window.onpopstate 配合使用。
+ 历史记录的改变会触发，window.onpopstate。

```
window.onpopstate = function(event) {
  alert("location: " + document.location + ", state: " + JSON.stringify(event.state));
};
//绑定事件处理函数
history.pushState({page: 1}, "title 1", "?page=1");    //添加并激活一个历史记录条目 http://x.com/?page=1,条目索引为1
history.pushState({page: 2}, "title 2", "?page=2");    //添加并激活一个历史记录条目 http://x.com/?page=2,条目索引为2
history.replaceState({page: 3}, "title 3", "?page=3"); //修改当前激活的历史记录条目 http://ex..?page=2 变为 http://ex..?page=3,条目索引为3
history.back(); // 弹出 "location: http://x.com/?page=1, state: {"page":1}"
history.back(); // 弹出 "location: http://x.com/, state: null
history.go(2);  // 弹出 "location: http://x.com/?page=3, state: {"page":3}
```

附

+ 前进: window.history.forward() / window.history.go(1)
+ 后退: window.history.back() / window.history.go(-1);
