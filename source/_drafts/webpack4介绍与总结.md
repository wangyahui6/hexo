---
title: webpack4介绍与总结
date: 2018-07-29 11:13:58
description: webpack是目前最火的打包工具，本文以最新的webpack4为背景，从概念方面介绍webpack是什么、能给我们带来什么好处以及如何配置。
categories: build
tags: webpack build
---

由于web应用扩展地得极其迅猛，前端技术也是日新月异，前端的苦不是有多难学，而是我刚学完，这东西就被淘汰了（手动哭脸）。框架方面我们有vue、react、angular，我们需要写vue单文件，也需要写jsx语法；js方面我们有Typescript、es6(7、8、9)，现在是每年一个小版本的迭代，语法也是在不断更新和淘汰；模块方面我们有es6的modlue、CMD、AMD；css方面我们有sass、less、postcss。新技术层出不穷，浏览器实现跟不上，还要考虑到浏览器兼容性问题，是一个令人头大的问题。但是webpack可以轻松帮我们解决这些问题，我们可以在webpack的世界中放心地使用上述提到的技术，以更方便、舒服的方式完成我们开发。

## webpack是什么

`At its core, webpack is a static module bundler for modern JavaScript applications. `这是官网的定义，webpack就是一个静态模块的打包工具。webpack可以将工程中的静态资源根据我们声明的依赖关系，打包出最后的正常运行的输出文件。官网显示的这幅图很形象地描述了这个过程：
![打包过程](/images/webpack4/bundler.png)

在webpack中，所有的静态资源都可以被处理为一个模块，包括js、图片、css、字体。模块化是前端开发的主要模式，模块化的好处有很多，我想到的有以下3种：

- 更轻松地拆分代码逻辑，对于大型工程尤其重要
- 更容易地去管理变量，模块会自动生成一个局部变量环境，减少全局变量的污染
- 显示地声明依赖关系，想想之前在head中引入很多script标签，由于script顺序错误而引起的bug

在es6之前，原生js是不支持模块化开发。因为js设计之初，只是为了实现表单验证等简单的交互，并没有考虑到要用js来写大型的web应用。随着js承担地职责越来越大，模块化开发的需求越来越急迫。于是，js社区诞生出了CMD、AMD这样的js模块化标准，随后在es6中，也终于加入了module的原生支持。现在最新版本的各大浏览器均已实现了对es6的module语法支持，但也仅限于最新的几个版本，详情可以查看一下[can i ues](https://caniuse.com/#search=import)。我们可以把webpack当成是模块化标准的实现方案，但webpack的功能不仅限于此。

webpack支持多种模块使用方式，包括es6的module、CMD、AMD。推荐使用es6的module语法，一方面是因为它是标准，是以后模块化语法的主要使用方式；另一方面，是因为它是静态的，webpack可以依靠静态分析，做一些优化，比如Treeshaking。webpack自带js模块处理功能，其他类型的静态资源我们需要通过配置相应的loader去处理。



## 概念

### 入口和出口

