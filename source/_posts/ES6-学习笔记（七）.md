---
title: ES6 学习笔记（七）
date: 2016-04-24 15:19:08
categories:
    - 前端
    - ECMAScript 6
tags: Javascript
---

# Generator
> `Generator` 函数是ES6提供的一种异步编程解决方案，语法行为与传统函数完全不同

## 声明

```JavaScript
// 这里的 * 只要在 function 和函数名之间使用就可以
function* iterator() {
    yield "x";
    yield "y";
    return "z";
}

let iter = iterator();

iter.next(); // {value: "x", done: false}
iter.next(); // {value: "y", done: false}
iter.next(); // {value: "z", done: true}
iter.next(); // {value: undefined, done: true}
```
调用 Generator 函数并不会如同普通函数一样返回函数的返回值，而是返回一个迭代器对象。`yield` 用于惰性执行 Generator 函数，当调用迭代器对象的 `next()` 时，就会执行函数内的代码知道遇到一个 `yield` 声明，并且其后面的值作为 `value` 的值返回，如果程序执行完毕，那么 `done` 是 `true`，反之是 `false`

## next()
`next()` 是 Generator 函数的步骤执行的调用者，同时可以利用它来为函数内部注入值

```JavaScript
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}

// 一开始 x 为 5，所以 yield 惰性求值返回为 6
var b = foo(5);
b.next() // { value:6, done:false }
// 这里相当于声明上一个 yield 的返回值是 12，所以 y 等于 2*12
b.next(12) // { value:8, done:false }
// 这里相当于声明上一个 yield 的返回值是 13，所以 z 等于 13
b.next(13) // { value:42, done:true }
```

## 使用 for...of
由于 Generator 函数调用返回一个迭代器对象 ，那么他就可以被所有支持迭代器遍历的特性使用，例如 for...of

```JavaScript
function *foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
// 由于当迭代器返回的 done 是 true 就停止遍历，所以 6 并不会被输出
```
除了 for...of，`...`  运算符和 `Array.from()` 同样支持 Generator

## Generator.prototype.throw()
> Generator函数返回的遍历器对象，都有一个throw方法，可以在函数体外抛出错误，然后在Generator函数体内捕获

```JavaScript
var g = function* () {
  try {
    yield;
  } catch (e) {
    console.log('内部捕获', e);
  }
};

var i = g();
i.next();

try {
  // 内部捕获后不再捕获此后的错误
  i.throw('a');
  // 内部不捕获，传递到外部
  i.throw('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 内部捕获 a
// 外部捕获 b
```

## Generator.prototype.return()
> Generator函数返回的遍历器对象，还有一个return方法，可以返回给定的值，并且终结遍历Generator函数

```JavaScript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```

## yield*
`yield*` 用于在 Generator 函数内调用另一个 Generator 函数

```JavaScript
function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  for (let v of foo()) {
    yield v;
  }
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// "x"
// "a"
// "b"
// "y"

// 另外一个例子
function* inner() {
  yield 'hello!';
}

function* outer1() {
  yield 'open';
  yield inner();
  yield 'close';
}

var gen = outer1()
gen.next().value // "open"
gen.next().value // 返回一个遍历器对象
gen.next().value // "close"

function* outer2() {
  yield 'open'
  yield* inner()
  yield 'close'
}

var gen = outer2()
gen.next().value // "open"
gen.next().value // "hello!"
gen.next().value // "close"
```

> Generator 函数的丰富超乎想象，这里写的笔记只是其冰山一角的特性，更多还是需要阅读相关书籍和资料
> 以上实例代码大部分来自阮一峰老师的 《ECMAScript 6 入门》 一书
> 书籍在线阅读地址: http://es6.ruanyifeng.com/#README
