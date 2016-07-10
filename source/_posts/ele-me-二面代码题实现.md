---
title: ele.me 二面代码题实现
date: 2016-04-13 15:53:59
tags: 面试
---
> 饿了么二面是在线写代码，有三道题，一道题十分钟内写完提交，在一个网站写，会自动录制代码编写过程

1. 首先写了一个判断数组的函数
```JavaScript
function isArray(array) {
    return !!array && Object.prototype.toString.call(array) === "[object Array]";
}
```

2. 有序数组的打乱
```JavaScript
function random(array) {
    if(!isArray(array)) return array;

    return array.sort(function() {
        return 0.8 - Math.random();
    });
}
```

3. 多维数组变为一维数组
```JavaScript
function flatten(array) {
    if(!isArray(array)) return [array];

    let len = array.length;
    let flattenArr = [];

    for(let idx=0;idx<len;idx++) {
        Array.prototype.push.apply(flattenArr, isArray(array[idx]) ? flatten(array[idx]) : [array[idx]]);
    }

    return flattenArr;
}

// 使用 ES6 的 Generator 函数实现
function flattenES6(array) {
    function* iterator(array) {
        if(isArray(array)) {
            for(let i=0;i<array.length;i++) {
                yield* iterator(array[i]);
            }
        }else {
            yield array;
        }
    }

    let result = [];
    for(let value of iterator(array)) {
        result.push(value);
    }

    return result;
}
```

4. 将一个乱序数组，里面有`*`和`数字`，将`*`提到最前，数字往后放，并且数字顺序不变
```JavaScript
function arrange(array) {
    if(!isArray(array)) return array;

    let prePoint = 0,
        nextPoint = 1,
        len = array.length;

    while(true) {
        while(array[prePoint] === "*" && prePoint < len) {
            prePoint++;
        }
        while(array[nextPoint] !== "*" && nextPoint < len) {
            nextPoint++;
        }
        while(prePoint > nextPoint) {
            nextPoint++;
        }
        if(prePoint >= len || nextPoint >= len) {
            break;
        }

        // exchange
        [array[prePoint], array[nextPoint]] = [array[nextPoint], array[prePoint]];
    }

    return array;
}
```
