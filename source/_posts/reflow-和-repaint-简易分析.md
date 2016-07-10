---
title: reflow 和 repaint 简易分析
date: 2016-06-17 10:16:50
categories:
    - 前端
    - CSS
tags: CSS
---
![福利](http://7xoehm.com1.z0.glb.clouddn.com/1448333443367.jpg)

# 浏览器渲染
当我们打开一个网页的时候，浏览器是如何将 HTML 代码转换为用户可见的视图的？浏览器又在何时进行 `repaint` 和 `reflow` 的操作？首先我们要先知道用于该操作的 `渲染树` 的由来。

## DOM Tree 的生成
> 浏览器首先会解析 HTML 代码，生成一颗 `DOM Tree`，DOM（文档对象模型）

简单地说，`DOM Tree` 的生成一般经历了四个阶段
* 转换——浏览器将从供应方（例如本地磁盘或服务器）获取到的 HTML 字节，根据 HTML 的文件编码格式转换为字符
* 符号化——浏览对转换好的字符串进行解析，将 `<>` 识别为对应的符号
* 词法分析——将符号化的字符串转换为 `对象`，一般来说是节点（Node）
* DOM 构建——对象生成完毕后，将根据对象之间的关系（父子、兄弟）生成 `DOM Tree`，在 `DOM Tree` 中可以确认 Node 节点间的关系


## CSSOM 的生成
> 在生成 `DOM Tree` 以后，将会生成 `CSSOM`（CSS 对象模型）的树形结构

与构建 `DOM` 的过程类似是，`CSSOM` 的构建过程也是读取 CSS 字节，进行转换解析，并生成对应的 `CSSOM Tree`，不同的是，`CSSOM` 为 CSS 样式服务，而 `DOM` 为节点服务

### CSSOM 能干什么？
CSSOM 通过复杂而具体的规则计算 CSS 样式，并将其映射到对应的需要样式的节点上，其遵循 `向下层叠` 的计算规则，例如下图

![向下层叠](http://7xoehm.com1.z0.glb.clouddn.com/cssom-tree.png)

可以看到，body 处使用了 `font-size: 16px`，根据 `向下层叠` 的规则，body 的子节点如果没有其自己的 `font-size` 规则，那么 body 的 `font-size` 规则将会层叠给该节点

### CSSOM 注意点
* `CSSOM` 的构建会阻塞页面的渲染——假设页面的呈现没有等待 `CSSOM` 的构建和计算，那么用户看到的将会是一堆没有样式的页面，等到 `CSSOM` 构建以后，页面又突然间变成有样式的页面，所以等待 `CSSOM` 的构建完成再进行渲染并呈现页面是必须的，不过如果 `CSSOM` 的构建的效率很低，那么将会出现常见的 `白屏` 现象。
* 只要重新加载页面，那么 `CSSOM` 也会重新构建——不管 CSS 样式文件是否被浏览器进行了缓存，`CSSOM` 是永远不会被缓存的，它会伴随页面的每一次重新加载而加载。
* JS 的运行所阻塞会被 `CSSOM` 的构建阻塞——在构建 `DOM` 时，遇见 `<script>` 标签时，浏览器会发出 HTTP 请求资源，并将控制权移交给 JavaScript 引擎，等待 JS 执行完毕归还控制权继续 `DOM` 的构建，然而，如果此时 `CSSOM` 未下载并构建完成，JS 的执行时机将被延迟

## Render Tree——渲染树
> `渲染树` 由 `DOM Tree` 和 `CSSOM Tree` 融合构成，渲染树与 `DOM Tree` 和 `CSSOM Tree` 不同，渲染树只包含需要渲染的节点信息，例如 `display: none` 的节点是不存在于 `渲染树` 内的

![合成渲染树](http://7xoehm.com1.z0.glb.clouddn.com/render-tree-construction.png)

可以发现，`span` 由于拥有 `display: none` 并未包含在 `渲染树` 里。

## 渲染树构建和渲染步骤
在这里我们简单说一下 `渲染树` 的构建和渲染步骤：
* 从 `DOM Tree` 根节点开始遍历所有可见节点
    * 不可见节点（如 `<script>` 和 `<meta>` 等）将不会包含在内，会被忽略
    * 通过 CSS 样式设置不可见的节点也不被包括，例如 `display: none`，不包括 `visibility: hidden` 等
* 从 `CSSOM Tree` 找到对应节点的规则，进行匹配
* 发射可见节点，连带其内容及计算的样式
* 根据生成的渲染树计算每个节点在屏幕中的绝对像素位置
* 根据计算结果开始渲染，这一步通常称为 `绘制` 或者 `栅格化`

# 重绘（repaint）和重排（reflow）
上面我们知道了 `渲染树` 由 `DOM Tree` 和 `CSSOM Tree` 融合构建而成，但是页面并不是进行一次渲染就可以适应各种节点的几何属性改变等变化的，页面会在某些时机进行 `重绘（repaint）` 和 `重排（reflow）`。

我们首先要知道其二者的区别：
* 重绘（repaint）——页面部分样式属性改变了（背景颜色，字体颜色等），但是几何属性没有改变，页面需要重绘该部分的内容，这就叫 `重绘（repaint）`
* 重排（reflow）——页面节点的几何属性改变，这时候需要重新计算元素的几何属性，重新构建 `渲染树`，这就叫 `重排（reflow）`

同时，我们要记得一下这句话：
> 重绘不一定导致重排，但是重排一定会导致重绘

## repaint 和 reflow 所带来的性能问题
通过前面的分析，可以预见的是，`repaint` 和 `reflow` 所需的性能消耗代价必然巨大，下面通过一个例子来说明：

HTML:
```html
<body>
    <div id="elem-a"></div>
    <div id="elem-b"></div>
    <div id="elem-c"></div>
    <div id="elem-d"></div>
</body>
```

JavaScript:
```javascript
// example 1
(function() {
    console.time("elem-a render time");
    for(let idx=0;idx<10000;idx++) {
        document.getElementById("elem-a").innerHTML += idx;
    }
    console.timeEnd("elem-a render time");
})();

// example 2
(function() {
    console.time("elem-b render time");
    let elemB = document.getElementById("elem-b");
    for(let idx=0;idx<10000;idx++) {
        elemB.innerHTML += idx;
    }
    console.timeEnd("elem-b render time");
})();

// example 3
(function() {
    console.time("elem-c render time");
    let str = "";
    for(let idx=0;idx<10000;idx++) {
        let elemC = document.getElementById("elem-c");
        str += idx;
    }
    document.getElementById("elem-c").innerHTML = str;
    console.timeEnd("elem-c render time");
})();

// example 4
(function() {
    console.time("elem-d render time");
    let elemD = document.getElementById("elem-d");
    let str = "";
    for(let idx=0;idx<10000;idx++) {
        str += idx;
    }
    elemD.innerHTML += str;
    console.timeEnd("elem-d render time");
})();
```

上面进行了 4 次试验，每次试验的内容不一样，下面进行分析：
* example 1——进行 10000 次的 “DOM 索引 + 重绘 + 重排”
* example 2——进行 10000 次的 “重绘 + 重排”，进行 1 次的 “DOM 索引”
* example 3——进行 10000 次的 “DOM 索引”，进行 1 次的 “重绘 + 重排”
* example 4——进行 10000 次的 “字符串拼接”，进行一次的 “DOM 索引 + 重绘 + 重排”

控制台打印结果如下：
* elem-a render time: 6020.826ms
* elem-b render time: 5797.140ms
* elem-c render time: 14.061ms
* elem-d render time: 3.905ms

由结果分析可知：
* `repaint` 和 `reflow` 消耗的性能是无比巨大的
* DOM 索引也消耗一定的性能，但是比起 `repaint` 和 `reflow` 简直是小巫见大巫
* 优化的重点在于减少 DOM 重复索引和循环引起的 `repaint` 和 `reflow`，尽量压缩为一次

## 如何减少 `repaint` 和 `reflow` 的发生？
要知道如何减少 `repaint` 和 `reflow` 的发生，我们就得先知道引起它们的原因， `repaint` 无疑就是改变 DOM 的背景颜色等导致，重点在于 `reflow` 的原因：
* 页面初始化必须进行一次的 `reflow`
* 缩放窗口
* 改变字体
* 添加或删除样式
* 添加或删除元素
* 内容改变，例如用户在输入框中输入文本
* 激活了伪类样式，例如：hover
* 脚本操作 DOM 并改变了其样式
* 计算 offsetWidth 和 offsetHeight
* 设置样式属性（width，height等）

以上种种都有可能引起页面的 `reflow`，而且不止这些，但是我们无法完全避免 `reflow`，所以我们必须想方设法去减少 `reflow`，例如：
* 减少单一操作样式属性，使用 class 一次性替换
* 对有动画的元素，使其 `position` 为 `fixed` 或 `absolute`，这样会减少元素间的影响
* 使用平滑的过渡动画，例如尽量少用 1 个像素的移动动画，可以改为 3 个像素，[具体原因](http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-performance-making-your-javascript-slow/#smooth)
* 避免使用 table 布局，[具体原因](http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-performance-making-your-javascript-slow/#tables)
* 减少在 CSS 样式中使用 JS 表达式
* 将元素 `display: none` 后再修改样式
* 创建一个新的节点元素，进行样式操作后替代原先的元素，不过可能会出现页面闪烁
* 创建 DocumentFragment 来进行更新

这里提一下浏览器自身对减少 `reflow` 的优化，下面例子：
```javascript
let elemA = document.getElementById("elem-a");

elemA.style.width = "100px";
elemA.style.height = "100px";
elemA.style.backgroundColor = "yellow";
```
以上例子浏览器只会一次性进行 `reflow` 而非 3 次

```javascript
let elemA = document.getElementById("elem-a");

elemA.style.width = "100px";
elemA.style.height = "100px";
elemA.getComputedStyle();
elemA.style.backgroundColor = "yellow";
```
以上例子浏览器会进行 2 次 `reflow`，因为中间需要获取当前的样式信息，浏览器必须先进行 `渲染树` 的重新计算，只要是获取以下样式信息的，都会引起浏览器立即重新渲染（如果必须则会 `reflow`）：
* offsetTop
* offsetLeft
* offsetWidth
* offsetHeight
* scrollTop
* scrollLeft
* scrollWidth
* scrollHeight
* clientTop
* clientLeft
* clientWidth
* clientHeight
* getComputedStyle()
* currentStyle（just for IE）

> 参考

* https://varvy.com/performance/cssom.html
* http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-performance-making-your-javascript-slow/
* https://developers.google.com/web/fundamentals/performance/critical-rendering-path/constructing-the-object-model?hl=zh-cn
* http://taligarsiel.com/Projects/howbrowserswork1.htm
* http://www.cnblogs.com/zichi/p/4720000.html
