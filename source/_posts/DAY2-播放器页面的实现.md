---
title: DAY2 播放器页面的实现
date: 2016-03-20 21:51:14
tags: Oxygen
categories: 项目
---
> 早上8点多起床转角遇到爱——一只好大的蜘蛛，面对面吓蒙逼了。下午将播放器的图给切出来，主要还是微调一下样式比较麻烦，测试了一下 IE 9+ 都是正常显示的，IE 8 不支持 `border-radius` 和 `box-shadow` 就没办法啦，`rem` 也不能用，但是这个能用正常的 `px` 解决。

## 最终页面效果图
![](http://7xoehm.com1.z0.glb.clouddn.com/github%E7%AC%AC%E4%BA%8C%E7%89%88.png)

## 实现过程
* 进度条的布局，典型的双飞翼布局方式，左右固定了 60px 作为左右两个时间的空间
* 进度条的实现，用了三个 div 分别代表 `当前进度`，`加载进度`，`进度总长`
* 歌曲的封面，为了实现宽度固定，焦点居中，使用 div 的 `background` 加载封面，`background-position` 可以实现焦点居中
* 歌词的显示，使用 `<ul>` 可以实现，后面也方便进行滚动
* 控制器的样式，引入了 `font-awesome` 作为图标
* 音量调节的弹出，使用 CSS 的 `:hover` 进行一个样式优先级转换，还是尽量避免使用 JS 实现效果
* 一些字体的凹凸感，使用了 `text-shadow` 实现，阴影往下偏移1个像素，发散一个像素，颜色使用比字体颜色偏浅的黑色


> [demo查看](http://www.libinhong.com:8080/index)
