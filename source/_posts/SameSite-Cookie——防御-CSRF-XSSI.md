---
title: 'SameSite-Cookie——防御 CSRF & XSSI'
date: 2016-07-10 23:06:38
categories:
    - 前端
    - 前端安全
tags: cookie
---
# SameSite——防御 CSRF & XSSI 新机制
> `SameSite-cookies` 是 Google 开发的用于防御 CSRF 和 XSSI（Cross Site Script Inclusion，跨域脚本包含）的新安全机制，只需在 `Set-Cookie` 中加入一个新的字段属性，浏览器会根据设置的安全级别进行对应的安全 cookie 发送拦截，而目前在 Chrome-dev（51.0.2704.4）中可用。

## XSSI——Cross Site Script Inclusion
`XSSI` 属于 `XSS` 攻击的一种攻击方式，一般来说，浏览器允许网页加载其他域的脚本或图片等，假设我们在安全的网站上 a.com 包含一个脚本文件 `getData.js` 用于读取用户的私人信息，第一次用户需要在 a.com 登录，然后就可以根据验证返回用户私人信息并设置 cookie 以便下次使用，此时我们只做一个恶意网站 c.com，并包含了 `getData.js` 这个脚本文件，当用户点击 c.com 时，他的信息就泄露了。

当然，这种方法可以用 `token` 令牌等来解决，另外一种方法就是 `SameSite-Cookies`，通过验证是否是从 a.com 访问的 getData.js 的方式来阻止 cookie 的发送。

## SameSite 使用方式
需要在 `Set-Cookie` 中加入 `SameSite` 关键字，例如
> Set-Cookie: key=value; HttpOnly; SameSite=Strict

## SameSite 的两个属性
### no_restriction
当没有添加 SameSite 关键字的时候，默认是空的

### Lax
* 允许发送安全 HTTP 方法（`GET`, `HEAD`, `OPTIONS`, `TRACE`）第三方链接的 cookies
* 必须是 TOP-LEVEL 即可引起地址栏变化的跳转方式，例如 `<a>`, `<link rel="prerender">`, `GET 方式的 form 表单`，此外，`XMLHttpRequest`, `<img>` 等方式进行 GET 方式的访问将不会发送 cookies
* 但是禁止发送不安全 HTTP 方法（`POST`, `PUT`, `DELETE`）第三方链接的cookies

### Strict
禁止发送所有第三方链接的 cookies，默认情况下，如果添加了 SameSite 关键字，但是没有指定 value（Lax or Stict），那么默认为 Strict

## Strict 下的 Request Cookie 变化
当我们通过其他网站来访问一个有 SameSite-Cookies 机制的网站时，例如从 a.com 点击链接进入 b.com，如果 b.com 设置了 `Set-Cookie:foo=bar;SameSite=Strict`，那么，`foo=bar` 这一 cookie 是不会随着 request 发送的，如下图：

![outerLink](http://7xoehm.com1.z0.glb.clouddn.com/outerLink.png)

可以看到 `foo=bar` 并没有发送，假如此时刷新一下页面或者直接从地址栏输入 b.com 的地址，那么 cookie 将会正常发送，不过假设用户点击了外链进入了 b.com，由于 SameSite-Cookies 的保护，他的 cookie 第一次受到了保护，但是假设用户此时手动刷新了一下页面，那么 cookie 将会被发送出去，所以可能需要其他一些验证手段来对 cookie 未随 request 发送的情况来做相对应的处理，不过该种情况仅限于 GET 方法下，如下图：

![innerLink](http://7xoehm.com1.z0.glb.clouddn.com/innerLink.png)

此外，为了更清楚地了解 cookie 中哪些字段是 SameSite 控制并且级别为 Lax 或 Strict 或 none，我们可以在 Chrome DevTools 上清楚地看到，如下图：

![devTools](http://7xoehm.com1.z0.glb.clouddn.com/devTools.png)

[点击查看 DEMO](http://www.libinhong.com:5000)

参考资料：
* http://www.sjoerdlangkemper.nl/2016/04/14/preventing-csrf-with-samesite-cookie-attribute/
* https://tools.ietf.org/html/draft-west-first-party-cookies-07
* https://chloe.re/2016/04/13/goodbye-csrf-samesite-to-the-rescue/
