---
title: clearfix 闭合浮动
date: 2016-03-18 13:57:45
categories: 前端
tags: 布局
---
```css
/** *  For modern browsers */

.clearfix:before, .clearfix:after {
    content: "";
    display: table;
}

.clearfix:after {
    clear: both;
}

/** * For IE 6/7 (trigger hasLayout) */
.clearfix {
    *zoom: 1;
}
```

`.clearfix:after {content:"."; display:block; height:0; visibility:hidden; clear:both; } `
这里也是一种方法，`content`为一个`.`的原因是在 Firefox 里，空的 content 会有额外间隙，所以使用`.`，此外的`height`等属性只是为了隐藏 content 不影响布局。而最新的方法 `display: table` 的 content 可以为空，且不会产生间隙，所以额外的属性都可以去除。

在父元素使用`overflow`清除浮动的原理是，将父元素变为一个新的`BFC`，根据 BFC 的特点，BFC 可以包含浮动元素，所以，不仅仅是 overflow 能清除浮动，只要能触发父元素的BFC的方式都可以清除浮动，例如`display: table-cell`,`float:left`等等都可以触发BFC。

传统的加一个空的div去清除浮动的方法有一个缺点，当子元素是浮动元素且有外边距是，清除浮动以后，父元素的高度仅仅是包裹子元素的高度（不包括外边距的高度），那么子元素明显会被边距顶出了父元素外边，虽然父元素会被撑开一段高度，但是由于子元素外边距的存在，父元素无法完全包裹子元素。

关于BFC的知识，可浏览
> http://www.cnblogs.com/leejersey/p/3991400.html

> http://kayosite.com/remove-floating-style-in-detail.html
