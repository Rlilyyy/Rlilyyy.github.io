---
title: ES6 学习笔记（五）
date: 2016-04-21 13:57:05
categories:
    - 前端
    - ECMAScript 6
tags: Javascript
---

# Set
> Set 是一个类数组的数据结构，但是它的值都是独一无二的，并且它是一个构造函数

Set 有以下几个特点
* 值是不重复的
* NaN 等于自身
* 添加值的时候不会进行类型转换，5 和 '5' 是不一样的
* 可以接受一组数组进行初始化，会自动过滤重复的数
* 两个空对象不互相相等

## Set 的方法

* Set.prototype.constructor：构造函数，默认就是Set函数
* Set.prototype.size：返回Set实例的成员总数
* add(value)：添加某个值，返回Set结构本身，由于返回值的特点，可以链式调用
```JavaScript
s.add(1).add(2).add(2);
```
* delete(value)：删除某个值，返回一个布尔值，表示删除是否成功
* has(value)：返回一个布尔值，表示该值是否为Set的成员
* clear()：清除所有成员，没有返回值
* keys()：返回一个键名的遍历器
* values()：返回一个键值的遍历器
* entries()：返回一个键值对的遍历器
* forEach()：使用回调函数遍历每个成员


# WeakSet

WeakSet 有以下几个特点
* 加入成员只能是对象
* 成员都是弱引用，即垃圾回收机制不会因其引用对象而保持对象的存在
* 无法引用成员对象
* 无法变量成员对象

## WeakSet 的方法

* WeakSet.prototype.add(value)：向WeakSet实例添加一个新成员
* WeakSet.prototype.delete(value)：清除WeakSet实例的指定成员
* WeakSet.prototype.has(value)：返回一个布尔值，表示某个值是否在WeakSet实例之中

# Map
> 与对象的 key-value 结构类似的一种集合数据结构，不用的是 Map 可以使用任意一种类型的值作为 key，不会发生自动转换为字符串的情况

* size：返回Map结构的成员总数
* set(key, value)：设置key所对应的键值，然后返回整个Map结构。如果key已经有值，则键值会被更新，否则就新生成该键
* get(key)：读取key对应的键值，如果找不到key，返回undefined
* has(key)：返回一个布尔值，表示某个键是否在Map数据结构中
* delete(key)：删除某个键，返回true。如果删除失败，返回false
* clear()：清除所有成员，没有返回值

# WeakMap
> 与 WeakSet 的特点类似，不过这个相对于 Map 的数据结构的，只能使用 `get()`、`set()`、`has()`、`delete()`

> 以上实例代码大部分来自阮一峰老师的 《ECMAScript 6 入门》 一书
> 书籍在线阅读地址: http://es6.ruanyifeng.com/#README
