---
title: ant-design-pro的使用及踩坑记录
date: 2020-02-26 14:32:33
tags: vue
---

ant 有关的记录和一些踩坑记录

<!-- more -->

## 表格复选框禁止选中

```js
computed: {
    rowSelection() {
        const _this = this
        const { selectedRowKeys } = this
        return {
        selectedRowKeys,
        onChange: (selectedRowKeys) => {
            this.selectedRowKeys = selectedRowKeys
        },
        getCheckboxProps: (record) => ({
            props: {
                // 全部默认禁止选中
                // disabled: true,
                // 某几项默认禁止选中(R: 当state等于1时)
                disabled: record.state === 1,
    　　　　　　　// 某几项默认选中(R: 当state等于1时)
    　　　　　　　// defaultChecked: record.state == 1,
    　　　　},
    　　　}),
    　　}
    },
}
```

## 选择器会跟随滚动条上下移动

1 < Select getPopupContainer={trigger => trigger.parentElement} >
2 且保证 parentElement 是 position: relative 或 position: absolute。
[#3487](https://github.com/ant-design/ant-design/issues/3487)
