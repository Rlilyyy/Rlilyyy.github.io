---
title: ES6 学习笔记（三）
date: 2016-04-18 16:21:39
categories:
    - 前端
    - ECMAScript 6
tags: Javascript
---

# Symbol
> `Symbol` 是 JavaScript 的第七种类型（前六种是：Undefined、Null、布尔值（Boolean）、字符串（String）、数值（Number）、对象（Object））。它用于生成独一无二的变量名，防止相同变量名的相互覆盖，例如在对象中，用字符串声明的属性如果相同，那么会互相覆盖

注意点：
* Symbol 不能与其他值一起计算，例如与字符串连接，但是可以使用 `toString()` 进行转换
* Symbol 不能使用点运算符作为属性名，只能使用方括号，避免与字符串方式冲突
* Symbol 作为属性名，该属性不会出现在 for...in、for...of 循环中，也不会被 Object.keys()、Object.getOwnPropertyNames() 返回
* Symbol 可以被 Object.getOwnPropertySymbols() 返回


```JavaScript
var s1 = Symbol('foo');
var s2 = Symbol('bar');
var s3 = Symbol();
var s4 = Symbol();
var s5 = Symbol('foo');

s1 // Symbol(foo)
s2 // Symbol(bar)

s3 === s4; // false
s1 === s5; // false

s1.toString() // "Symbol(foo)"
s2.toString() // "Symbol(bar)"

var mySymbol = Symbol();
var a = {};

a.mySymbol = 'Hello!';
a[mySymbol] // undefined
a['mySymbol'] // "Hello!"
```

## Symbol.for() & Symbol.keyFor()
> `Symbol.for(name)` 与 `Symbol(name)` 不同的地方在于，前者在没有存在 name 注册的 Symbol 变量时，会注册一个新的 Symbol 变量，如果存在，那么就会返回该 name 注册的 Symbol。后者不管 name 有没有注册过，都会产生一个新的 Symbol

```JavaScript
var s1 = Symbol.for('foo');
var s2 = Symbol.for('foo');

s1 === s2 // true

Symbol.for("bar") === Symbol.for("bar")
// true

Symbol("bar") === Symbol("bar")
// false
```

> `Symbol.keyFor(Symbol)` 会返回已注册过的 Symbol 的值的 desc

```JavaScript
var s1 = Symbol.for("foo");
Symbol.keyFor(s1) // "foo"

var s2 = Symbol("foo");
Symbol.keyFor(s2) // undefined
```

# Proxy
> Proxy用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种“元编程”（meta programming），即对编程语言进行编程

## new Proxy(target, handler)
> target 为要通过 handler 进行处理拦截的对象

## get()
> 用于拦截对对象属性访问时的处理

```JavaScript
var person = {
  name: "张三"
};

var proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError("Property \"" + property + "\" does not exist.");
    }
  }
});

proxy.name // "张三"
proxy.age // 抛出一个错误

// 对数组负数的索引值进行处理
function createArray(...elements) {
  let handler = {
    get(target, propKey, receiver) {
      let index = Number(propKey);
      if (index < 0) {
        propKey = String(target.length + index);
      }
      return Reflect.get(target, propKey, receiver);
    }
  };

  let target = [];
  target.push(...elements);
  return new Proxy(target, handler);
}

let arr = createArray('a', 'b', 'c');
arr[-1] // c
```

## set()
> 用于拦截设置属性时的操作

```JavaScript
let validator = {
  set: function(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }

    // 对于age以外的属性，直接保存
    obj[prop] = value;
  }
};

let person = new Proxy({}, validator);

person.age = 100;

person.age // 100
person.age = 'young' // 报错
person.age = 300 // 报错
```

## apply(target, context, args)
> 用于拦截函数的调用、call和apply操作。target 为目标函数，context 为函数执行上下文，args 为函数参数

```JavaScript
var target = function () { return 'I am the target'; };
var handler = {
  apply: function () {
    return 'I am the proxy';
  }
};

var p = new Proxy(target, handler);

p()
// "I am the proxy"
```

## has()
> 用于处理一些在执行 “xxx in obj” 的拦截，可以对某些属性进行隐藏

```JavaScript
var handler = {
  has (target, key) {
    if (key[0] === '_') {
      return false;
    }
    return key in target;
  }
};
var target = { _prop: 'foo', prop: 'foo' };
var proxy = new Proxy(target, handler);
'_prop' in proxy // false
```

## construct()
> 用于拦截 new 操作时的处理

```JavaScript
var p = new Proxy(function() {}, {
  construct: function(target, args) {
    console.log('called: ' + args.join(', '));
    return { value: args[0] * 10 };
  }
});

new p(1).value
// "called: 1"
// 10
```

## deleteProperty()
> 用于拦截删除对象属性时的操作

```JavaScript
var handler = {
  deleteProperty (target, key) {
    invariant(key, 'delete');
    return true;
  }
}
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}

var target = { _prop: 'foo' }
var proxy = new Proxy(target, handler)
delete proxy._prop; // 报错
```

## defineProperty()
> 用于拦截定义对象属性时的操作

```JavaScript
var handler = {
  defineProperty (target, key, descriptor) {
    return false
  }
}
var target = {}
var proxy = new Proxy(target, handler)
proxy.foo = 'bar';
console.log(proxy.foo); // undefined
```

## enumerate()
> 用于拦截 “for xxx in obj” 时的处理操作，跟 has() 不是同种拦截，互不影响

```JavaScript
// 这个例子在 FF 上成功，但是 Chrome 上依旧打印了所有属性
var handler = {
  enumerate (target) {
    return Object.keys(target).filter(key => key[0] !== '_')[Symbol.iterator]();
  }
}
var target = { prop: 'foo', _bar: 'baz', _prop: 'foo' }
var proxy = new Proxy(target, handler)
for (let key in proxy) {
  console.log(key);
  // "prop"
}
```

# Reflect
> Reflect 对象提供了若干个能对任意对象进行某种特定的可拦截操作（interceptable operation）的方法，它所提供的静态方法在 Proxy 中都有对应的相同方法，可以这么说，Reflect 保留了原先的默认处理方式，以供 Proxy 在修改默认处理的同时也能应用默认处理后再进行一些其他处理。某些方法也是 Object 上的方法，不过修改相应的返回值，更加完善
> 参考资料：
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect

```JavaScript
// 老写法
try {
  Object.defineProperty(target, property, attributes);
  // success
} catch (e) {
  // failure
}

// 新写法
if (Reflect.defineProperty(target, property, attributes)) {
  // success
} else {
  // failure
}

// 老写法
'assign' in Object // true

// 新写法
// 行为被函数化
Reflect.has(Object, 'assign') // true

// 添加了自己的行为，同时使用 Reflect 的相应静态方法执行默认行为
var loggedObj = new Proxy(obj, {
  get(target, name) {
    console.log('get', target, name);
    return Reflect.get(target, name);
  },
  deleteProperty(target, name) {
    console.log('delete' + name);
    return Reflect.deleteProperty(target, name);
  },
  has(target, name) {
    console.log('has' + name);
    return Reflect.has(target, name);
  }
});
```
> 可能有人会对 Reflect 的存在感到疑惑，我个人认为 Reflect 存在的意义大概有两点，一是对 ES5 中 Object 一些方法的返回值进行修改完善，但是为了兼容性，开拓了一个新的对象来存放这些静态方法。二是为了让 Proxy 可以方便地在拦截器中添加处理方法的同时调用默认处理方法，所以 Reflect 中存在的静态方法都跟 Proxy 中的方法一一对应，同时修改了部分 Object 中的方法的返回值，例如 Object 中一些调用失败的方法会抛出错误，而在 Reflect 中修改为返回布尔值，以上是一点点见解，可以参考一下：https://tc39.github.io/ecma262/#sec-reflect-object

> 以上实例代码大部分来自阮一峰老师的 《ECMAScript 6 入门》 一书
> 书籍在线阅读地址: http://es6.ruanyifeng.com/#README
