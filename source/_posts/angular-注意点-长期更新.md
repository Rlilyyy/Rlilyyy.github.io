---
title: angular 注意点（长期更新）
date: 2016-06-05 17:30:38
categories:
    - 前端
    - Angular
tags: Javascript
---
> 这篇 blog 主要存放我在学习和使用 angular 踩到的坑和需要注意的点

# ng-repeat
> `ng-repeat` 用于标识某个 elem 需要重复输出，同时重复输出的内容需为唯一

```html
<div ng-app="app" ng-controller="control">
    <h3 ng-repeat="content in repeatContent">ng-repeat: {{ content }}</h3>
</div>
```

```JavaScript
let app = angular.module("app", []);
app.controller("control", ($scope) => {
    // 输出李滨泓
    $scope.repeatContent = ["李", "滨", "泓"];
    // 下面存在两个“泓”，会报错
    // $scope.repeatContent = ["李", "滨", "泓", "泓"];
})
```

# provider, service, factory 之间的关系

## factory
> `factory` 很像 `service`，不同之处在于，service 在 Angular 中是一个单例对象，即当需要使用 service 时，使用 new 关键字来创建一个（也仅此一个）service。而 factory 则是一个普通的函数，当需要用时，他也仅仅是一个普通函数的调用方式，它可以返回各种形式的数据，例如通过返回一个功能函数的集合对象来将供与使用。

定义：
```Javascript
let app = angular.module("app", []);

// 这里可以注入 $http 等 Provider
app.factory("Today", () => {
    let date = new Date();
    return {
        year: date.getFullYear(),
        month: date.getMonth() + 1,
        day: date.getDate()
    };
});
```

使用注入：
```Javascript
app.controller("control", (Today) => {
    console.log(Today.year);
    console.log(Today.month);
    console.log(Today.day);
});
```

## service
> `service` 在使用时是一个单例对象，同时也是一个 constructor，它的特点让它可以不返回任何东西，因为它使用 new 关键字新建，同时它可以用在 controller 之间的通讯与数据交互，因为 controller 在无用时其作用域链会被销毁（例如使用路由跳转到另一个页面，同时使用了另一个 controller）

定义：
```Javascript
let app = angular.module("app", []);

// 这里可以注入 $http 等 Provider
// 注意这里不可以使用 arrow function
// arrow function 不能作为 constructor
app.service("Today", function() {
    let date = new Date();
    this.year = date.getFullYear();
    this.month = date.getMonth() + 1;
    this.day = date.getDate();
});
```

使用注入：
```Javascript
app.controller("control", (Today) => {
    console.log(Today.year);
    console.log(Today.month);
    console.log(Today.day);
});
```

## provider
> `provider` 是 `service` 的底层创建方式，可以理解 provider 是一个可配置版的 service，我们可以在正式注入 provider 前对 provider 进行一些参数的配置。

定义：
```Javascript
let app = angular.module("app", []);

// 这里可以注入 $http 等 Provider
// 注意这里不可以使用 arrow function
// arrow function 不能作为 constructor
app.provider("Today", function() {
    this.date = new Date();
    let self = this;

    this.setDate = (year, month, day) => {
        this.date = new Date(year, month - 1, day);
    }

    this.$get = () => {
        return {
            year: this.date.getFullYear(),
            month: this.date.getMonth() + 1,
            day: this.date.getDate()
        };
    };
});
```

使用注入：
```Javascript
// 这里重新配置了今天的日期是 2015年2月15日
// 注意这里注入的是 TodayProvider，使用驼峰命名来注入正确的需要配置的 provider
app.config((TodayProvider) => {
    TodayProvider.setDate(2015, 2, 15);
});

app.controller("control", (Today) => {
    console.log(Today.year);
    console.log(Today.month);
    console.log(Today.day);
});
```

# handlebars 与 angular 符号解析冲突
场景：
> 当我使用 node.js 作为服务端，而其中使用了 handlebars 作为模板引擎，当 node.js 对某 URL 进行相应并 render，由于其模板使用 `{ {} }` 作为变量解析符号。同样地，angular 也使用 `{ {} }` 作为变量解析符号，所以当 node.js 进行 render 页面后，如果 `{ {} }` 内的变量不存在，则该个区域会被清空，而我的原意是这个作为 angular 的解析所用，而不是 handlebars 使用，同时我也想继续使用 handlebars，那么此时就需要将 angular 默认的 `{ {} }` 解析符号重新定义。即使用依赖注入 `$interpolateProvider` 进行定义，如下示例：

```Javascript
app.config($interpolateProvider => {
    $interpolateProvider.startSymbol('{[{');
    $interpolateProvider.endSymbol('}]}');
});
```

# ng-annotate-loader
> `ng-annotate-loader` 应用于 webpack + angular 的开发场景，是用于解决 angular 在进行 JS 压缩后导致依赖注入失效并出现错误的解决方法

安装
```bash
$ npm install ng-annotate-loader --save-dev
```

配置
```Javascript
// webpack.config.js
{
    test: /\.js?$/,
    exclude: /(node_modules|bower_components)/,
    loader: 'ng-annotate!babel?presets=es2015'
},
```

# 双向数据绑定
> 当我们使用非 Angular 自带的事件时，$scope 里的数据改变并不会引起 `$digest` 的 `dirty-checking` 循环，这将导致当 `model` 改变时，`view` 不会同步更新，这时我们需要自己主动触发更新

HTML
```html
<div>{{ foo }}</div>
<button id="addBtn">go</button>
```

JavaScript
```Javascript
app.controller("control", ($scope) => {
    $scope.foo = 0;
    document.getElementById("addBtn").addEventListener("click", () => {
        $scope.foo++;
    }, false);
})
```

很明显，示例的意图是当点击 button 时，`foo` 自增长并更新 View，但是实际上，$scope.foo 是改变了，但是 View 并不会刷新，这是因为 foo 并没有一个 $watch 检测变化后 $apply，最终引起 $digest，所以我们需要自己触发 $apply 或者创建一个 $watch 来触发或检测数据变化

JavaScript（使用 $apply）
```Javascript
app.controller("control", ($scope) => {
    $scope.foo = 0;
    document.getElementById("addBtn").addEventListener("click", () => {
        
        $scope.$apply(function() {
            $scope.foo++;
        });

    }, false);
})
```

JavaScript（使用 $watch & $digest）
```Javascript
app.controller("control", ($scope) => {
    $scope.foo = 0;
    $scope.flag = 0;

    $scope.$watch("flag", (newValue, oldValue) => {

        // 当 $digest 循环检测 flag 时，如果新旧值不一致将调用该函数
        $scope.foo = $scope.flag;
    });

    document.getElementById("addBtn").addEventListener("click", () => {
       
        $scope.flag++;
        // 主动触发 $digest 循环
        $scope.$digest();
    }, false);
})
```

## $watch(watchExpression, listener, [objectEquality])
> 注册一个 `listener` 回调函数，在每次 `watchExpression` 的值发生改变时调用

* `watchExpression` 在每次 `$digest` 执行时被调用，并返回要被检测的值（当多次输入同样的值时，`watchExpression` 不应该改变其自身的值，否则可能会引起多次的 $digest 循环，`watchExpression` 应该幂等）
* `listener` 将在当前 `watchExpression` 返回值和上次的 `watchExpression` 返回值不一致时被调用（使用 `!==` 来严格地判断不一致性，而不是使用 `==` 来判断，不过 `objectEquality == true` 除外）
* `objectEquality` 为 `boolean` 值，当为 `true` 时，将使用 `angular.equals` 来判断一致性，并使用 `angular.copy` 来保存此次的 Object 拷贝副本供给下一次的比较，这意味着复杂的对象检测将会有性能和内存上的问题

## $apply([exp])
> `$apply` 是 `$scope` 的一个函数，用于触发 `$digest` 循环

$apply 伪代码
```Javascript
function $apply(expr) {
    try {
        return $eval(expr);
    } catch (e) {
        $exceptionHandler(e);
    } finally {
        $root.$digest();
    }
}
```

* 使用 `$eval(expr)` 执行 `expr` 表达式
* 如果在执行过程中跑出 exception，那么执行 $exceptionHandler(e)
* 最后无论结果，都会执行一次 `$digest` 循环