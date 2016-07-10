---
title: Function的类型判断
date: 2016-03-18 13:47:55
categories: 前端
tags: Javascript
---
## 1. typeof

首先，使用 `typeof` 去判断一个函数是一种通用的做法，不过这种做法存在一种风险，即在低版本浏览器下会出现误判的现象，在 IE6/7/8 下，一些 JS 的核心函数是以 `COM` 对象去构造的函数，例如 `document.getElementById`，正常情况下，typeof 应该对其判断为 `function`，但是在 IE6/7/8 下，会将其判断为 `object`。
在 Chrome 和 Safari 的旧版本中，`正则表达式`进行 typeof 的判断结果为 `function`，其他浏览器则返回 `object`，这会导致误判，所以使用 typeof 判断函数是一种不稳定的做法，存在一定的风险。

## 2. 使用Object.prototype.toString.call()

* 使用 Object 的原型里的 toString 去判断类型也是一种方法，而且也稳定安全得多，其结果返回的格式是“[object NativeConstructorName]”
* 和 typeof 一样存在的一个问题是其在 IE6/7/8 仍会对某些核心函数判断为 `[object Object]`，这是 IE 特有的 `JScript` 所导致的，IE 中一些函数是以 `COM` 对象构造的。
* 在这种方法下，`正则表达式`的返回就正确了，结果为 `[object RegExp]`。

总结：在 IE9+ 和比较新的 Chrome、Safari 下，用 typeof 还是很方便的，而且效率比使用第二种方法快得多，在100万次的执行效率对比下，typeof 判断完消耗时间大概为`4.5ms`，而 Object.prototype.toString 则消耗高达`50ms`以上，其效率差距10几倍。以下代码可以判断是否使用 typeof 判断 Function：

```javascript
if(typeof /./ != "function" && typeof Int8Array != "object") {
    np.isFunction = function(func) {
        return typeof func === "function" || false;
    }
}
```
