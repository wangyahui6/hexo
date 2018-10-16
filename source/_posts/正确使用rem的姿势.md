---
layout: drafts
title: 正确使用rem的姿势
tags: css
categories: 前端
date: 2018-09-04 14:17:01
---
移动端的屏幕大小各异,尤其是安卓机,千奇百怪,各种尺寸的机型都有,而且有1倍屏,2倍屏，3倍屏之分，作为精益求精的前端，我们希望找到一种完美适配各种机型的方案。rem是现在主流的移动端自适应布局方案，本文主要介绍了rem布局的原理和通用方案
<!-- more -->
## 原理

首先说一下，我们想要达到的自适应效果是什么。很简单：元素、字体大小能随着屏幕大小（一般相对于宽度来说）变化而变化，在大一点的屏幕上，尺寸大一些；在小一点的屏幕上，尺寸小一些；尺寸的大小和屏幕大小成正比。

最简单最直接的方案就是直接用百分比设置元素的尺寸。我们用百分比设置元素大小可以实现元素尺寸的自适应，但是无法实现字体大小的自适应，而且尺寸转化为百分比计算很麻烦，还有就是元素尺寸的高很难相对屏幕宽度设置百分比。百分比适用于某种具体场景，不是通用解决方案。

其实我们需要的是一个和屏幕宽度正相关的单位，而且这个单位要和px很容易互相转化。这样我们就可以使用这种单位进行元素尺寸和字体大小的设置。那么css中存在这种单位吗？答案是：不存在的。。。不过不要灰心，我们可以借助rem来实现这种我们需要的单位。

rem是一个相对单位，1rem等于html元素上字体设置的大小。我们只要设置html上font-size的大小，就可以改变rem所代表的大小。这样我们就有了一个可控的统一参考系。我们现在有两个目标：

- rem单位所代表的尺寸大小和屏幕宽度成正比，也就是设置html元素的font-size和屏幕宽度成正比
- rem单位和px单位很容易进行换算，方便我们按照标注稿写css

这里有一个前提，无论是设置html的font-size和屏幕宽度成正比，还是换算单位，我们都是以我们的标注稿为参考的。移动端的标注稿一般是640px(iphone5)或者750px(iphone6/7/8),现在750px用的比较多一些，我们假设标注稿是750px的。这里的750px是指设备的实际尺寸，也是UI标注稿的实际尺寸。而我们编码写的px是指css尺寸，是设备无关的尺寸，css尺寸和屏幕实际尺寸不是1比1的映射关系，而是取决于屏幕的像素密度。比如iphoneX是3倍屏，iphone8是2倍屏，但是两个的屏幕css尺寸都是375px。而实际的设备尺寸，iphonex是1125px，iphone8是750px，我们编码过程中只需要设置css尺寸，设备会自动帮我们映射实际的尺寸。我们按照标注稿写完页面之后，页面应该是可以在其他所有尺寸设备上正常自适应地显示的。

### rem单位所代表的尺寸大小和屏幕宽度成正比

首先，设置rem单位所代表的尺寸大小和屏幕宽度成正比，有3中方案,先不必纠结其中的数值：

- 媒体查询, 设定每种屏幕对应的font-size
```css
@media screen and (min-width:240px) {
    html, body, button, input, select, textarea {
        font-size:9px;
    }
}
@media screen and (min-width:320px) {
	html, body, button, input, select, textarea {
		font-size:12px;
	}
}
// 红米Note2
@media screen and (min-width:360px) {
	html, body, button, input, select, textarea {
		font-size:13.5px;
	}
}
@media screen and (min-width:375px) {
	html, body, button, input, select, textarea {
		font-size:14.0625px;
	}
}
```
- js设置html的font-size大小

```js
document.documentElement.style.fontSize = document.documentElement.clientWidth / 750 + 'px';

```
- 使用vw设置，vw也是一个相对单位，100vw等于屏幕宽度
```
html{
    font-size: 10vw;
}
```

这3种方式，都可以设置html的font-size和屏幕宽度成正比。这3种的单位是css尺寸,无论第一种方法的`min-width`还是第二种`document.documentElement.clientWidth`都是相对于设备的css尺寸而言，在iphonex和iphone8得到的结果都是375px。

第一种，需要设置需要每种屏幕都设置对应的font-size,这些font-size都是根据比例算出来的，比较繁琐，而且还有可能漏掉某些屏幕尺寸，不推荐。第二种用js设置，比较方便，现在大部分网站采用这种方式。第三种通过css的vw来设置，也很方便，而且不用写css，但是兼容性还不是特别好。综合推荐使用第二种。

### 单位换算

现在我们要使用rem设置元素尺寸和字体大小。有两种思路：

- 设置特殊的html的font-size,使rem和标注稿上px容易换算，比如把我们的html的font-size设置成1px，这样1rem就等于1px，因为我们标注稿750px，是基于二倍屏的，1个css单位等于2个实际单位，所以我们的font-size设置为0.5px，这样我们设置尺寸时，rem和标注稿的px，就是1比1映射的。当然这里所有的大小都是相对于标注稿尺寸来说的，如果是其他屏幕的尺寸，html的font-size肯定要相应的变大或者变小，通过第二种js方法可以实现：

```js
document.documentElement.style.fontSize = document.documentElement.clientWidth / 750 + 'px';

```
- 通过css预编译或者webpack插件，实时计算
比如，我们将html根元素设置为16px，标注稿上有一个div元素宽度为100px,我们在scss中可以这样写

```scss
$rem: 32px;

div{
	width: 100/$rem;
}
```

webpack的插件也是基于这样的计算原理，比如[px2rem](https://github.com/Jinjiang/px2rem-loader)

```js
module.exports = {
  // ...
  module: {
    rules: [{
      test: /\.css$/,
      use: [{
        loader: 'style-loader'
      }, {
        loader: 'css-loader'
      }, {
        loader: 'px2rem-loader',
        // options here
        options: {
          remUnit: 32,
          remPrecision: 8
        }
      }]
    }]
  }
}
```