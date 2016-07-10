---
title: ES6 学习笔记（四）
date: 2016-04-20 09:26:29
categories:
    - 前端
    - ECMAScript 6
tags: Javascript
---

# 二进制数组
> 二进制数组不是数组，只是类数组，二进制数组用于直接操作内存

## ArrayBuffer(long)
> `ArrayBuffer` 用于创建一段连续的内存地址，`long` 表示内存的长度（字节），但是不可以直接读写，只能通过 `视图` 读写，例如 `TypeArray`，`ArrayBuffer` 每个内存地址默认值都为 0。不同 `TypeArray` 操作同一组内存会互相影响

```JavaScript
var buf = new ArrayBuffer(32);
var dataView = new DataView(buf);
dataView.getUint8(0) // 0

// 操作同一组内存
var buffer = new ArrayBuffer(12);

var x1 = new Int32Array(buffer);
x1[0] = 1;
var x2 = new Uint8Array(buffer);
x2[0]  = 2;

x1[0] // 2
```

### Array.prototype.byteLength
> 获取分配的内存字节长度

```JavaScript
var buffer = new ArrayBuffer(32);
buffer.byteLength
// 32

if(buffer.byteLength) {
    // success
}else {
    // error
    // 分配的内存空间太大，可能存在没有那么大的内存空间而失败的情况
}
```

### Array.prototype.slice(start, end)
> 生成一段新的内存，拷贝 start 到 end-1 位置的内容到新的内存

```JavaScript
var buffer = new ArrayBuffer(8);
var newBuffer = buffer.slice(0, 3);
```

### ArrayBuffer.isView(buffer)
> 判断 buffer 是否是视图的实例化对象，该方法为 ArrayBuffer 的静态方法，直接调用

```JavaScript
var buffer = new ArrayBuffer(8);
ArrayBuffer.isView(buffer) // false

var v = new Int32Array(buffer);
ArrayBuffer.isView(v) // true
```

# TypeArray
> `TypeArray` 不是一个构造函数，而是一组构造函数，它包含 9 个不同的视图构造函数，都是类数组，都能用下标访问，都有 length 属性

* Int8Array：8位有符号整数，长度1个字节。
* Uint8Array：8位无符号整数，长度1个字节。
* Uint8ClampedArray：8位无符号整数，长度1个字节，溢出处理不同。
* Int16Array：16位有符号整数，长度2个字节。
* Uint16Array：16位无符号整数，长度2个字节。
* Int32Array：32位有符号整数，长度4个字节。
* Uint32Array：32位无符号整数，长度4个字节。
* Float32Array：32位浮点数，长度4个字节。
* Float64Array：64位浮点数，长度8个字节。

## TypedArray(buffer [, byteOffset [, length]])
> TypeArray 的构造函数可以传入三个参数

* 第一个参数，ArrayBuffer 的实例对象，必选
* 第二个参数，从 ArrayBuffer 的第 byteOffset 个字节开始读取，默认从 0 开始，注意，如果是 16 位的视图，需要是 2 的倍数，如果是 32 位 的视图，需要是 4 的倍数，以此类推，因为例如是 32 位的视图，那么该视图中 4 个字节为一个下标，为了完整读取内存，必须是每一个下标字节大小的整数倍，可选
* 第三个参数，读取 ArrayBuffer 的 length 个字节，默认读取到末尾，可选

```JavaScript
// 创建一个8字节的ArrayBuffer
var b = new ArrayBuffer(8);

// 创建一个指向b的Int32视图，开始于字节0，直到缓冲区的末尾
var v1 = new Int32Array(b);

// 创建一个指向b的Uint8视图，开始于字节2，直到缓冲区的末尾
var v2 = new Uint8Array(b, 2);

// 创建一个指向b的Int16视图，开始于字节2，长度为2
var v3 = new Int16Array(b, 2, 2);
```

## TypeArray(length)
> TypeArray 的构造函数还能直接通过分配内存创建视图

```JavaScript
var f64a = new Float64Array(8);
f64a[0] = 10;
f64a[1] = 20;
f64a[2] = f64a[0] + f64a[1];
```

## TypeArray(typedArray)
> TypeArray 的构造函数能接受另外一个 TypeArray 的实例作为构造参数，新的实例是开辟了新的内存地址同时复制了相同的值，不会互相影响

```JavaScript
var x = new Int8Array([1, 1]);
var y = new Int8Array(x);
x[0] // 1
y[0] // 1

x[0] = 2;
y[0] // 1
```

## TypedArray(arrayLikeObject)
> TypeArray 的构造函数能接受一个类数组的参数，会开辟一段新的内存实例化视图

```JavaScript
var x = new Int8Array([1, 1]);
var y = new Int8Array(x.buffer);
x[0] // 1
y[0] // 1

x[0] = 2;
y[0] // 2
```

## 数组方法
> 普通数组的方法对 TypeArray 也完全适用

```JavaScript
// 缺少 contact 方法
TypedArray.prototype.copyWithin(target, start[, end = this.length])
TypedArray.prototype.entries()
TypedArray.prototype.every(callbackfn, thisArg?)
TypedArray.prototype.fill(value, start=0, end=this.length)
TypedArray.prototype.filter(callbackfn, thisArg?)
TypedArray.prototype.find(predicate, thisArg?)
TypedArray.prototype.findIndex(predicate, thisArg?)
TypedArray.prototype.forEach(callbackfn, thisArg?)
TypedArray.prototype.indexOf(searchElement, fromIndex=0)
TypedArray.prototype.join(separator)
TypedArray.prototype.keys()
TypedArray.prototype.lastIndexOf(searchElement, fromIndex?)
TypedArray.prototype.map(callbackfn, thisArg?)
TypedArray.prototype.reduce(callbackfn, initialValue?)
TypedArray.prototype.reduceRight(callbackfn, initialValue?)
TypedArray.prototype.reverse()
TypedArray.prototype.slice(start=0, end=this.length)
TypedArray.prototype.some(callbackfn, thisArg?)
TypedArray.prototype.sort(comparefn)
TypedArray.prototype.toLocaleString(reserved1?, reserved2?)
TypedArray.prototype.toString()
TypedArray.prototype.values()
```

## 小端字节序
> 如果使用一个 32 位的视图读取并修改一段内存后，再使用一个 16 位的视图读取该段内存缓冲时，其长度应该是 32 位视图的两倍，这时候 16 位视图中的数据存放就会出现一个现象，跟字节排放的规则相关，看示例

```JavaScript
var buffer = new ArrayBuffer(16);
var int32View = new Int32Array(buffer);

for (var i = 0; i < int32View.length; i++) {
  int32View[i] = i * 2;
}

console.log(int32View); // [0, 2, 4, 6]

var int16View = new Int16Array(buffer);

console.log(int16View); // [0, 0, 2, 0, 4, 0, 6, 0]
```

由于 32 位视图是 4 个字节存放一个数据，16 位视图是 2 个字节存放一个数据，那么 16 位视图明显要使用 2 个数据位来存放一个 32 位视图的数据，而之所以 16 位视图都是两个数据位的第一个数据位显示为正确数据，第二个显示为 0，这就涉及现代计算机都采用了小端字节序的规则在内存中存放数据的原因，而 TypeArray 也是采用该种策略。小端字节序简略地说就是从后往前一个字节一个字节地存放数据到内存的每个地址中，更详细的解释可以参考：http://blog.csdn.net/ce123/article/details/6971544 小端字节序也导致了 TypeArray 无法正确读取大端字节序的内容

## TypeArray.prototype.BYTES_PER_ELEMENT
> 用于输出每一个 TypeArray 类型视图的每一位所占的字节数

```JavaScript
Int8Array.BYTES_PER_ELEMENT // 1
Uint8Array.BYTES_PER_ELEMENT // 1
Int16Array.BYTES_PER_ELEMENT // 2
Uint16Array.BYTES_PER_ELEMENT // 2
Int32Array.BYTES_PER_ELEMENT // 4
Uint32Array.BYTES_PER_ELEMENT // 4
Float32Array.BYTES_PER_ELEMENT // 4
Float64Array.BYTES_PER_ELEMENT // 8
```

## 溢出
> TypeArray 对于溢出的处理就是直接抛弃溢出的位

* 正向溢出（overflow）：当输入值大于当前数据类型的最大值，结果等于当前数据类型的最小值加上余值，再减去1
* 负向溢出（underflow）：当输入值小于当前数据类型的最小值，结果等于当前数据类型的最大值减去余值，再加上1

``` JavaScript
var int8 = new Int8Array(1);

int8[0] = 128;
int8[0] // -128

int8[0] = -129;
int8[0] // 127
```

## 其他方法

* TypedArray.prototype.buffer，返回视图整段内存引用
* TypedArray.prototype.byteLength，返回视图的内存字节长度
* TypedArray.prototype.byteOffset，返回视图从内存第几个字节开始引用
* TypedArray.prototype.length，返回视图的成员长度，与其每个成员占用字节数相关
* TypedArray.prototype.set(typeArray[, startIndex])，将一段内存完全复制到另一端内存中
* TypedArray.prototype.subarray(startIndex, endIndex)，从一个视图上截取一部分创建为一个新的视图
* TypedArray.prototype.slice(index)，从一个视图上截取一段创建一个新的视图，index 可以为负数，表示倒数
* TypedArray.of()，将参数创建为一个新的视图
* TypedArray.from(arrayLike[, callback])，将 arrayLike 转化为一个相应的可遍历的视图类型，callback 的功能类似于数组的 map 功能

# DataView
> DataView 可以自行设置大端字节序或者小端字节序，同时有 8 个获取数据的方法和 8 个设置数据的方法

* getInt8：读取1个字节，返回一个8位整数。
* getUint8：读取1个字节，返回一个无符号的8位整数。
* getInt16：读取2个字节，返回一个16位整数。
* getUint16：读取2个字节，返回一个无符号的16位整数。
* getInt32：读取4个字节，返回一个32位整数。
* getUint32：读取4个字节，返回一个无符号的32位整数。
* getFloat32：读取4个字节，返回一个32位浮点数。
* getFloat64：读取8个字节，返回一个64位浮点数。

这 8 个方法都传入一个正整数表示从第几个字节开始读取，第二个参数为一个布尔值，true 代表使用小端字节序，false 代表使用大端字节序，默认为大端字节序

* setInt8：写入1个字节的8位整数。
* setUint8：写入1个字节的8位无符号整数。
* setInt16：写入2个字节的16位整数。
* setUint16：写入2个字节的16位无符号整数。
* setInt32：写入4个字节的32位整数。
* setUint32：写入4个字节的32位无符号整数。
* setFloat32：写入4个字节的32位浮点数。
* setFloat64：写入8个字节的64位浮点数。

这 8 个方法都可以传入三个参数，第一个表示从第几个字节写入数据，第二个表示写入数据的内容，第三个指定大小端字节序

# 其他一些新特性

## AJAX
> AJAX 可以设置 `responseType` 为 `arraybuffer` 明确指定响应数据位二进制数据，如果不明确，可以设置为 `blod`

## Canvas
> Canvas 使用 `Uint8ClampedArray` 读取二进制像素数据，该 TypeArray 可以自动过滤高位溢出，取值范围始终为 0-255

## WebSocket
> WebSocket 可以发送二进制数据

## Fetch API
> Fetch API取回的数据，就是 `ArrayBuffer` 对象

## File API
> File API 可以将文件数据作为二进制的 `ArrayBuffer` 类型读取处理

> 以上实例代码大部分来自阮一峰老师的 《ECMAScript 6 入门》 一书
> 书籍在线阅读地址: http://es6.ruanyifeng.com/#README
