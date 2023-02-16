---
title: wenGPU
date: 2023-02-14 18:36:17
---

## 简介

+ 一种图形标准，主要在 Web 浏览器端使用 JavaScript 编程实现，也有其他语言（例如 Rust 的 Wgpu等）的实现。在这里以 Web 端为主，无特殊情况均指 Web 浏览器端的应用开发。

## webGPU和webGL的区别

### 起源不同

+ WebGL 的爹是 OpenGL，着色器语言 GLSL 继承自 OpenGL；WebGPU 的爹是 Metal、D3D12、Vulkan，标准的推动者有苹果、谷歌、Mozilla、科纳斯组（OpenGL 也是科纳斯组的规范之一）等，其着色器语言 WGSL 也是新设计的

+ WebGL 在设计之初就是拿来绘图的（的确如此），没考虑 GPU 的其它功能，后来才逐渐加入其它功能。WebGL 的上下文对象是与 canvas 元素强关联的，没有 canvas 创建不了上下文
+ WebGPU 则更强调“GPU”本身，它是需要自己制定绘制目标的

### 设计理念不同

+ WebGL 是走一步算一步的 API，若使用原生 API，需要对渲染的每一步进行精确调整，而且得按顺序，这意味着异步编程会比较麻烦。
+ WebGPU 是事先打包好数据、设计好计算过程、约定好输入输出，最后一次编码提交才进行计算，这种带“提前设计”的思维，提高了 GPU 访问数据的效率，减少了 CPU 到 GPU 的过程。

### 实际使用

WebGL 即使到了 version2，对通用计算的支持还是不太行，WebGL 的主战场还是绘图。

WebGPU 在娘胎里就设计好了，它有两大功能：绘图和通用计算。

不过，绘图方面，在量小的时候，二者几乎没有差别，WebGL 甚至写起来还快一些

### 现状

chrome开发者版可使用webGPU

[参考：WebGPU 摘学总目录](https://juejin.cn/post/7010596192606224397)
