---
title: DAY3 Backbone.js API阅读
date: 2016-03-21 14:04:56
tags: Oxygen
categories: 项目
---
> 原来机房看 API 的学习效率奇低，今天在机房上一天课，下午5点半下个脚都是麻的，今天主要还是熟悉一下 `Backbone.js` 的API，下面看几个比较重要的 API，也算是对今天学习的一个备忘录。官方文档写得有点简单了，后来看官方提供的 `Todos` 的例子的源码加上一边看官方文档才渐渐明白运行机制。

# Backbone.js

`Backbone.js` 强制依赖 `Underscore.js`，其很多实现方式都利用了 Underscore 的 API，由于之前通读过 Underscore 的源代码，所以这个并没有太大问题。此外，Backbone.js 还半依赖 `JQuery.js`，主要是 Backbone 里的一个 `save` 函数当执行时会执行 `Backbone.sync` 从而需要使用 `ajax` 访问服务器进行 `Model` 的保存，当然，可以通过重载 `Backbone.sync` 来使用其他方式保存 Model，例如[Backbone.localStorage.js](https://github.com/Rlilyyy/Backbone.localStorage)就是一个用于重载 sync 的库。

## Model

Model 用于处理 `数据交互`和 `逻辑`，一般地，使用 Backbone.Model.extend(properties, [classProperties]) 创建一个新的模型，properties 为一个对象实例，里面包含多个属性：
* defaults——用于设置一些初始的模型属性，如下面代码所示：
```JavaScript
var Meal = Backbone.Model.extend({
  defaults: {
    "appetizer":  "caesar salad",
    "entree":     "ravioli",
    "dessert":    "cheesecake"
  }
});
```
* initialize——在模型创建后执行的函数
* save——将模型的 attributes 模型状态保存到持久层
* validate——用于验证调用 save 以后传入的 attributes 的值的合法性，如果有返回数据，代表验证失败，save 就不会成功保存模型，如果在 set 中传入 `{validate:true}`，那么会执行 validate 验证。
* fetch——从服务器更新模型数据，如果数据有更新，则会触发 `change` 事件，同时接受 `success` 和 `error` 回调
* previous——在该模型的 change 事件中，可以通过此函数获取上一次更改的值
* previousAttributes——同样用过此函数获取上一次更改的属性散列副本
* 可以重载 Model 中的函数，如 `get`，`set`
