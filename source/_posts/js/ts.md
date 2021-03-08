---
title: ts问题记录
---

对于ts遇到的问题做个记录

<!-- more -->

## [TS] PROPERTY 'AAA' DOES NOT EXIST ON TYPE 'WINDOW'

```js
// 第一种
(window as any).aaa

// 第二种
declare global {
    interface Window { aaa: any; }
}

window.aaa = window.aaa || {};

// 第三种：
interface MyWindow extends Window {
    aaa(): void;
}
 
declare var window: MyWindow;
```
