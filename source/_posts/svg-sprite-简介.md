---
title: svg sprite 简介
date: 2016-10-24 22:14:25
categories:
    - 前端
    - SVG
tags: 小技巧
---
# 前言
&emsp;&emsp;首页的高速加载和渲染一直是前端开发者们津津乐道的事情，因此各种技术也应运而生。在 HTTP1.1 时代，为了减少请求的发送，加快首页加载，压缩和合并成了必不可少的技术，其中包括了 JavaScript 文件的压缩、混淆和合并，还有 CSS 文件的压缩和合并，最后还有一个是针对小图片的请求优化，也就是 `CSS Sprite`，也叫 `雪碧图` 或 `CSS 精灵`。其大概的思想就是将多个小图片按照一定的尺寸和位置排列好，然后合成一张图片，最后用户访问页面时，只要请求这一张合成图，而开发者利用 `background-position` 等属性控制显示合成图某个位置的图片，即可达到一张图片多个图标的效果，同时也将请求数量压缩为一个。当然，这次我们要说的并不是这个技术，而是与之运用思想类似的 `SVG Sprite`。

# SVG Sprite
&emsp;&emsp;`SVG Sprite` 使用 `<symbol>` 标签来定义一个图形模板对象，好处在于其可以重复利用，我们可以看一下 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/symbol) 中对 `<symbol>` 的定义：
> symbol元素用来定义一个图形模板对象，它可以用一个<use>元素实例化。symbol元素对图形的作用是在同一文档中多次使用，添加结构和语义。结构丰富的文档可以更生动地呈现出来，类似讲演稿或盲文，从而提升了可访问性。注意，一个symbol元素本身是不呈现的。只有symbol元素的实例（亦即，一个引用了symbol的 <use>元素）才能呈现。

&emsp;&emsp;可以看到，`<symbol>` 定义的图形并不会第一时间显示出来，只有使用了 `<use>` 标签进行实例化以后才会显现。[MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/use) 中对其做出了如下定义：
> use元素在SVG文档内取得目标节点，并在别的地方复制它们。它的效果等同于这些节点被深克隆到一个不可见的DOM中，然后将其粘贴到use元素的位置，很像HTML5中的克隆模板元素。

&emsp;&emsp;而要使用 `<use>` 来实例化一个 `svg图形模板对象`，则要使用其中的 `xlink:href` 属性，在我们处理好的 `<symbol>` 上都会带有一个 `id`，如下所示（伪代码）：

```html
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" style="position:absolute;width:0;height:0;visibility:hidden">
  <defs>
    <symbol id="icon1">...</symbol>
    <symbol id="icon2">...</symbol>
    <symbol id="icon3">...</symbol>
  </defs>
</svg>
```

&emsp;&emsp;根据每一个 `<symbol>` 的 `id`，我们可以使用 `<use>` 根据这些 `id` 来使用 `svg`，如下所示：

```html
<div class="icons">
  <svg><use xlink:href="#icon1"/></svg>
  <svg><use xlink:href="#icon2"/></svg>
  <svg><use xlink:href="#icon3"/></svg>
</div>
```

&emsp;&emsp;`SVG Sprite` 的基本原理就是运用这些元素，相比较 `CSS Sprite`，`SVG Sprite` 显得更为友好，不用多余的例如 `background-position` 属性来控制位置。

# 兼容性
&emsp;&emsp;运用一个技术的前提都是其兼容性满足项目的最低要求，或者在不兼容的情况下有相对应的替代方案。在饿了么 Web 的兼容性要求为 PC 端 IE9+，安卓移动端 4.4+，IOS7+，具体可以看 ElemeFE 的 [style-guide](https://github.com/ElemeFE/style-guide/blob/master/compatibility-web.md)。下面是在 [caniuse](http://caniuse.com/#search=SVG) 上的结果：
![SVG-兼容性](http://7xoehm.com1.z0.glb.clouddn.com/svg-%E5%85%BC%E5%AE%B9%E6%80%A7.jpeg)

&emsp;&emsp;可以看到兼容性在 PC 端上是完全没有问题的，而在移动端上也能支持，所以可以安心地用起来了。

# 结合 Webpack 使用 `SVG Sprite`
&emsp;&emsp;我们使用 `Webpack` 来对多个分离的 SVG 文件进行自动化处理为合并好的多个 `<symbol>`，并插入到 `<body>` 顶部。首先我们要对 `webpack.config.js` 进行配置。

首先看一下基本的文件目录结构：
```bash
├── LICENSE
├── README.md
├── css
│   └── index.css
├── dist
│   ├── app.css
│   ├── app.js
│   └── index.html
├── js
│   └── index.js
├── package.json
├── static
│   ├── analytics.svg
│   ├── archives.svg
│   ├── businessman.svg
│   ├── businessmen.svg
│   ├── certificate.svg
│   ├── chat.svg
│   └── contract.svg
├── template
│   └── index.html
└── webpack.config.js
```

webpack.config.js
```javascript
const path = require('path');
const webpack = require('webpack');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

var plugins = [
  new HtmlWebpackPlugin({
    filename: './index.html',
    template: './template/index.html'
  })
];

plugins.push(
  new ExtractTextPlugin('[name].css')
);

module.exports = {
  entry: {
    app: './js/index.js',
  },

  output: {
    path: './dist',
    publicPath: './',
    // filename: '[name].[chunkhash:6].js'
    filename: '[name].js'
  },

  resolve: {
    extensions: [ '', '.js' ]
  },

  module: {
    loaders: [
      { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader' },
      { test: /\.css$/, loader: ExtractTextPlugin.extract('style', 'css') },
      { test: /\.svg$/, loaders: [ 'svg-sprite-loader', 'svgo-loader?useConfig=svgoConfig' ] },
      { test: /\.(gif|png|jpg|ttf|woff2|woff|eot)$/, loader: 'url?limit=1000&name=[path][name].[hash:6].[ext]' }
    ]
  },

  svgoConfig: {
    plugins: [
      { removeTitle: true },
      { convertColors: { shorthex: true } },
      { convertPathData: true },
      { cleanupAttrs: true },
      { removeComments: true },
      { removeDesc: true },
      { removeUselessDefs: true },
      { removeEmptyAttrs: true },
      { removeHiddenElems: true },
      { removeEmptyText: true }
    ]
  },
  plugins: plugins
};
```

&emsp;&emsp;可以看到，配置文件中使用了 `svg-sprite-loader` 和 `svgo-loader` 对 `svg` 文件进行处理，`svg-sprite-loader` 的作用就是将多个 `svg` 文件合并为一个 `<svg>` 元素。至于 `svgo-loader`，作用是将 `<svg>` 中一些无用的信息过滤去除，精简结构，详细配置可以自行查阅对应的文档说明，可以根据实际需求进行过滤。接下来将列出 `index.css`, `index.js`, `index.html` 的内容：

index.css
```css
* {
  box-sizing: border-box;
}

.icons svg {
  width: 100px;
  height: 100px;
}
```

index.js
```javascript
import indexStyle from '../css/index.css';
import analytics from '../static/analytics.svg'
import archives from '../static/archives.svg'
import businessman from '../static/businessman.svg'
import businessmen from '../static/businessmen.svg'
import certificate from '../static/certificate.svg'
import chat from '../static/chat.svg'
import contract from '../static/contract.svg'

console.log('demo complete');
```

index.html
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>svg-sprites-demo</title>
  </head>
  <body>
    <div class="icons">
      <svg><use xlink:href="#analytics"/></svg>
      <svg><use xlink:href="#archives"/></svg>
      <svg><use xlink:href="#businessman"/></svg>
      <svg><use xlink:href="#businessmen"/></svg>
      <svg><use xlink:href="#certificate"/></svg>
      <svg><use xlink:href="#chat"/></svg>
      <svg><use xlink:href="#contract"/></svg>
    </div>
  </body>
</html>
```

&emsp;&emsp;以上就是主要的一些配置和内容，如果需要完整的项目，可以到我的 [github](https://github.com/Rlilyyy/svg-sprites-demo) 下 clone 项目到你的本地进行构建。下面是构建后的网页效果和结构：

![效果](http://7xoehm.com1.z0.glb.clouddn.com/%E6%95%88%E6%9E%9C.jpeg)

# 优点 & 缺点
优点：
* 将多个请求压缩为无请求
* svg 对比 image 其屏幕适应性更好，任何分辨率都能达到高清效果
* svg 体积更小
* 每一个 `<symbol>` 都可以重复利用

缺点：
* svg 不利于变动性大的图片，例如需要经常修改颜色
* 兼容性对于需要兼容 IE8- 的网站不好，需要对低版本浏览器有替代方案

参考资料：
> https://developer.mozilla.org/en-US/docs/Web/SVG/Element/svg
https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/symbol
https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/defs
https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/use
