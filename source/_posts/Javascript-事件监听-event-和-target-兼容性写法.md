---
title: Javascript 事件监听 event 和 target 兼容性写法
date: 2016-03-18 13:54:21
categories: 前端
tags: Javascript
---
IE,Chrome都有`window.event`对象
FireFox没有`window.event`对象

```javascript
elem.onclick = function(event) {
    var e = window.event || event;
    var target = e.srcElement || e.target;
}
```

以上写法能保证在这三个浏览器下获取事件对象和当前事件元素的兼容性。
