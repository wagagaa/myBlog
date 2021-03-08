---
title: vm._render() - 源码分析
date: 2021-01-15 14:05:31
tags: 
  - vue
  - vue源码分析
---

在上一系列中，我么已经了解到$mount是如何工作的，最后通过调用 updateComponent方法来执行vm._update(vm._render(), hydrating);,接下来我们来分析一下 vm._render()方法是如何运行的。

<!-- more -->

