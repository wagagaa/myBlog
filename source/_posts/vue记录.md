---
title: vue记录
date: 2019-10-30 18:05:43
tags: vue
---

记录平时遇到的问题
<!-- more -->

## 动态添加input
- model:'item'  渲染后的input不能正常输入

```
mode:'items[index]' 这么替换后就可正常输入
```

 ## resetFields清除表单数据
 ```
 this.$refs[formName].resetFields();
 form表单设置 -- ref="formList"
 el-form-item上设置prop字段 -- prop="xxxx"
 ```
