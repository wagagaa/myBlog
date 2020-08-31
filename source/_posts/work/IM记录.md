---
title: 直播中IM记录
date: 2020-07-15 11:43:41
---

直播只中的即时通讯IM使用流程和坑点记录一下。基于腾讯IM

<!-- more -->

## IM使用流程

+ 拿到appid和签名 => 登录IM => 创建群聊（或者进入群聊） => 发送消息

### 小程序

下载dome：https://gitee.com/cloudtencent/TIMSDK/tree/master/WXMini

### H5

下载dome：https://cloud.tencent.com/document/product/269/45036（附件功能比较多，单纯IM没有其他功能不建议用这个）
或者：https://gitee.com/cloudtencent/TIMSDK/tree/master/h5

如果是npm最后打包的话，可以直接用他的例子。
如果是html文件直接实例化tweblive.js。

秘钥（生成签名用）的话放在后端安全点。前端的话容易被蹭流量。
html这种方式引入的话，可以用es6的module。（GenerateTestUserSig.js）

文档地址：https://imsdk-1252463788.file.myqcloud.com/IM_DOC/Web/SDK.html

### 注意

+ 直播页面的聊天记录不显示
  
```js
 watch: {
    'chatList': function(){
      this.$nextTick(() => {
        var div = document.getElementById('chat')
        div.scrollTop = div.scrollHeight
       })

    }
  }
```

+ 实例化的构造函数是：TWebLive
