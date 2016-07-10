---
title: ES6 学习笔记
date: 2016-04-06 08:11:04
categories:
    - 前端
    - ECMAScript 6
tags: Javascript
---

# let 和 const

## let
> 使用 `let` 定义的变量只在块级作用域里可以访问，而 `var` 定义的变量没有块级作用域的概念，只要在作用域内即可访问

### 我们可以在循环内使用 let 代替 var
```JavaScript
for(let i=0;i<5;i++) {
    console.log(i);// 0,1,2,3,4
}
console.log(i);// Uncaught ReferenceError: i is not defined
```

### let 声明的变量不存在变量提升
```JavaScript
console.log(i);// undefined
console.log(j);// Uncaught ReferenceError

var i = 5;
let j = 6;
```

### 暂时性死区(temporal dead zone，简称TDZ)
> 在使用 let 和 const 定义变量的块级作用域里，会形成封闭的块级作用域，在使用 let 或 const 定义变量的语句之前，该变量无法被赋值，都会抛出 ReferenceError

```JavaScript
if(true) {
    tmp = "a";// ReferenceError
    console.log(tmp);// ReferenceError

    // 死区结束
    let tmp;
    console.log(tmp);// undefined

    tmp = "b";
    console.log(tmp);// b
}
```

### 其他一些限制
* 使用 let 声明的变量不允许在同一块级作用域内重复声明
* 块级作用域内声明的函数只在该块级作用域内可用
* ES6 块级作用域内的声明的函数不存在函数提升

## const
> 使用 `const` 声明的变量无法更改，严格模式下重复赋值会报错，而常规模式下不报错也不赋值成功

### 限制
* const 同样存在暂时性死区
* const 不允许重复定义，不存在变量提升
* 使用 const 声明一个对象，只保证该对象的指针不被修改，不保证该对象内部属性不被修改
* 使用 const 声明的变量只在当前块级作用域内有效
* 使用 const 声明的变量可以跨模块使用
```JavaScript
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```
* 在 window 作用域下使用 const 声明的变量不会变为 window.? 变量

# 解构赋值
> ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）

```JavaScript
// ES5
var a = 1;
var b = 2;
var c = 3;

// ES6
// 解构成功
var [a, b, c] = [1, 2, 3];

// 解构失败——部分解构
var [x, y] = [1];
console.log(x);// 1
console.log(y);// undefined

// 解构成功——不完全解构
var [i, [j], k] = [1, [2, 3], 4];
console.log(i);// 1
console.log(j);// 2
console.log(k);// 4

// 模式匹配的解构
var [a1, [a2, a3]] = [1, [2, 3]];
console.log(a1);// 1
console.log(a2);// 2
console.log(a3);// 3

var [, , tmp] = [1, 2, 3];
console.log(tmp);// 3

var [x1, ...x2] = [1, 2, 3, 4, 5];
console.log(x1);// 1
console.log(x2);// [2, 3, 4, 5]

var [y1, y2, ...y3] = [1];
console.log(y1);// 1
console.log(y2);// undefined
console.log(y3);// []

// 有默认值的解构赋值
// 只有对应赋值是 undefined，默认值才会生效
var [z1 ,z2 = 2] = [1, undefined];
console.log(z1);// 1
console.log(z2);// 2

// 对象的解构赋值
// 位置可以不同
var {q2, q1} = {q1: 1, q2: 2};
console.log(q1);// 1
console.log(q2);// 2

// 下面这个有点反人类
// 真正被赋值的是后者 t1, t2，而不是前者 r1, r2
var {r1: t1, r2: t2} = {r1: 1, r2: 2};
console.log(t1);// 1
console.log(t2);// 2

// 字符串的解构赋值
var [s1, s2, s3] = "abc";
console.log(s1);// a
console.log(s2);// b
console.log(s3);// c

// 函数参数的解构赋值
function func([x, y]) {
    return x + y;
}
func([1, 2]);// 3

function func2([x = 1, y = 2] = []) {
    return [x, y];
}
func2([2, 3]);// [2, 3]
func2([2]);// [2, 2]
func2([]);// [1, 2]

// 变量赋值交换
function exchange([x, y]) {
    var [x, y] = [y, x];
    return [x, y];
}
exchange([1, 2]);// [2, 1]
```

# 字符串

## Unicode 表示法
```JavaScript
// 单字节 Unicode
console.log("\u0061");// a

// 双字节 Unicode
console.log("\u{20BB7}");// 𠮷

let hello = 123;
hell\u{6F} // 123

// 对于双字节字符，用 codePointAt() 正确获取对应十进制字符码
var s = "𠮷a";
// 这里是“𠮷”的十进制字符码
console.log(s.codePointAt(0));// 134071
// 这里是“𠮷”的第二个十进制字符码
console.log(s.codePointAt(1));// 57271
// 这里是“a”的十进制字符码
console.log(s.codePointAt(2));// 97

// 合成字符的规范化，用于正确判断相等性
console.log("\u01D1".normalize() === "\u004F\u030C".normalize()) // true
console.log("\u01D1" === "\u004F\u030C");
```

## 其他函数
* includes(code, startIndex)：返回布尔值，表示是否找到了参数字符串
* startsWith(code, startIndex)：返回布尔值，表示参数字符串是否在源字符串的头部
* endsWith(code, startIndex)：返回布尔值，表示参数字符串是否在源字符串的尾部
* repeat()：返回一个新字符串，表示将原字符串重复n次

## 模板字符串
```JavaScript
// ES5
$("#result").append(
  "There are <b>" + basket.count + "</b> " +
  "items in your basket, " +
  "<em>" + basket.onSale +
  "</em> are on sale!"
);

// ES6
// 注意模板字符串使用 `` 代替 ""
// ${} 代表变量
$("#result").append(`
  There are <b>${basket.count}</b> items
   in your basket, <em>${basket.onSale}</em>
  are on sale!
`);

// 可以在模板字符串内运算
var x = 1;
var y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

var obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// 3

// 可以调用函数
function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar

// raw 会自动转义斜杠
String.raw`Hi\n${2+3}!`;// Hi\n5!

// 此外还有标签模板等扩展，更详细的看教程
```

# 数组的扩展

## 新的数组函数
* Array.from()，用于将类数组对象转换为真正的数组，如果有第二个参数可是实现类似 map() 的效果，使用[...array]会有相同结果
* Array.of()，用于将一组数值转换为数组对象
* (new Array()).copyWithin(target, startIndex, endIndex)，用于将 startIndex 到 endIndex 的元素复制到 target 位为开头的位置上
* (new Array()).find(arr, callback)，用于寻找 arr 数组中符合 callback 中条件的第一个元素
* (new Array()).findIndex(arr, callback)，用于寻找 arr 数组中符合 callback 中条件的第一个元素的索引
* (new Array()).fill(char, starIndex, endIndex)，用于用 char 填充数组
* (new Array()).entries()，返回所有键-值(key-value)对
* (new Array()).keys()，返回所有键(key)的值
* (new Array()).values()，返回所有值(value)

> 以上例子参考自阮一峰老师的《ECMAScript 6入门》一书的开源版本，地址：http://es6.ruanyifeng.com/#README
