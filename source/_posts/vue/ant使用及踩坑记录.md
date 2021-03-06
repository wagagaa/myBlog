---
title: ant-design-pro的使用及踩坑记录
date: 2020-11-26 14:32:33
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
    },
}
```

## 选择器会跟随滚动条上下移动

1 < Select getPopupContainer={trigger => trigger.parentElement} >
2 且保证 parentElement 是 position: relative 或 position: absolute。
[#3487](https://github.com/ant-design/ant-design/issues/3487)

## 筛选table

```js
column = [
    {
        title: "标签名称",
        dataIndex: "title",
        align: "center",
        filteredValue: this.title ? [this.title] : null,  // 关键
        onFilter: (value, record) => record.title.indexOf(value) >= 0 // 关键
    },
    {
        title: "标签简介",
        dataIndex: "summary",
        align: "center"
    },
    {
        title: "创建时间",
        dataIndex: "created_at",
        align: "center"
    },
    {
        title: "操作",
        dataIndex: "operation",
        scopedSlots: { customRender: "operation" },
        align: "center"
    }
];
```

## select分页（到底触发）

```js
@popupScroll="onPopupScroll"

onPopupScroll (e) {
const target = e.target
    if (target.scrollTop + target.offsetHeight === target.scrollHeight) {
    // scrollToEnd, do something!!!
    }
},
```
