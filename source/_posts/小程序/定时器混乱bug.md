---
title: 定时器混乱bug
date: 2020-03-28 16:31:15
tag: 
  - 定时器
  - 小程序
---

小程序的倒计时有时候会出现混乱。

<!-- more -->

## 起因

+ 小程序倒计时走的好好的。如果你切换到桌面，再进来的时候，倒计时会混乱。

## 解决办法

```js
// 一: 在小程序方法中直接清除定时器。没有成功
// 二: 外部定义一个数组，将每次添加的定时器加入进去。onshow的话循环清除数组里定时器，然后数组清空。成功
```

## 实例

```js
let barrageTimerList=[] // 定时器倒计时

  // 倒计时
  startTimer: function(totalSecond, v) {
    let that = this
    // 倒计时
    var totalSecond = totalSecond;
    var type = type
    console.log(totalSecond)
    if (totalSecond == 0) {
      console.log('daojishijieshu')
      that.setData({})
    } else {
      if (interval){
        clearInterval(interval);
      }
        var interval = setInterval(function() {
        // 秒数
        var second = totalSecond;
        // 天数位
        var day = Math.floor(second / 3600 / 24);
        var dayStr = day.toString();
        if (dayStr.length == 1) dayStr = '0' + dayStr;

        // 小时位
        var hr = Math.floor((second - day * 3600 * 24) / 3600);
        var hrStr = hr.toString();
        if (hrStr.length == 1) hrStr = '0' + hrStr;

        // 分钟位
        var min = Math.floor((second - day * 3600 * 24 - hr * 3600) / 60);
        var minStr = min.toString();
        if (minStr.length == 1) minStr = '0' + minStr;

        // 秒位
        var sec = second - day * 3600 * 24 - hr * 3600 - min * 60;
        var secStr = sec.toString();
        if (secStr.length == 1) secStr = '0' + secStr;

        this.data.list[v].countDownHour = hrStr;
        this.data.list[v].countDownMinute = minStr;
        this.data.list[v].countDownSecond = secStr;
        this.setData({
          list: this.data.list
        });
        totalSecond--;
        if (totalSecond < 0) {
          clearInterval(interval);
          this.data.list[v].countDownHour = '00';
          this.data.list[v].countDownMinute = '00';
          this.data.list[v].countDownSecond = '00';
        }

      }.bind(this), 1000);
      barrageTimerList.push(interval) // 添加到数组中
    }

  },

  onShow: function() {
    // 清除倒计时
    barrageTimerList.forEach((item, index) => {
      clearInterval(item)
    })
    barrageTimerList = []
  },


```
