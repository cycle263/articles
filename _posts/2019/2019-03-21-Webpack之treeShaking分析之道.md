---
layout:     post
title:      "Webpack之treeShaking分析之道"
subtitle:   "treeshaking"
date:       2019-03-21
author:     "Cycle263"
header-img: "img/article-bg.jpg"
tags:
    - 前端开发
---

## treeshaking

tree shaking 字面可以理解为摇树，所做的优化就是，过滤不必要和未用到的打包。


* 背景

Tree shaking一直是一个美丽而遥不可及的话题，它是一个术语，通常用于描述移除 JavaScript 上下文中的未引用代码。这个术语和概念实际上是兴起于 ES2015 模块打包工具 rollup。可以简单地理解为摇树，抖落掉枯萎无用的树叶。

* side effects

影响tree shaking的根本原因在于side effects（副作用），其中最广为人知的一条side effect就是动态引入依赖的问题。ES6其实也提供import()方法支持动态引入依赖，所以以下写法其实也是完全可行的。

Webpack 4官方提供了sideEffects属性，通过将其设置为false，可以主动标识该类库中的文件只执行简单输出，并没有执行其他操作，可以放心shaking。除了可以减小bundle文件的体积，同时也能够提升打包速度。为了检查side effects，Webpack需要在打包的时候将所有的文件执行一遍。而在设置sideEffects之后，则可以跳过执行那些未被引用的文件。

webpack 4之前，可以考虑在babel中开启loose模式，或者去掉babel-loader，也就是说不需要 babel 把 ES Module 转换成曾经的 commonjs 模块了，要使用 Tree Shaking，关闭 babel 默认的模块转义，避免babel转化产生新的副作用，导致shaking not working。然后webpack打包结束后，再执行babel编译文件。

* 依赖包shaking

目前大量的第三方库，都采用了同时提供两份代码，一份commonjs模块化代码，一份ES6模块化代码，并在package.json里指出两个入口。`jsnext:main`作为ES6模块化代码的入口已经成为社区的一个默认约定。

```js
// redux为例，有两份代码
node_modules/redux/es/index.js
node_modules/redux/lib/index.js

// package.json
{
  "main": "lib/index.js",
  "jsnext:main": "es/index.js", 
}

// webpack.config.js
{
  resolve: {
    "mainFields": ["main", "jsnext:main"]   // 配置顺序决定优先用哪份代码
  }
}
```

* **失效情况**

必须采用ES6模块化语法的代码，因为ES6模块化的代码是静态的，webpack才能分析出需要剔除哪些代码。