---
title: gulp 简单使用
date: 2016-04-04 21:08:57
tags: gulp
categories: 前端
---

# 前言

> 今天必须安利一下 gulp 这个前端自动化构建工具，天呐噜，今天学了一下怎么用以后，我已经忘记什么是 Grunt 了！gulp 比 Grunt 好的地方就是，基于 Node.js 的流，使用 JS 语法，配置项相对于 Grunt 的简直就简单得不要不要的（虽然我 Grunt 也用得很少，因为发现用 Grunt 的配置时间比我自己去压缩什么用的时间还长……），另外，两个配置文件的代码清晰度和简洁度的对比上，gulp 就是完胜啊么么哒~

# 引入和使用

1. 安装好 `Node.js` 和 `npm` 两个神器以后，往下看

2. 使用 npm 全局安装 gulp，命令行如下所示
> npm install gulp -g

3. 或者你也可以不全局安装，仅仅作为项目开发依赖，那么使用如下命令行安装 gulp
> npm install gulp --save-dev

4. 在项目根目录下创建一个 `gulpfile.js` 文件，用于 gulp 任务的配置

5. 在 `gulpfile.js` 里进行任务配置，例如：
```JavaScript
var gulp = require("gulp");
// JavaScript 压缩
var uglify = require("gulp-uglify");
// JavaScript 语法检测
var jshint = require("gulp-jshint");
// 文件输出重命名
var rename = require("gulp-rename");

gulp.task("jshint", function() {
    gulp.src("Nope.js")
        .pipe(jshint())
        .pipe(jshint.reporter("default"));
});

gulp.task("compress", function() {
    gulp.src("Nope.js")
        .pipe(rename("Nope.min.js"))
        .pipe(uglify())
        .pipe(gulp.dest("gulpJS/"));
});

gulp.task("default", ["jshint", "compress"]);
```

6. 在当先目录下使用命令行工具，输入命令执行 gulp 进行自动构建
> gulp
或者
> gulp jshint

如果单独输入 gulp，那么会执行 `default` 任务，如果在 gulp 后面加一个任务名称，那么就是执行特定名称任务

# API 说明

* gulp.task(name[, deps], fn)
> task 用于配置一个单独的任务，name 是该任务的名称；deps 是该任务的依赖任务，会在该任务执行前执行完毕；fn 是该任务的执行体。

* gulp.src(globs[, options])
> src 用于配置该任务的目标文件，globs 可以配置文件路径，当然不止于此，更多的写法可以查看官网的 API 文档；options 用于配置一些选项，如 `{buffer: false}`，将文件输入改为 stream 而不是 buffer。

* gulp.pipe()
> 使用过 Node.js 就很容易明白中间件的概念，pipe 的作用类似于中间件的作用，对文件进行处理后输出。

* gulp.dest(path[, options])
> dest 用于将 pipe 进来的文件流输出到 path 下，options 用于配置写文件的一些权限等。

# 结束语

> 上面提到了 gulp 的几个常用 API，此外还有几个 API 我没用到，例如 watch，可能后续在使用 less 的时候会用到。gulp 的 API 就是这么少，更多的依赖中间件处理，所以代码也更简洁，gulp 还有很多有趣的地方我没发现，希望后续学习可以发掘更多 gulp 的优点。

> 官方中文文档：http://www.gulpjs.com.cn/docs/api/
