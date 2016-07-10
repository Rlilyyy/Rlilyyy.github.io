---
title: Ajax的多种跨域方式
date: 2016-03-18 13:19:02
categories: 前端
tags: Javascript
---
## CORS（Cross-Origin Resource Sharing，跨源资源共享）
> CORS 的实现相对简单，当我们使用 XMLHttpRequest 进行跨域时，会给request的地址加入一个 Origin 头部，其中包含请求页面的源信息（`eg. Origin: http://www.baidu.com`），如果服务器接收到 request 后，需要给 response 加上 Access-Control-Allow-Origin 头部（`eg. Access-Control-Allow-Origin: http://www.baidu.com`），当然，如果是公共资源，将地址改成`*`就可以允许所有源地址进行跨域访问。如果服务器没有加入和 Origin 头部地址对应的头部或者 Access-Control-Allow-Origin 允许的跨域地址不相符合（`eg. Access-Control-Allow-Origin: http://www.google.com`），那么浏览器将会驳回请求，导致跨域失败。

### 1.XDomainRequest

XDomainRequest（以下简称XDR）是IE部分实现CORS规范的一个类型，和 XMLHttpRequest（以下简称XHR）不同的是，XDR 有以下不同：

* cookie 不会随请求发送，也不会随响应返回
* 只能设置请求头部信息中的 Content-Type 字段
* 不能访问响应头部信息
* 只支持 GET 和 POST 请求
* 浏览器支持：只有IE8+

### 2.XMLHttpRequest

使用 XMLHttpRequest（以下简称XHR）跨域和 XDR 跨域不同之处有以下几点：

* 可以访问status
* 可以访问statusText
* 支持同步请求

同时，跨域 XHR 也有以下限制（每个浏览器安全策略不同，例如chrome可以对跨域的XHR使用setRequestHeader）：

* 不能使用 setRequestHeader() 设置自定义头部
* 不能发送和接收cookie（可以通过设置 withCredentials 属性为 true 来允许发送cookie，同时服务器必须设置 Access-Control-Allow-Credentials: true）
* 调用 getAllResponseHeaders() 方法总会返回字符串

### 3.Preflighted Requests

Preflighted Requests 是一种透明服务器验证机制，使用即可支持开发人员使用自定义头部、GET 或 POST 以外的方法以及不同类型的主体内容。上面提到跨域XHR不能使用 setRequestHeader() 设置自定义头部，如果使用了 setRequestHeader() 设置自定义头部（eg. xhr.setRequestHeader("halo", "loha")），那么在发出正式请求之前，浏览器首先会发出一个 OPTION 方法的请求，其中包括以下头部内容：

* Origin: 参照上面
* Access-Control-Request-Method: 请求自身的方法，例如GET
* Access-Control-Request-Headers: 自定义头部信息，多个头部以逗号分隔，例如halo

当服务器接收到OPTION方法请求后，返回以下头部信息：

* Access-Control-Allow-Origin: 必须同源
* Access-Control-Allow-Method: 必须包含OPTION请求中的方法
* Access-Control-Allow-Headers: 必须包含 OPTION请求中的Header名
* Access-Control-Max-Age: 设置该 Preflighted 可以缓存的时间

当OPTION成功请求并收到返回，浏览器将会检查response的头部信息，如果符合条件则正式发起请求，同时缓存此次 Preflighted 请求的结果。反之，如果不符合，那么真正的请求会失败。

### 4.JSONP

JSONP（JSON with padding， 填充式JSON）与 JSON 并无关系，JSONP 的跨域实现主要运用 `<script>` 不受同域限制的原理，在请求的 URL 后加入一个 callback 参数，该 callback 参数代表服务器需要返回的JS代码的函数名称（`eg. <script src="http://localhost?callback=func">`），“func”这个名称将是服务器返回 JS 代码的函数名称（`eg. 以php为例，echo "func(123)"`），当 `<script>` 接收到这段代码的时候，便会运行 func 函数，并把“123”作为数据代进去（前提是你在前面已经写了一个func函数），至此，JSONP 完成。不过 JSONP 有以下缺点：

* 安全性无法保证
* 只能使用 GET 方法请求

### 5.图像ping

图像ping 利用 `<img>` 去访问一个 URL，同时对该 `<img>` 的 `onload` 和 `onerror` 事件进行监听，可以得知请求的成功与失败，但无法获取返回数据，多用于跟踪网页点击次数等。图像ping 有以下缺点：

* 只能使用 GET 方法请求
* 无法获取返回数据

### 6.iframe

想要操作 iframe 的内容，那么 iframe 和主页面必须同域。
使用 document.domain 可以修改iframe和主页面的域名为同一个，前提是两个页面处于同一主域名下（eg. p2p.a.com和www.a.com可以设置为a.com），这样就可以操作 iframe 里的 contentDocument 了
使用 location.hash（eg. 将a.com变为a.com#abc不会导致页面刷新），不过由于同域原因，要加多一个和主页面同域的页面才能成功修改主页面的 hash，主页面使用定时器检测 hash 变化。可以查看以下网址了解：
>http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html

使用 window.name，同样需要第三个和主页面同域的页面做代理，window.name 可容纳2M的内容。具体方法查看：
> http://www.cnblogs.com/rainman/archive/2011/02/21/1960044.html

使用 postMessage，参考
> http://www.cnblogs.com/dolphinX/p/3464056.html

### 7.Web Sockets

以 node.js 为例，基于 socket.io 可以很容易建立 Web Sockets， HTML5 的 Web Sockets 使用的是 `ws://` 和 `wss://` 协议，这样可以减少 `http://` 和 `https://` 协议字节级的开销，带宽消耗小。具体实现请查阅Google。
