---
title: 简单使用 require.js
date: 2016-03-28 13:25:44
tags: 模块化
categories: Javascript
---

# 前言

> `CommonJS` 和 `AMD` 是目前前端模块化的两个常用规范，此外，还有从 `sea.js` 引申出来的 `CMD` 规范，与 `AMD` 规范有异曲同工之妙，由于没有实际使用过，所以还是不赘述了。

## CommonJS

CommonJS 规范在 JavaScript 上的应用比较广泛的当属 Node.js。Node.js 是运行在 V8 引擎上的以 JavaScript 作为编程语言的服务端语言，模块化对服务端是非常重要的。美国程序员 `Ryan Dahl`（Node.js 创始人）在使用了 CommonJS 规范来对 Node.js 的模块进行规范管理，并且模块的加载时同步的。你可以像下面一样引入模块：

```Javascript
var express = require("express");
var app = express();

app.get("/something", function(req, res, next) {

    // doing something

})
```

以上代码引入了 `express` 框架模块，并且执行了 express，这样 `app` 这个变量就得到了执行后的一个实例，并可以使用其中的各种函数方法。

## 为什么使用 AMD？

既然有了 CommonJS 规范来进行 JavaScript 的模块化管理，为什么还会出现 AMD 规范？原因很简单，环境不同，在服务端环境中，由于模块文件都存在于服务器中，所以请求模块的时间几乎可以忽略，而且请求模块的次数大多数只请求一次就可以从服务器开启一直使用到服务器重启。而在客户端环境中，每次刷新页面都会重新请求 JavaScript 文件，并且，CommonJS 是同步加载模块文件，所谓同步，就是必须等到 JavaScript 模块文件请求并加载完毕才会继续往下执行程序，在客户端页面中，如果一个 JavaScript 模块文件请求和加载的时间很长，那么页面的渲染和后续的程序执行将会一直被阻塞。所以，客户端的模块化管理需要一种 `非阻塞式` 和 `异步` 的加载方式，而 AMD 规范就是这么一种规范。

## require.js

`require.js` 是对 AMD 规范的一种实现和执行，此外，require.js 也可以使用 CommonJS 风格的方式。

### 引入

```Javascript
<script data-main="javascripts/main.js" src="javascripts/require.js"></script>
```

以上，引入了 require.js 的同时，使用 `data-main` 指定了最早加载的一个模块，require.js 里的 `baseUrl` 同时默认被设置为 `javascripts`，如果此后没有显式地使用 require.config() 对 baseUrl 和 paths 进行设置，那么所有依赖模块的基础路径都将会以该 baseUrl 路径作为基础路径进行搜索。

### require.config(options)

```Javascript
require.config({

    baseUrl: "./libs",

    paths: {
        jquery: "jquery-2.2.2",
        underscore: "/underscore",
        backbone: "http://backbonejs.org/backbone"
    }
})
```

以上代码中，我们显式地设置了模块搜索路径的 baseUrl 为 `./libs`，同时在 paths 中，我们指定了 `jquery`，`underscore` 和 `backbone` 的路径，这里使用了三种不同的路径定义方式，会有不同的搜索结果。假设我们当前页面的路径为 `http://localhost`，那么三个模块文件的路径如下：

* jquery 的路径为 `baseUrl + paths路径 = http://localhost/libs/jquery-2.2.2.js`

* underscore 的路径为 `当前路径 + paths路径 = http://localhost/underscore.js`

* backbone 的路径为 `paths 路径 = http://backbonejs.org/backbone.js`

这里可以发现，所有 JavaScript 的后缀 `.js` 都不能加上去，不然会请求类似 `underscore.js.js` 这样的错误文件。以上就是模块文件路径指定的多种不同方式所产生的不同结果。

### define(name, deps, callback)

require.js 使用 `define(name, deps, callback)` 来定义模块，如下代码所示：

```Javascript
// first
define(function() {

    // do somthing

    return {
        // return a key-value object
        api: "myApi"
    };
});

// second
define(["jquery", "underscore"], function($, _) {

    // do something

    return ……;
});

// third
define("foo/user", ["jquery", "underscore"], function($, _) {

    // do something

    return ……;
});
```

* 第一种方法是简单的定义模块，这里可以在 callback 里直接定义一些东西，使用 return 返回一个对象字面量

* 第二种方法首先定义了该模块的依赖模块 jquery 和 underscore，然后定义一个 callback函数，当 jquery 和 underscore加载完成后，其暴露的接口将会依次传给 callback 的 arguments

* 第三种方法首先定义了该模块的名称，后面的如第二种方法一样，但是不推荐自定义模块名称，如果你移动了该模块的路径，那么就得重命名

### require(deps, callback)

require.js 使用 `require(deps, callback)` 来请求加载模块，如下代码所示：

```Javascript
require(["jquery"], function($) {

    // do something
    // never need return anything
});
```

require() 和 define() 的使用上很相似，但是 require() 用来加载，define() 用来定义，区别是前者不需要返回一个对象，后者需要返回一个对象来代表该模块的接口。


### CommonJS 风格的 require.js

```Javascript
// 可以解决循环依赖
define(function(require, exports, module) {

    var base = require("./base");

    exports.a = function() {
        return base;
    };
});

// 就近依赖
define(["require", "jquery"], function(require) {

    var $ = require("jquery");

    return function() {
        return $;
    }
});

// 定义模块并暴露接口
define(["exports", "a"], function(exports, a) {

    exports.bar = function() {
        return a.bar();
    };
});
```

> 以上三种 CommonJS 风格的定义模块方法，更详细的请参考 http://www.requirejs.cn/

> 参考来源: http://www.ruanyifeng.com/blog/2012/11/require_js.html
