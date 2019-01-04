---
title: 浏览器缓存和webpack缓存配置
author: 王亚辉
description: 网络请求会耗费大量时间和请求，如果可以重用为改变的网络资源，对于用户来说可以更快更流畅的查看网页，对于服务器来说减少了很多负荷，所以浏览器缓存是前端优化的重要内容。本文介绍了浏览器缓存的机制和缓存在webpack中的应用。
categories: 前端
date: 2017/02/10
tags:
 - 缓存
 - webpack
---
## 浏览器缓存
浏览器缓存分为两种类型：
- **强缓存**：也称为本地缓存，不向服务器发送请求，直接使用客户端本地缓存数据
- **协商缓存**：也称304缓存，向服务器发送请求，由服务器判断请求文件是否发生改变。如果未发生改变，则返回304状态码，通知客户端直接使用本地缓存；如果发生改变，则直接返回请求文件。

浏览器缓存机制的过程如下：

![浏览器缓存机制](/images/cache_process.png)
## 强缓存(本地缓存)
强缓存是最彻底的缓存，无需向服务器发送请求，通常用于css、js、图片等静态资源。浏览器发送请求后会先判断本地是否有缓存。如果无缓存，则直接向服务器发送请求；如果有缓存，则判断缓存是否命中强缓存，如果命中则直接使用本地缓存，如果没命中则向服务器发送请求。判断是否命中本地缓存的方法有两种：**Expires**和**Cache-Control**。
### Expires
Expires是http1.0的响应头，代表的含义是资源本地缓存的过期时间，由服务器设定。服务器返回给浏览器的响应头中如果包含Expires字段，浏览器发送请求时拿当前时间和Expires字段值进行比较，判断资源缓存是否失效。如下图所示：

![Expires](/images/expires.png)

*Date*代表请求资源的时间，*Expires*代表资源缓存的过期时间，可以看到服务器设置资源的缓存时间为5分钟。2017-02-10 07:53:19之前，请求这个资源就是命中本地缓存。超过这个时间再去请求则不命中。
### Cache-Control
Cache-Control是http1.1中新增的字段。由于Expires设置的是资源的具体过期时间，如果服务器时间和客户端时间不一样，就会造成缓存错乱，比如人为调节了客户端的时间，所以设置资源有效期的时长更合理。http1.1添加了Cache-Control的max-age字段。max-age代表的含义是资源有效期的时长，是一个相对时长，单位为s。

![Expires](/images/cache_control.png)

Cache-Control: max-age = 300设置资源的过期时间为5分钟。浏览器再次发送请求时，会把第一次请求的时间和max-age字段值相加和当前时间比较，以此判断是否命中本地缓存。max-age使用的都是客户端时间，比Expires更可靠。如果max-age和Expires同时出现，max-age的优先级更高。Cache-Control提供了更多的字段来控制缓存：

- no-store,不判断强缓存和协商缓存，服务器直接返回完整资源
- no-cache,不判断强缓存，每次都需要向浏览器发送请求，进行协商缓存判断
- public,指示响应可被任何缓存区缓存
- private,通常只为单个用户缓存，不允许任何共享缓存对其进行缓存,通常用于用户个人信息

## 协商缓存
协商缓存的判断在服务器端进行，判断是否命中的依据就是这次请求和上次请求之间资源是否发生改变。未发生改变命中，发生改变则未命中。判断文件是否发生改变的方法有两个：**Last-Modified、If-Modified-Since**和**Etag、If-None-Match**。
### Last-Modified、If-Modified-Since
Last-Modified是http1.0中的响应头字段，代表请求的资源最后一次的改变时间。If-Modified-Since是http1.0的请求头，If-Modified-Since的值是上次请求服务器返回的Last-Modified的值。浏览器第一次请求资源时，服务器返回Last-Modified,浏览器缓存该值。浏览器第二次请求资源时，用于缓存的Last-Modified赋值给If-Modified-Since，发送给服务器。服务器判断If-Modified-Since和服务器本地的Last-Modified是否相等。如果相等，说明资源未发生改变，命中协商缓存；如果不相等，说明资源发生改变，未命中协商缓存。

![Last-Modified](/images/last_modified.png)

可以看到该请求返回的是304状态码，说明资源的Last-Modified未改变，所以这次请求的Last-Modified和If-Modified-Since是一致的。
### Etag、If-None-Match
Last-Modified、If-Modified-Since使用的都是服务器提供的时间，所以相对来说还是很可靠的。但是由于修改时间的精确级别或者定期生成文件这种情况，会造成一定的错误。所以http1.1添加Etag、If-None-Match字段，完善协商缓存的判断。Etag是根据资源文件内容生成的资源唯一标识符，一旦资源内容发生改变，Etag就会发生改变。基于内容的标识符比基于修改时间的更可靠。If-None-Match的值是上次请求服务器返回的Etag的值。Etag、If-None-Match的判断过程和Last-Modified、If-Modified-Since一致，Etag、If-None-Match的优先级更高。

## 工程中遇到的问题
强缓存的优势很明显，无需向服务器发送请求，节省了大量的时间和带宽。但是有一个问题，缓存有效期内想更新资源怎么办？我在工程中还遇到另外一个问题，一个项目有四个环境，测试环境、开发环境、在线确认环境、在线环境，四个环境的域名相同，这样就会造成四个环境的缓存共用问题。比如先访问了测试环境，index.js被换成到浏览器中，再切换到在线环境，在线环境会请求index.js,此时浏览器就会使用本地缓存中测试环境的index.js,造成代码错乱。

如何使强缓存失效，是问题的关键。通常的解决方法是更新文件名，文件名不一样的话，浏览器就会重新请求资源。我们要保证新发布版本和不同环境中的文件名是不一样的。其中一种方法在文件名后加版本号：

```js
index.js?version=1
index.css?version=1
```
webpack提供了很简单的方法可以配置缓存。
```js
// webpack.config.js
module.exports = {
  entry: "main.js",
  output: {
    path: "/build",
    filename: "main.[hash].js"
  }
};
```
通过hash占位符，在每次生成打包文件时，都会通过文件内容生成唯一的hash，并添加到输出的文件名中。如果有多个入口文件，可以使用name占位符设置输出：
```js
// webpack.config.js
module.exports = {
  entry: {
      main:"main.js",
      sub:"sub.js"
  },
  output: {
    path: "/dist",
    filename: "[name].[hash].js"
  }
};
```
这时候有一个问题是，此时的hash是根据两个文件的内容来生成的，两个文件名使用的hash是一致的。如果main.js和sub.js只有一个改变，两个文件名都会改变，两个文件都会重新请求，造成资源浪费。webpack提供了chunkhash来代替hash在多入口情况下使用。chunkhash是根据每个入口文件单独生成的哈希值，避免上述情况。

webpack打包动态生成文件名，我们需要动态地把文件引用插入到html启动文件中。[html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)可以帮我很好地解决这个问题。`html-webpack-plugin`可以动态地生成一个html文件，并在html文件中动态插入webpack打包生成的资源文件。
```js
var HtmlWebpackPlugin = require('html-webpack-plugin');
var webpackConfig = {
  entry: 'main.js',
  output: {
    path: '/dist',
    publicPath: '/dist',
    filename: 'main.[hash].js'
  },
  plugins: [new HtmlWebpackPlugin()]
};
```
默认在`webpackConfig.output.path`路径下生成`index.html`,生成的html文件如下：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Webpack App</title>
  </head>
  <body>
    <script src="main.2a6c1fee4b5b0d2c9285.js"></script>
  </body>
</html>
```
通常html启动文件都有自定义的内容，所以`html-webpack-plugin`提供了模板功能，template字段设置模板的路径，`html-webpack-plugin`以template为模板，动态添加webpack打包生成的资源路径。
```js
var HtmlWebpackPlugin = require('html-webpack-plugin');
var webpackConfig = {
  entry: 'main.js',
  output: {
    path: '/dist',
    publicPath: '/dist',
    filename: 'main.[hash].js'
  },
  plugins: [new HtmlWebpackPlugin(
      {
          template:'index.html'
      }
  )]
};
```
原index.html内容（\index.html）：
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>stat-front</title>
    <link rel="stylesheet" href="//at.alicdn.com/t/font_ejl5slgdvtg74x6r.css">
  </head>
  <body>
    <div id="app" class="app-root">
        <router-view></router-view>
    </div>
    <!-- built files will be auto injected -->
  </body>
</html>
```
生成的index.html内容（\dist\index.html）：
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>stat-front</title>
    <link rel="stylesheet" href="//at.alicdn.com/t/font_ejl5slgdvtg74x6r.css">
  </head>
  <body>
    <div id="app" class="app-root">
        <router-view></router-view>
    </div>
    <!-- built files will be auto injected -->
    <script src="main.2a6c1fee4b5b0d2c9285.js"></script>
  </body>
</html>
```
最开始的时候静态的index.html在根目录下，`webpack-dev-server`设置的启动路径就是根目录下的index.html,如果要启动生成的index.html，还需要设置`webpackConfig.output.publicPath`：
```js
var HtmlWebpackPlugin = require('html-webpack-plugin');
var webpackConfig = {
  entry: 'main.js',
  output: {
    path: '/dist',
    publicPath: '/',
    filename: 'main.[hash].js'
  },
  plugins: [new HtmlWebpackPlugin(
      {
          template:'index.html'
      }
  )]
};
```
这样webpack-dev-server在内存中生成的资源都存放在根目录下，生成的index.html会代替原index.html启动。