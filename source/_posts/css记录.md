---
title: css记录
date: 2020-02-06 18:17:45
tags: css
---

年即大了，把一些容易忘记的css记录下来

<!-- more -->

## 文本省略号

+ 单行文本

```css
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
```

+ 多行文本

```css
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 2;
```

```css
word-break: break-all;
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 2;
overflow: hidden;
text-overflow: ellipsis;
```

## 其他

+ 中划线

`text-decoration: line-through;`

+ 图片文本居中

`vertical-align: middle;`

+ 部分手机不能滑动

```css
  #app::-webkit-scrollbar {
    display: none;
  }

  #app {
    overflow-x: hidden;
  }
```

+ 小程序switch

```css
  .wx-switch-input{
  　　zoom: 0.8 (缩放比例)
  }
```

+ 设置空img的css样式

```css

  img[src=""],img:not([src]){
      opacity:0;
  }
```

+ 宽度不自动100%；

```css
第一种: display:inline-block
第二种: width:fit-content;
```

+ 小程序不出现滚动轴

```css
::-webkit-scrollbar{
  width: 0;
  height: 0;
  color: transparent;
}
```

+ 段落空两格

`text-indent: 2em;`

+ file文件选择的样式

input[type="file"]::file-selector-button
