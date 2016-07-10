---
title: ES6 学习笔记（二）
date: 2016-04-16 22:55:15
categories:
    - 前端
    - ECMAScript 6
tags: Javascript
---

# 函数的扩展

## 扩展符号...
> ES6 新增了符号 `...`，该符号类似于剩余函数的逆运算，能将一个数组变为一组参数传入函数中，只要实现了 `Iterator` 接口的对象，都可以使用该符号转换数组

```JavaScript
function push(array, ...items) {
  array.push(...items);
}

function add(x, y) {
  return x + y;
}

var numbers = [4, 38];
add(...numbers) // 42

var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
arr1.push(...arr2);

var arr1 = ['a', 'b'];
var arr2 = ['c'];
var arr3 = ['d', 'e'];

// ES5的合并数组
arr1.concat(arr2, arr3);
// [ 'a', 'b', 'c', 'd', 'e' ]

// ES6的合并数组
[...arr1, ...arr2, ...arr3]
// [ 'a', 'b', 'c', 'd', 'e' ]
```

## 箭头函数
> 箭头函数能很方便地创建匿名函数，特别是在 JavaScript 这种回调大量应用的语言中，箭头函数能让语法看起来更自然，使用起来也更便捷

箭头函数需要注意一下几个点：
* `this` 指向创建函数时的对象，而非执行时的对象
* 无法将其作为构造函数，即无法 `new`，否则会报错
* 无法使用 `arguments`，不过可以使用剩余函数 `rest` 来实现相同效果
* 无法使用 `yield` 命令，即无法作为 `Generator` 函数

```JavaScript
var result = values.sort((a, b) => a - b);

var f = () => 5;
// 等同于
var f = function (){ return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同于
var sum = function(num1, num2) {
  return num1 + num2;
};

var sum = (num1, num2) => { return num1 + num2; };

var getTempItem = id => ({ id: id, name: "Temp" });

// 嵌套函数
function insert(value) {
  return {into: function (array) {
    return {after: function (afterValue) {
      array.splice(array.indexOf(afterValue) + 1, 0, value);
      return array;
    }};
  }};
}

insert(2).into([1, 3]).after(1); //[1, 2, 3]

// 使用箭头函数改写
let insert = (value) => ({into: (array) => ({after: (afterValue) => {
  array.splice(array.indexOf(afterValue) + 1, 0, value);
  return array;
}})});

insert(2).into([1, 3]).after(1); //[1, 2, 3]
```
ES6 对函数还有一个 `Tail call optimization` （即“尾调用优化”），等后面再重新写一篇文章分析，尾调用优化将会大大避免递归的栈溢出

# 对象的扩展

## 简写
> ES6 中对象中属性的声明有了简便的做法，请看代码

```JavaScript
var birth = '2000/01/01';

var Person = {

  name: '张三',

  //等同于birth: birth
  birth,

  // 等同于hello: function ()...
  hello() { console.log('我的名字是', this.name); }

};

// 在 CommonJS 中使用 exports 暴露接口时很方便
module.exports = { getItem, setItem, clear };
// 等同于
module.exports = {
  getItem: getItem,
  setItem: setItem,
  clear: clear
};
```

## 判断对象相等
> ES6 使用 `Object.is()` 来判断两个对象是否相等，相对于使用 `==` 和 `===`，有以下两个区别

```JavaScript
+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```

## 对象复制
> ES6 使用 `Object.assign()` 对一个对象进行复制，但是执行的是浅复制，如果被复制的对象中有属性是一个对象，那么该对象的属性将会被共享

```JavaScript
var target = { a: 1, b: 1 };

var source1 = { b: 2, c: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}

Object.assign(target) === target // true

Object.assign(undefined) // 报错
Object.assign(null) // 报错

var v1 = 'abc';
var v2 = true;
var v3 = 10;

// 字符串是可枚举的，所以可以被复制，不可枚举属性将被忽略
var obj = Object.assign({}, v1, v2, v3);
console.log(obj); // { "0": "a", "1": "b", "2": "c" }

var obj = Object.assign(true); // true会被转换为对象返回

Object.assign([1, 2, 3], [4, 5]); // [4, 5, 3]

```

> 以上实例代码大部分来自阮一峰老师的 《ECMAScript 6 入门》 一书
> 书籍在线阅读地址: http://es6.ruanyifeng.com/#README
