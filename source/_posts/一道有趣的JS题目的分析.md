---
title: 一道有趣的JS题目的分析
date: 2016-03-18 14:08:29
categories: 前端
tags: Javascript
---
今天在微博看到有人发了一道题目，代码如下：

```javascript
function Foo() {
    getName = function() {console.log(1);}
    return this;
}
Foo.getName = function() {console.log(2);};
Foo.prototype.getName = function() {console.log(3);};
var getName = function() {console.log(4);};
function getName() {console.log(5);};

Foo.getName();//2
getName();//4
Foo().getName();//1
getName();//1
new Foo.getName();//2
new Foo().getName();//3
new new Foo().getName();//3

Foo.getName()，这里调用的方法是第5行代码的函数，这个没什么疑问。
```

`getName()`，这里调用的是第 7 行的函数，有人要问了，为什么不是第 8 行，这里涉及到JS解析器中一个`function declaration hoisting(函数声明提升)`的过程，在 JS 代码中，使用`var x = function(){}`叫作`函数表达式`，使用`function x(){}`叫作`函数声明`，在 JS 代码加载的过程中，函数声明会在第一时间加载到源代码树的顶部，所以即使你调用使用函数声明的函数比该函数的声明还早，也是无异常的。反之，函数表达式则要在 JS 加载到该行代码时，函数才算是真正可以使用。所以这道题里，第 8 行比第 7 行的函数声明早，所以第7行的函数表达式覆盖了第 8 行的函数内容。

`Foo().getName()`，这里执行 Foo() 后返回的 this 是 window，即全局对象，而 Foo() 内部的`getName`由于没有使用 var 去声明，故默认为 window 的 getName 属性，所以又再一次覆盖了原先的 getName()，所以打印是1。

`getName()`，由于上一行代码的执行已经导致了 window 下的 getName() 被覆盖，所以依旧打印1。

`new Foo.getName()`，这里 new 的是 getName() 而不是 Foo()，所以打印了2，与第 10 行的调用方法类似，不过这里还会返回一个 Foo.getName 的实例。

`new Foo().getName()`，这里首先 new Foo() 返回 Foo() 的实例对象，这里注意，构造函数中如果没有指明 return，那么默认返回构造函数的实例，如果指定了 return，那么返回指定的 return 的值（只能是引用类型的值，如{},[]等），这里指定了 return this，this 在 new 的情况下是实例本身，所以跟默认情况下是一致的，然后又调用了 getName()，这里调用的是原型中的函数，所以打印3。

`new new Foo().getName()`，先不看第一个 new，则后面的步骤跟上面一致，不过到了调用函数的时候，第一个 new 发生作用了，实例化了 getName()，不过这里的 getName() 依旧是原型中的函数，只不过将他实例化了，所以依旧打印3。
