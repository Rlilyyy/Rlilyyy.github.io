---
title: DAY4 Backbone.js API阅读（二）
date: 2016-03-22 12:52:00
tags: Oxygen
categories: 项目
---

## Collection
> Backbone.js 的 C 层不同于通常 MVC 架构的 `Controller` 层，而是 `Collection` 层，Collection 层不仅负责与数据库的交互和 Model 的数据交互，同时也是 Model 的一个有序集合。当集合中的 Model 发生变化时，将会触发该集合的 `change` 事件，当使用 `fetch()` 函数时，可能会触发 `add` 或者 `remove` 事件。而一些在 Model 上触发的事件可能也会在 Collection 中触发。

* extend——用于创建一个新的 Collection 集合并对其进行相应扩充和定义，如下定义：
```JavaScript
var books = Backbone.Collection.extend({
    model: Book,
    ……
})
```
* model——用于定义该集合中的 Model 类型
* add——向 Collection 添加一个或多个 Model，同时会触发 `add` 事件
* remove——从 Collection 移除一个或多个 Model，同时会触发 `remove` 事件
* reset——当有大量 Model 需要更改的时候，这时候使用 reset 插入一个模型组，同时触发 `reset` 事件，但是不会触发 `add` 和 `remove` 事件
* set——通过传入一组 Model，如果传入的 Model 在 Collection 已经存在，那么将会合并，如果不存在，那么将会添加，如果 Collection 中存在传入 Model 组中不存在的 Model，那么该 Model 将会被删除。以上将会触发 `add`，`remove`，`change` 事件
* fetch——从服务端拉取更新数据，并使用 `reset` 到 Collection 中。
* comparator——设置 Collection 排序的依据，例如：
```JavaScript
var Chapter  = Backbone.Model;
var chapters = new Backbone.Collection;

chapters.comparator = 'page';

chapters.add(new Chapter({page: 9, title: "The End"}));
chapters.add(new Chapter({page: 5, title: "The Middle"}));
chapters.add(new Chapter({page: 1, title: "The Beginning"}));

alert(chapters.pluck('title'));
```
* Collection 同时提供了多个 Underscore.js 的函数，例如：
```JavaScript
books.each(function(book) {
    // doing something
})
```
> `reset` 用于重置数据，即将原有数据删除，添加传入的新数据，而 `set` 则更新数据，只有当原先集合中的某个 Model 不存在于新传入的数据中时，该 Model 才会被删除，否则只是合并数据。总结就是 reset 是重置，set 是更新。

## View
>View 用来处理视图的渲染以及视图的更新，我们可以自定义一个 `render` 函数来定义当模型数据发生变化时（render 可以绑定在自身模型的 change 事件上）如何渲染视图。

* 一般地，我们如下所示去扩充一个自定义 View：
```JavaScript
var DocumentRow = Backbone.View.extend({

  tagName: "li",

  className: "document-row",

  events: {
    "click .icon":          "open",
    "click .button.edit":   "openEditDialog",
    "click .button.delete": "destroy"
  },

  initialize: function() {
    this.listenTo(this.model, "change", this.render);
  },

  render: function() {
    ...
  }

});
```
* tagName——代表 View 的标签类型，设置以后同时也会设置 View 的 `el`
* className——设置 View 的样式类
* events——在该视图中绑定相应的事件
* initialize——在 View 被实例化时执行
* render——默认无任何操作，我们可以重载该函数，定义如何渲染视图，将其绑定到 Model 的 change 事件，这样当 Model 发生变化时，可以立即调用 render 函数更新视图。
* el——引用了该 View 的 dom 元素
* $el——通过引用 el，对其进行封装，其可以使用 JQuery 的函数
* setElement——将 View 从旧的引用 dom 对象切换到新的 dom 对象引用，视图的事件委托和 $el 也响应迁移
* template——利用 Underscore 的 template 方法可以为 View 设定一个模板，有两种方法，如下所示：
```JavaScript
// first
var LibraryView = Backbone.View.extend({
  template: _.template(...)
});

// second
<script id="teamTemplate" type="text/template">
    <%= name %>
</script>

App.Views.Team = Backbone.View.extend({
    className : '.team-element',
    tagName : 'div',
    model : new App.Models.Team
    render : function() {
        // Compile the template
        var compiledTemplate = _.template($('#teamTemplate').html());
        // Model attributes loaded into the template. Template is
        // appended to the DOM element referred by the el attribute
        $(this.el).html(compiledTemplate(this.model.toJSON()));
    }
});
```
>明天继续看 Router 的使用，今晚参加了网易的笔试，还是有点累。
