---
title: webpack4介绍与总结
categories: build
tags: webpack build
description: webpack是目前最火的打包工具，本文以最新的webpack4为背景，从概念方面介绍webpack是什么、能给我们带来什么好处以及如何配置。
date: 2018-07-29 11:13:58
---


由于web应用扩展地得极其迅猛，前端技术也是日新月异，前端的苦不是有多难学，而是我刚学完，这东西就被淘汰了（手动哭脸）。框架方面我们有vue、react、angular，我们需要写vue单文件，也需要写jsx语法；js方面我们有Typescript、es6(7、8、9)，现在是每年一个小版本的迭代，语法也是在不断更新和淘汰；模块方面我们有es6的modlue、CommonJS、AMD；css方面我们有sass、less、postcss。新技术层出不穷，浏览器实现跟不上，还要考虑到浏览器兼容性问题，是一个令人头大的问题。但是webpack可以轻松帮我们解决这些问题，我们可以在webpack的世界中放心地使用上述提到的技术，以更方便、舒服的方式完成我们开发。

## webpack是什么

`At its core, webpack is a static module bundler for modern JavaScript applications. `这是官网的定义，webpack就是一个静态模块的打包工具。webpack可以将工程中的静态资源根据我们声明的依赖关系，打包出最后的正常运行的输出文件。官网显示的这幅图很形象地描述了这个过程：
![打包过程](/images/webpack4/bundler.png)

在webpack中，所有的静态资源都可以被处理为一个模块，包括js、图片、css、字体。模块化是前端开发的主要模式，模块化的好处有很多，我想到的有以下3种：

- 更轻松地拆分代码逻辑，对于大型工程尤其重要
- 更容易地去管理变量，模块会自动生成一个局部变量环境，减少全局变量的污染
- 显示地声明依赖关系，想想之前在head中引入很多script标签，由于script顺序错误而引起的bug

在es6之前，原生js是不支持模块化开发。因为js设计之初，只是为了实现表单验证等简单的交互，并没有考虑到要用js来写大型的web应用。随着js承担地职责越来越大，模块化开发的需求越来越急迫。于是，js社区诞生出了CommonJS、AMD这样的js模块化标准，随后在es6中，也终于加入了module的原生支持。现在最新版本的各大浏览器均已实现了对es6的module语法支持，但也仅限于最新的几个版本，详情可以查看一下[can i ues](https://caniuse.com/#search=import)。我们可以把webpack当成是模块化标准的实现方案，但webpack的功能不仅限于此。

webpack支持多种模块使用方式，包括es6的module、CommonJS、AMD。推荐使用es6的module语法，一方面是因为它是标准，是以后模块化语法的主要使用方式；另一方面，是因为它是静态的，webpack可以依靠静态分析，做一些优化，比如Treeshaking。webpack自带js模块处理功能，其他类型的静态资源我们需要通过配置相应的loader去处理。

## 概念和配置

webpack中最重要的概念有以下几个：

- Entry, 工程的入口文件配置
- Output, 打包的输出的文件配置
- Chunk, webpack处理和输出的包
- Loaders, 加载器，用于处理各种不同类型的模块，可扩展
- Plugins, 插件，在webpack打包过程中不同时机执行一些任务，比如清除打包目录、复制静态文件、抽取css文件
- Mode, 区分开发环境和生成环境

webpack一般根据配置文件去执行打包任务，我们创建一个webpack.config.js文件来编写我们的打包配置。

### Entry

Entry，顾名思义就是工程的入口文件，Entry的配置写法有三种：

- 对象，可配置多入口，可配置chunk名，灵活可扩展，最常用,一个属性就是一个entry chunk

```js
module.exports = {
  entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }
};
```

- 字符串, 最简单直接方式，单入口，chunk名默认为main

```js
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```
- 数组, 多入口，将多个入口文件打包为一个chunk，chunk名默认为main

```js
module.exports = {
  entry: ['./path/to/my/entry/file.js', './path/to/my/entry/file1.js']
};
```

入口一般用对象写法即可，其他两种写法的可忽略。

### Output

Output用于配置打包输出的文件，包括输出文件的文件名、输出路径、静态资源地址，这里列出最常用的4种：

```js
module.exports = {
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: 'js/[name].js',
    chunkFilename: 'js/[name].js',
    path: __dirname + '/dist',
    publicPath: 'http://cdn.example.com/assets/[hash]/'
  }
};
```
配置项如下：

- filename: 配置输出文件名，可添加路径配置(例子中js/)，可使用占位符，占位符有以下5种：
    - name: chunk名，在该例子中就是app和search
    - hash: 模块标识符的hash值，跟工程内容相关
    - chunkhash: chunk内容的hash值，只和当前chunk内容相关,可用于缓存设置
    - id: 模块标识符
    - query：模块查询参数，取文件名中`?`后面的内容


- path: 文件的输出路径，必须是绝对地址

- publicPath: 用于设置打包过程中产生的静态文件的最终引用地址，静态文件的最终引用地址为`output.publicPath + output.filename`，很多时候，你的静态文件放置在CDN上，通过publicPath就可以很方便地设置。如果你的静态引用地址在运行时才能确定，可以在入口文件中设置`__webpack_public_path__ `来决定publicPath的值:
    ```js
    __webpack_public_path__ = myRuntimePublicPath;

    // rest of your application entry
    ```

- chunkFilename: 用于设置非entry入口的chunk的输出文件名，非entry入口的chunk一般在动态引入和`CommonsChunkPlugin`中产生，这是一个Plugin，用于抽取公共代码或者进行代码分割等操作，该plugin已经在webpack4中废除，由webpac4内置的`optimization.splitChunks`替代，后面会讲到

output还有其他很多配置，这4个是常用配置。

### Loaders

Loaders可以理解为不同类型模块的处理器，将这些类型的模块处理为浏览器可运行和识别的代码。比如babel-loader将es6以上代码转换为es5代码；sass-loader将sass代码解析为css代码；url-loader和file-loader可以将图片、字体等静态文件解析为base64码或者静态文件地址。Loaders给我们提供了处理模块的入口，在里面可以使用全部的js功能，从而使webpack具有了强大而灵活的能力。webpack及webpack社区提供了功能强大的loader供开发者使用，你也可以自己编写loader。下面介绍一下在工程中常用的loader。

#### js Loaders

##### babel-loader

使用[babel](https://babeljs.io/)将ES2015+的代码转码为ES5的代码，babel的具体配置可参考babel官网

```js
modlue: {
    rules: [
        {
             test: /\.js$/,
             exclude: /(node_modules|bower_components)/,
             use: 'babel-loader
        }
    ]
}
```
exclude表示不处理的目录，一般`node_modules`中的第三方js文件不在我们的处理范围内。

##### script-loader

该loader使对应的js文件在全局环境中运行一次。比如我们在工程中使用了jquery的插件，需要全局暴露$,我们就需要让jquery文件在全局环境中运行，以便让它把$挂载到全局变量中,效果和在浏览器中加script标签一样。

```js
import 'jquery';

module: {
    rules: [
        {
        test: /jquery$/,
        use: [ 'script-loader' ]
        }
    ]
}
```
#### css Loaders

- style-loader：将css模块以style标签的形式加入到html中

- css-loader：主要用来解析css中的静态资源，将@import和url()解析为import/require()，解析出的除css以外的静态资源，一般交给url-loader和file-loader去处理

- postcss-loader：可以对css进行各种处理，功能强大，比如自动添加css前缀，也可自定义插件

- sass-loader/less-loader：将sass/less代码转换为css

解析一个sass文件，并不只需要一个loader，它需要多个loader串行处理，webpack可以配置多个loader串行处理：

```js
module: {
    rules: [
        {
        test: /\.sass$/,
        use: [ 'style-loader', 'css-loader', 'postcss-loader', 'sass-loader' ]
        }
    ]
}
```

我们可以将use配置为一个数组，loader从右往左依次执行，且前一个loader的结果是下一个loader的输入。最后一个loader的输出就是我们最终要的结果。一个sass文件首先经过`sass-loader`处理，变成css文件，又经过`postcss-loader`处理，添加浏览器前缀等功能，接着交给`css-loader`去解析css文件引用的静态变量，最后由`style-loader`以`script`标签的形式加入到html中。

#### Files Loaders

`url-loader`和`file-loader`是一对用来处理图片、svg、视频、字体等静态资源文件的loader。一般体积比较小的资源交给`url-loader`处理，编码为base64字符串，直接嵌入js文件中。体积较大的文件由file-loader处理，直接释放为了一个输出文件。

```js
{
    test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
    loader: 'url-loader',
    options: {
        limit: 10000,
        name: 'img/[name].[hash:7].[ext]'
    }
},
```
一般只配置`url-loader`即可，在资源超过limit的时候，`url-loader`会将资源自动交给`file-loader`处理，并将options内容也传递给file-loader。

### Plugins

loaders用来转换某种特定类型的module，plugins则用来在一些合适的时机执行一些特定的任务，比如代码分割、静态资源处理、环境变量的注入、将所有css的module抽取为单个文件等。webpack自身也是用插件系统构建起来的。插件的目的是做任何loaders做不了的事情。

#### HtmlWebpackPlugin

HtmlWebpackPlugin插件可以用来生成包含你所有打包文件（js和css）的html文件，特别是你在打包文件名配置了hash，就不得不用这个插件了。

```
module.exports = {
  plugins: [
      new HtmlWebpackPlugin({
          template: path.resolve(__dirname, 'src/index.html'),  
          filename: 'static/index.[hash].html',  
          inject: true, 
          minify: false, 
          chunks: ['app', 'vendor'] 
      })
  ]
};
```

- template: HtmlWebpackPlugin生成html文件的模板，如果简单的话可以直接通过其他配置项生成，不必单独提供一个html文件模板，详情可参考[HtmlWebpackPlugin](https://github.com/jantimon/html-webpack-plugin)
- filename: 输出的html文件名，规则和output的filename相同
- inject: 是否注入打包的文件，包括js和通过MiniCssExtractPlugin打包输出的css文件，默认为true
- minify: 是否压缩
- chunks: 只注入某些特定chunk的输出文件，在多文件场景下比较有用

#### MiniCssExtractPlugin

`MiniCssExtractPlugin`将一个chunk中的css抽取为一个单独的css文件，如果chunk中不包含css，则不生成文件。且支持按需加载和sourceMap。webpack4新增插件，在webpack4之前是使用`ExtractTextWebpackPlugin`来做这件事。官方文档总结了`MiniCssExtractPlugin`相对`ExtractTextWebpackPlugin`的四个优势：

- 按需异步加载
- 没有重复编译（性能）
- 使用更简单
- 专门为css设计

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const devMode = process.env.NODE_ENV !== 'production'

module.exports = {
  plugins: [
    new MiniCssExtractPlugin({
      // Options similar to the same options in webpackOptions.output
      // both options are optional
      filename: "[name].css",
      chunkFilename: "[id].css"
    })
  ],
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          devMode ? 'style-loader' : {  // MiniCssExtractPlugin目前还没有HMR功能，所以最好只在生成环境使用
            loader: MiniCssExtractPlugin.loader,
            options: {
              publicPath: '../' // 可单独配置publicPath,默认采用output中的publicPath
            }
          },
          "css-loader"
        ]
      }
    ]
  }
}
```

#### CopyWebpackPlugin

CopyWebpackPlugin用来处理静态文件，可以将文件或者文件夹原封不动地移动到打包目录。

```js
const CopyWebpackPlugin = require('copy-webpack-plugin')

const config = {
  plugins: [
    new CopyWebpackPlugin([
        { from: 'source', to: 'dest' },
        { from: 'source', to: 'dest', toType: 'dir|file|template' }, // 手动设置to的类型，比如设置成dir，即使to设置为a.js,最后也会生成a.js文件夹
        { from: 'source', to: 'dest', context: '/app' }, // 基准目录，from相对于context解析
        { from: 'source', to: 'dest', ignore: ['*.js'] }, // ignore, 忽略匹配的文件
        'source' // 只有from，to默认为output的path
     ], {
         context: '/app', // 同上
         ignore: ['*.js'], // 同上
     })
  ]
}
```

#### CleanWebpackPlugin

CleanWebpackPlugin用来清除打包目录，主要用于每次重新生成的时候，清除残留文件。在文件有hash值的情况下，是必要的。

```js
const CleanWebpackPlugin = require('clean-webpack-plugin');
const config = {
  plugins: [
    new CleanWebpackPlugin(['dist', 'bulid/*.js'], {
        watch: false, // 是否在--watch模式下也清除files然后重新编译，默认为false
        exclude: [ 'files', 'svg', 'css' ] // 不删除的子目录和文件
    }),
  ]
}
```
#### DefinePlugin

DefinePlugin用来定义webpack编译期间的全局变量。我们可以根据这些变量，来做不同的动作。最典型的就是可以区分开发环境和生产环境，比如在开发环境打印各种警告、错误，在生产环境去掉这些跟业务无关的代码。

DefinePlugin的参数是一个对象，键名是一个标识符，或者用`.`隔开的多级标识符，参数遵循以下规则

- 如果参数值是一个字符串，则被当做代码块
- 如果参数值不是字符串，则会自动转换为字符串
- 如果参数值是一个对象，则对象的每个值都按照上述规则被定义为全局变量
- 如果键名前面有typeof，则只是定义typeof的调用
```js
new webpack.DefinePlugin({
 'process.env.NODE_ENV': JSON.stringify('production'),
  PRODUCTION: JSON.stringify(true),
  VERSION: JSON.stringify('5fa3b9'),
  BROWSER_SUPPORTS_HTML5: true,
  TWO: '1+1',
  'typeof window': JSON.stringify('object')
});
```
index.js
```js
console.log('PRODUCTION', PRODUCTION);
console.log('VERSION', VERSION);
console.log('BROWSER_SUPPORTS_HTML5', BROWSER_SUPPORTS_HTML5);
console.log('TWO', TWO);
console.log('Object', typeof window);
```

被编译为
```js
console.log('PRODUCTION', true);
console.log('VERSION', "5fa3b9");
console.log('BROWSER_SUPPORTS_HTML5', true);
console.log('TWO', 1+1);
console.log('Object',  false ? undefined : _typeof(window));    // 不太明白
```

DefinePlugin的原理很简单，只是在编译过程中遇到这些定义好的键名，就用键值做简单的文本替换。所以，你如果想给全局变量赋一个字符串，需要这样写`'"production"'`,一般使用`JSON.stringify`来转一下。

### Mode

webpack4新增了`mode`配置。webpack会根据`mode`值自动帮你做一个不同的优化:

- production（默认值）
  - 在`DefinePlugin `中将`process.env.NODE_ENV`设置为`production`
  - 默认启用了如下插件：`FlagDependencyUsagePlugin`, `FlagIncludedChunksPlugin`, `ModuleConcatenationPlugin`, `NoEmitOnErrorsPlugin`, `OccurrenceOrderPlugin`, `SideEffectsFlagPlugin` and `UglifyJsPlugin`

- development:
  - 在`DefinePlugin `中将`process.env.NODE_ENV`设置为`development`
  - 默认启用了如下插件：`NamedChunksPlugin `, `NamedModulesPlugin`

- none: 什么都不做

`mode`的两种使用方式

- 配置：

```js
module.exports = {
  mode: 'production'
};
```
- 命令行：

```js
webpack --mode=production
```
## 代码分割

代码分割可以把代码按照一定的逻辑分割，用来做按需加载或者并行加载，以减少加载时间。在webpack中，主要有以下3中方式，实现代码分割：

- Entry Points: 手动在`entry`入口配置处配置多个入口
- SplitChunks: webpack4默认带这个优化插件，用于抽取不同chunk的公共部分，或者直接代码分割
- Dynamic Imports: 动态加载，在函数中使用`import()`语法，webpack使用`SplitChunks`将`import()`加载的module分割为一个单独的chunk，并在函数执行时加载该chunk对应的js文件。

第一种没什么好说的，主要说一下后两种。

### SplitChunks

在webpack4之前的版本，都是使用`CommonsChunkPlugin`来抽取公共chunk，在webpack4中废弃了`CommonsChunkPlugin`，转而支持内置的`optimization.splitChunks`。

`splitChunks`默认只对按需加载的chunk起作用，会自动将import()引入的module分割为单独chunk，但是需要满足以下条件：

- 新chunk中的模块被共享或者来自`node_modules`
- 新chunk必须大于30kb(before min+gz)
- 按需加载的chunks并行加载的数量要小于等于5
- 入口文件的并行加载的请求应该小于等于3

在执行后两个原则的时候，体积大的chunk被优先生成。

`splitChunks`的默认配置如下,我们可以看出正好是和上面4个条件对应的：

```js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'async',
      minSize: 30000, // chunk只有超过这个大小才会被分割
      maxSize: 0, // 大于这个体积的chunk会被自动分割为更小的chunk
      minChunks: 1, // 一个模块被共享的chunk数量大于minChunks时，才会被分割出来
      maxAsyncRequests: 5, // 按需加载最大的并行数
      maxInitialRequests: 3, // 初始加载最大的并行数
      automaticNameDelimiter: '~', // name为true时，新chunk的文件名由cacheGroups的key加上chunks属性的一些信息生成，automaticNameDelimiter是分隔符
      name: true,
      cacheGroups: {  // 配置拆分规则，会继承splitChunks所有的配置项，所有splitChunks配置项都可以在这里重写覆盖，test、prioprity、reuseExistingChunk是cacheGroups独有的属性
        vendors: {
          test: /[\\/]node_modules[\\/]/, // 模块匹配规则，可以是正则表达式或者函数，不写默认选择所有模块
          priority: -10 // 优先级，当同一个模块同时包含在不同cacheGroup中，该模块将被划分到优先级高的组中
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true  // 如果该chunk包含的modules都已经另一个被分割的chunk中存在，那么直接引用已存在的chunk，不会再重新产生一个
        }
      }
    }
  }
};
```
#### chunks

`splitChunks`的chunks属性表示作用的chunk范围。chunks可以是一个函数，完全由开发者控制；也可以是一个字符串，字符串一共有3个值：

- initial,表示只作用于入口chunk
- async，表示只作用于动态引入的异步chunk
- all，所有chunk

#### name

name属性表示最后生成chunk的文件名，有以下3中类型取值：

- boolean，设置为`true`，则会根据cacheGroup的key和chunks属性的信息自动生成
- string，如果设置在`splitChunks`下，那所有chunk设置为相同的名称，这会造成不同的chunk合成为一个chunk,当然你可以设置在`cacheGroup`下
- function，完全由开发者控制

## Tree Shaking

`Tree Shaking`这次词很形象，摇树，把烂掉的树叶摇下来。我们工程中的`烂树叶`就是那些我们导出了，但是没有用到的代码，`Tree Shaking`可以帮助我们去除无效代码，减小打包体积，有其在大工程中，效果明显。

`Tree Shaking`的使用三部曲：

### 静态的模块语法

使用`Tree Shaking`的第一个前提条件就是必须使用ES2015的模块语法，`import`,`export`。因为ES2015的模块语法是`静态加载`的，而CommonJS和AMD都是`动态加载`。`静态加载`是指在编译阶段你就能确定导出和加载的内容。ES2015在语法上规定：你只能在顶层模块作用域进行导入导出操作，不允许在条件语句中导入导出，也不允许导入和导出的内容有变量。这意味着你只分析源码就可以确定导入和导出的内容，而不是等到运行时才能确定。比如说下面`CommonJS`的语法，在ES6中就是不能使用的：
```js
var my_lib;
if (Math.random()) {
    my_lib = require('foo');
} else {
    my_lib = require('bar');
}
```
你只有在运行的时候，才知道到底加载是的`foo`还是`bar`。ES6强制模块语法静态化，失去了一定的灵活性，但是带来了更多的好处，其中之一就是我们可以通过静态分析去实现Tree Shaking。

### sideEffects

在完全的Es6模块世界中，代码是没有副作用的，但是现在我们可能用到的各种地方库会有副作用。`副作用`是指代码除了导入和导出，还做了一些其他影响了其他代码的行为，比如定义了全局变量。最典型的例子就是polyfills,比如Promise的polyfills就定义了Promise全局变量。这时候如果我们分析到Promise有未使用的导出代码，则不能删除，否则可能会影响Promise的使用。哪些是有`副作用`的代码，需要你识别，并且告诉webpack，方式就是通过设置sideEffects,可以设置在package.json文件中，也可以设置的module.rules中。
```js
{
  "name": "your-project",
  "sideEffects": [
    "./src/some-side-effectful-file.js",
    "*.css"
  ]
}
```

### UglifyJSPlugin

webpack会通过静态分析找到冗余代码，并打上标记，我们看一下官网例子：

math.js

```js
export function square(x) {
  return x * x;
}

export function cube(x) {
  return x * x * x;
}
```

index.js

```js
 import { cube } from './math.js';

  function component() {
   var element = document.createElement('pre');

   element.innerHTML = [
     'Hello webpack!',
     '5 cubed is equal to ' + cube(5)
   ].join('\n\n');

    return element;
  }

  document.body.appendChild(component());
```

在`development`模式下打包，内容如下：

```js
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
  'use strict';
  /* unused harmony export square */
  /* harmony export (immutable) */ __webpack_exports__['a'] = cube;
  function square(x) {
    return x * x;
  }

  function cube(x) {
    return x * x * x;
  }
});
```

我们可以看到代码中并未用到`math.js`中的`square`方法，webpack输出文件中表示它未被使用。如果想去掉未被使用的代码，则需要用到`UglifyJSPlugin`插件，它是用来压缩js文件的，自动启用了去除冗余代码的功能。我们可以在`production`模式下打包，会发现`square`的代码已经被去除。