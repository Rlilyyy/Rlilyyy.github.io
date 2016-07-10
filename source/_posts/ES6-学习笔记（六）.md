---
title: ES6 学习笔记（六）
date: 2016-04-23 22:13:41
categories:
    - 前端
    - ECMAScript 6
tags: Javascript
---

# Iterator
> Iterator的作用有三个：一是为各种数据结构，提供一个统一的、简便的访问接口；二是使得数据结构的成员能够按某种次序排列；三是ES6创造了一种新的遍历命令for...of循环，Iterator接口主要供for...of消费

## 部署 Iterator 接口
数组、Map、Set、类数组都有部署 Iterator 的接口，但是对象并没有，下面展示如何为对象部署 Iterator 接口

```JavaScript

// 第一种
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }

  // Symbol.Iterator 即部署接口
  [Symbol.iterator]() { return this; }

  // next() 代表迭代器的指针运动逻辑
  // value 默认是当前值
  // done 表示当前值是否为末尾，布尔值
  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    } else {
      return {done: true, value: undefined};
    }
  }
}

function range(start, stop) {
  return new RangeIterator(start, stop);
}

for (var value of range(0, 3)) {
  console.log(value);
}

// 第二种
function Obj(value){
  this.value = value;
  this.next = null;
}

Obj.prototype[Symbol.iterator] = function(){

  var iterator = {
    next: next
  };

  var current = this;

  function next(){
    if (current){
      var value = current.value;
      var done = current === null;
      current = current.next;
      return {
        done: done,
        value: value
      }
    } else {
      return {
        done: true
      }
    }
  }
  return iterator;
}

var one = new Obj(1);
var two = new Obj(2);
var three = new Obj(3);

one.next = two;
two.next = three;

for (var i of one){
  console.log(i)
}
// 1
// 2
// 3

// 第三种
let obj = {
  data: [ 'hello', 'world' ],
  [Symbol.iterator]() {
    const self = this;
    let index = 0;
    return {
      next() {
        if (index < self.data.length) {
          return {
            value: self.data[index++],
            done: false
          };
        } else {
          return { value: undefined, done: true };
        }
      }
    };
  }
};
```
上面的三个例子都有一个共同点，就是部署 Iterator 接口都是在对象的 `Symbol.iterator` 属性中返回一个对象，而该对象包含一个 `next()` 函数指明指针的运动逻辑，并返回 `value` 和 `done` 两个值。还有另外一个 `return()` 方法用于在中断遍历时的操作，必须返回一个对象。此外，也可以直接使用数组的 Iterator 作为对象的迭代器

```JavaScript
let iterable = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
for (let item of iterable) {
  console.log(item); // 'a', 'b', 'c'
}
```

注意，如果 `Symbol.iterator` 属性返回并不是一个迭代器的标准对象，那么会报错

```JavaScript
var obj = {};

obj[Symbol.iterator] = () => 1;

[...obj] // TypeError: [] is not a function
```

## Iterator 的使用场景

### 解构赋值
> 对数组的解构赋值默认会调用 `Symbol.iterator` 方法

```JavaScript
let set = new Set().add('a').add('b').add('c');

let [x,y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```

### 扩展运算符

```JavaScript
var str = 'hello';
[...str] //  ['h','e','l','l','o']

// 例二
let arr = ['b', 'c'];
['a', ...arr, 'd']
// ['a', 'b', 'c', 'd']
```

### yield*
> 这个还没学到，不予评论，后面学到了再填坑

### 其他场景
* for...of
* Array.from()
* Map(), Set(), WeakMap(), WeakSet()（比如new Map([['a',1],['b',2]])）
* Promise.all()
* Promise.race()
* 字符串也部署了 Iterator 接口


# for…of

* for…of 遍历数组，key 值保持为数字，而 for…in 会把 key 值变为字符串
* for…of 可以 break 或 return，而 forEach() 不可以
* for…of 可以正确识别 32 位 UTF-16 字符

> 以上实例代码大部分来自阮一峰老师的 《ECMAScript 6 入门》 一书
> 书籍在线阅读地址: http://es6.ruanyifeng.com/#README
