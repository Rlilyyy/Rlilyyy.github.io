---
title: 'DAY5 完成 & 总结'
date: 2016-03-27 16:20:18
tags: Oxygen
categories: 项目
---
## 最终效果图
![](http://7xoehm.com1.z0.glb.clouddn.com/githubzuizhong.png)

## 演示地址
> http://www.libinhong.com:90/

## 踩到的坑

* Backbone 的 View 监听本身 Model 的 `change` 事件，仅在 Model 的属性实际发生变化才会被触发，如果 set 一个相同的值进 Model，实际上并不会触发事件

* Backbone 的 View 中的 `el` 代表了该 View 对应的 dom 节点，同时，在该 View 中定义的事件所对应的 dom 只能处于该 el 下，否则无法正确找到 dom 并添加事件

* Backbone 的 `events` 无法绑定 `audio` 的事件，估计是因为 JQuery 无相对应的事件，所以只能自己手动绑定事件

## 总结

* 最终没有使用 `require.js`

* 在使用 Backbone 的过程中，V 层聚集了大量的渲染页面的逻辑，C 层则更多的提供了集合的功能，并且向服务器 fetch 数据，M 层则是模型层，没有涉及逻辑。

* 对 MVC 结构有了更深的了解，对分层的好处也有了体会

* 编码期间不断对功能性函数的重构是很重要的，即使是两行代码也有可能需要独立为一个功能函数，到了后期发现每一个独立的功能函数都能应用到对应的地方，避免了后期代码行的重复

## 吐槽
> Oxygen 从 20 号开始写，从 需求 —— 原型 —— 实现 一步步走下来一共经历 6 天，实际工作时间不会超过 5 天，中间夹杂着各种上课、做实验、做作业，这次算是对前面一个阶段学习的总结。第一次尝试了 Backbone.js 和 JQuery.js，使用手段颇为生疏，希望以后有机会再次熟悉一下各种 API 的使用。最后吐槽一下接下来十几周都要面对 JavaEE 和 JSP 和 SSH 的课程的痛苦……
