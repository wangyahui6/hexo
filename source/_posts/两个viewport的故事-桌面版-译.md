---
title: 两个viewport的故事-桌面版(译)
date: 2017-01-16 15:56:43
categories: web前端
tags: css
---
在这个系列文章中，我将说明viewports和重要元素的宽度是如何工作的，比如`<html>`元素、window和 scrren的宽度。

这篇文章是关于桌面浏览器的，目的是为介绍移动浏览器做好准备。大部分的web开发者已经对桌面浏览器的一些概念很熟悉了。在移动浏览器上我们会发现同样的概念，只不过要更复杂一些，回顾一下这些熟悉的概念将对我们理解移动浏览器有很大的帮助。
<!-- more -->
### 设备像素和css像素
你需要理解的第一个概念是css像素和设备像素之间的区别。

设备像素，顾名思义，无论你用什么设备，设备像素都是表示设备的实际分辨率。设备像素可以从`screen.width/height`读取。

如果你给一个元素`widht:128px`,你的显示器是1024px宽，你最大化你的浏览器，这个元素可以在屏幕上平铺8个。(大概；忽略一些不确定因素)

如果用户缩放了页面，这个值将发生改变。如果用户放大浏览器到200%，你的128px的元素只能在屏幕上平铺4个了。

用户缩放在浏览器中是通过拉伸像素实现的。也就是说，元素的宽度并没有从128px变成256px，而是像素的尺寸变成了原来的两倍。综上，这个元素仍旧有128px的css像素,但是此时它却占有256px的设备像素。

换句话说，放到到200%使一个css像素尺寸了变成了4倍的设备像素的尺寸。(两倍的宽，两倍的高)。

下面几个图片可以很清楚的说明这个概念。第一个是缩放为100%，这个没什么可看的。css像素完全覆盖了设备像素。

![image](http://www.quirksmode.org/mobile/pix/viewport/csspixels_100.gif)

现在我们缩小页面。css像素开始缩小，意味着一个设备像素可以覆盖若干个css像素。

![image](http://www.quirksmode.org/mobile/pix/viewport/csspixels_out.gif)

如果你放大，相反的事情就发生了。css像素开始变大，现在一个css像素可以覆盖若干个设备像素

![image](http://www.quirksmode.org/mobile/pix/viewport/csspixels_in.gif)

关键点在于你只需要关系css像素，它决定你的样式如何渲染。

设备像素对于你来说几乎是无用的。对用户来说不是，用户会缩放页面直到页面看起来舒服位置。但是这个缩放比对你来说不重要，浏览器会自动根据缩放比来缩小或者放大的你的css像素。
### 100% zoom

我上面提到的例子，前提是100%的缩放。现在可以更严格的定义一下：
```
在100%缩放的情况下，css像素和设备像素是严格相等的。
```
这100%缩放的概念在我们的这个解释中是非常有用的，但是我们在日常开发中不需要过度担心这个。在桌面浏览器开发中通常你都是在100%缩放的情况下，即使用户缩放页面，css像素的原理也能保证你的布局保持比例，不能打乱。
### Screen Size

让我们来看一下实际的尺寸吧。我们从`screen.width`和`screen.height`开始。他们表示用户屏幕的总宽度和总高度。他们的单位是设备像素，因为它们从来不会改变：它们是显示器的特性，不是浏览器的特性。

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_screen.jpg)

很有意思！但是我们能用这个信息做什么那？

基本没有用。用户的显示器尺寸对我们来说不重要，除非你想做一个web资料数据库。

### Window Size

相反，你关心的是浏览器窗口的内部尺寸是什么。那会告诉你用于展示你的css布局的空间是多大。你可以通过`window.innerWidth`和`window.innerHeight`来获取。

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_inner.jpg)

很明显，窗口的内部宽度的单位是css像素。你需要知道的是你的css布局有多少可以呈现在浏览器窗口中，而且这个呈现的数量会随着用户放大页面而减少(css像素越大，所呈现的内容越少)。因此如果用户放大页面，你可用的空间也就越少，在`window.innerWidth/Height`反应为值减小。

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_inner_zoomed.jpg)

注意！这个宽度和高度包括滚动条。滚动条也被认为是浏览器窗口的一部分。(这个有历史的原因)
### Scrolling offset

`window.pageXOffset`和`window.pageYOffset`,表示document横向和纵向的滚动偏移。通过这两个属性，你可以知道用户滚动的位置。

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_page.jpg)

这两个属性的单位也是css像素。理论上，如果用户向上滚动然后放大，`window.pageX/YOffset`应该发生改变。但是浏览器做了一些处理，在用户缩放的时候，浏览器试图使同一个元素保持在浏览器窗口的顶部来使页面看起来不会跳动。这意味着`window.pageX/YOffset`在用户缩放的时候不会改变：被滚动出屏幕的css像素数不会改变。

![](http://www.quirksmode.org/mobile/pix/viewport/desktop_page_zoomed.jpg)

### 概念:the viewport

在我们继续介绍更多的js属性之前，我们必须介绍另外一个概念：viewport。

viewport的作用是限制`<html>`元素，是网站的最顶级的块级元素。

这听起来或许有一点模糊，我们来举一个实际的例子。假设你有一个流体布局，你的侧边栏是`width:10%`.当你调整浏览器的大小时，侧边栏随着增大或减小。那它到底是怎么工作的那？

从技术上说，侧边栏在获取它父元素的宽度的10%的时候发生了什么。我们假定`<body>`元素为父元素。所以现在问题变成了`<body>`元素（你没有给`<body>`赋宽度）的宽度是多少。

通常来说，所有的块级元素的宽度都是父元素的宽度的100%。因此`<body>`元素是和它的父元素`<html>`一样宽的。

现在`<html>`元素的宽度是多少那？为什么它和浏览器一样宽。这也是为什么你的`width:10%`的侧边栏占整个浏览器宽度的10%的原因。所有的web开发者都知道这个事实。

你或许不知道这其中的工作原理。理论上，html元素的宽度是被viewport的宽度限制的。html元素等于viewport的宽度。

viewport和浏览器窗口相等的：它就是这么被定义的。viewport不是一个HTML结构，所以你不能靠css影响它。它就是有浏览器窗口的宽度和高度(桌面);在移动浏览器上它是比较复杂的。

### 结果

这些东西有时候会有一些奇怪的结果。你能这个网站(http://www.quirksmode.org/mobile/viewports.html)中看到其中一个。滚动到最顶部，然后放大页面2到3倍，使页面内容溢出浏览器窗口。

现在滚动到最右边，你将会看到顶部的蓝色栏不再被正确的排列。

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_htmlbehaviour.jpg)

这个行为是viewport被定义的方式导致的。我给了蓝色顶部栏一个`width:100%`.什么的100%？html元素的100%。`<html>`元素和viewport是等宽的，所以和浏览器窗口也是等宽的。

重点是：在100%缩放时，它是正常的，现在我们放大页面，导致viewport变的比我们页面的总宽度小。对于它自己来说这不重要，页面的内容溢出了html元素，但是html元素是`[overflow](http://www.quirksmode.org/css/overflow.html): visible`，这就意味着超出的元素会被显示。

但是蓝色顶部栏没有溢出。我给了它width:100%,浏览器会给它一个viewport的宽度。他们不关心现在的宽度太小了。

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_100percent.jpg)

### document width?

我真正想知道的是页面内容的总宽度是多少，包括突出的部分。据我所知，浏览器并未提供这个值。

我开始相信我们需要一个js属性对来表示我称作"document width"的值。

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_documentwidth.jpg)

如果我们真是觉得这样不爽，为什么不把document width的值暴露给css那？我希望蓝色顶部栏继承document宽度，而不是html元素的宽度。（这个确实有些棘手，如果不可能实现我也不会觉得惊讶）

浏览器厂商，你们怎么认为那？

### viewport尺寸

你或许想要知道viewport的尺寸。他们可以通过`document.documentElement.clientWidth/clientHeight`来获取.

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_client.jpg)

如果你了解dom结构，你就知道`document.documentElement`其实是`<html>`元素：`<html>`文档的根元素。然而，viewport是更高一级的，可以说它是包含`<html>`元素的元素。如果你给了`<html>`元素一个宽度，那就变得比较重要了(不推荐这样做，但是这是可以的)。即使在这种情况下，`document.documentElement.clientWidth/clientHeight`仍旧给出的是viewport的尺寸，而不是html的尺寸。(这是一个只对这个元素和这个属性对起作用的特例。其他情况下`clientWidth/clientHeight`都是取元素的真实尺寸)。

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_client_smallpage.jpg)

因此`document.documentElement.clientWidth/clientHeight`总是给出viewport的尺寸，无视html的尺寸。

### 两对属性值

那么viewport的尺寸是不是也可以由`window.innerWidth/Height`给出。答案是也不是。

这两对属性值的唯一区别在于，`window.innerWidth/Height`包括滚动条的宽度，`document.documentElement.clientWidth/clientHeight`不包括。

我们之所以有两对属性是浏览器大战的产物。当时Netscape只支持`window.innerWidth/Height`，而IE只支持`document.documentElement.clientWidth/clientHeight`。当所有其他浏览器开始支持`document.documentElement.clientWidth/clientHeight`的时候，IE仍旧不支持`window.innerWidth/Height`.在桌面浏览器上有两个属性对是一个烦人的事情，但是在移动浏览器上它是一个福音。

### html元素的尺寸

`document.documentElement.clientWidth/clientHeight`在所有的情况下都给出的是viewport的尺寸。那么我们从哪里获取html元素自身的宽和高那？他们被存在`document.documentElement.offsetWidth/offsetHeight`里。

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_offset.jpg)

这两个属性真地给了你一个访问`<html>`元素作为块级元素的接口。如果你设置`width`或者`offsetWidth`将会影响这两个属性。

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_offset_smallpage.jpg)

### 事件坐标

有一些事件坐标值。当一个鼠标事件发生时，不少于5对属性值会被暴露出来给你事件发生的具体位置信息。其中3对是对我们的讨论来说重要的：
```
1. pageX/Y给出相对于html元素的坐标，单位是css像素
2. clientX/Y给出相对于viewport的坐标，单位是css像素
3. screenX/Y给出相对于屏幕的坐标，单位是设备像素
```

pageX/Y

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_pageXY.jpg)

clientX/Y

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_clientXY.jpg)

screenX/Y

![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_screenXY.jpg)

你90%的情况下都会使用`pageX/Y`;通常你想要知道相对于document的位置。其他的10%你想要用`clientX/Y`.你基本不需要知道相对于屏幕的尺寸。

### 媒体查询

最后，说一些媒体查询。这个思想很简单：你可以指定在页面在不同条件下运行不同的css，比如页面宽度大于、等于、小于某个尺寸的时候。
```
div.sidebar{
    width:30%;
}
@media all and (max-width:400px) {
    //styles assigned when width is smaller than 400px;
    div.sidebar {
        width: 100px;
    }
}
```
现在这个sidebar在宽度大于400px的时候宽300px,小于等于400px的时候宽100px;

问题是哪个宽度和400px比较？

有两个相关的媒体查询：`width/height` 和 `device-width/device-height`.

```
1. `width/height`用的是`documentElement.clientWidth/height`（就是viewport）。单位是css像素。
2. `device-width/device-height`用的是`screen.width/height`.单位是设备像素。
```
![image](http://www.quirksmode.org/mobile/pix/viewport/desktop_mediaqueries.jpg)

应该用哪个宽度？想都不用想，当然是`width`。web开发者对设备宽度不感兴趣，只是对浏览器窗口的宽度感兴趣。

在桌面浏览器上使用`width`，忘记`device-width`.正如我们所看到的，这在移动设备上是更复杂的。

### 总结

这篇文章总结了我们对桌面浏览器的探索。第二篇文章将介绍这些概念在移动浏览器的应用，并重点说明和桌面浏览器的不同。

原文出处：http://www.quirksmode.org/mobile/viewports.html


