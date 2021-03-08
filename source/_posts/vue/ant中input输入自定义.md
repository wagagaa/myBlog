---
title: ant的input操作集合
date: 2021-01-21 15:07:33
tags: 
  - vue
  - ant
---

input输入进行为所欲为的展示

<!-- more -->

## 遇到的情况

+ 第一种情况。引入unity打包的webgl后，input的输入只能汉字输入。字母数字不行了（原生基础input也不行）~
+ 第二种情况。使用扫码枪，对拿到的字符串进行再加工

## 监听键盘事件

```js

textInput(id: string) {
  (document as any).getElementById(id).addEventListener('keyup', this.keyup) // 回调方法（this.keyup）要封装
}
keyup(e: any) {
  const inputData: string = e.target.id
  let originInput = (this as any)[inputData]
  if (e.key == 'Backspace') { // Backspace-回退标志
    (this as any)[inputData] = originInput.substr(0, originInput.length - 1)
  } else if (e.key.length == 1 && e.key !== ' ' && e.isComposing == false) { // isComposing-非汉字
    (this as any)[inputData] = originInput + e.key
  }
}
```

## input事件

```js
<a-textarea v-model="someSerial" @change="changeSome"/>

changeSome(e) {
  e.target.selectionStart = this.someSerial.length-1 // 光标开始位置
  e.target.selectionEnd = this.someSerial.length // 光标结束位置
}
```

## 获取input焦点

```js
this.$nextTick(() => {
  this.$refs['name' + (index + 1)].focus()
})
```
