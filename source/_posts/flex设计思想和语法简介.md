---
title: flex设计思想和语法简介
categories: 前端 css flex
tags: css flex 弹性布局
description: flex弹性盒子模型是新一代的布局模式，在大部分情况下可以完全替代之前的display属性 + position属性 +
  float属性的布局方式，设计友好，目前兼容性已经很好了，很多主流网站都已经开始采用。本文总结一下flex的设计思想和语法，以及在实际的布局应用中为我们带来了哪些便利。
date: 2018-10-22 10:13:19
---

### flex设计思想

我们可以仔细想一下我们之前布局的难点和麻烦的地方都有什么，我总结了几个：

- 自适应，比如最简单的左侧是固定宽度导航栏，右侧自适应填充剩余空间。一般使用float和margin来实现或者利用BFC的特性来实现
- 居中，特别是垂直居中，一行有多个高度不一致的项目，要垂直居中，一般结合line-height和vertical-align来实现
- 水平等间距的多项目排列，最两端的两侧没有间距，通常使用负margin或者css选择器来实现
- 需要解决float造成的高度塌陷问题

其实`float`、`position`、`line-height`、`vertical-align`设计之初并不是为了应对这种复杂布局，只是比较适用于web1.0的图文布局界面。后来人以自己的聪明才智利用种种css特性，组合使用，实现了复杂的布局方案。这种布局方案相对比较复杂，需要很扎实的基本功，不是太容易理解。flex弹性布局就是为了应对这种情况而生的。

对于上述种种不便，如果让我们自己去要求有一种新的布局方案，我们会怎么要求，我希望的是这样

- 我希望可以定义某个元素自动扩大或者自动缩小，以实现自适应
- 我希望可以定义行或者列的排列方式，从左边开始、居中、等间隔、从右边开始等，都可以任意设置
- 我希望设置垂直方向和设置水平方向一样简单

我的要求就这么简单满足着3条就可以解决我日常开发中的大部分问题，提高开发效率。flex的布局设计就可以轻松满足上述要求，而且还可以满足我们更多的要求。

我们来了解一下flex的设计，首先我们了解一下2个概念：

- flex容器：凡是设置了`display: flex`的元素都是`flex容器`
- flex项目：所有`flex容器`的子元素都被称为`flex项目`

#### flex 容器
`flex容器`默认有两个轴线分别对应水平方向和垂直方向，叫做主轴和交叉轴。

![container](/images/flex/container.png)

默认主轴是水平方向，交叉轴是垂直方向，但是我们也可以通过设置调换，所以在flex设计中水平方向和垂直方向是一样容易设置的。我们可以在`flex容器`中设置主轴方向、排列方式。容器一共有6个css属性可以设置，其实都是围绕主轴方向、排列方式设置的。

- `flex-direction`: 决定主轴方向，一共有4个值
    - row(默认)： 主轴为水平方向，从左到右排列
    - row-reverse: 主轴为水平方向，从右到左排列
    - column：主轴为垂直方向，从上到下排列
    - column-reverse: 主轴为垂直方向，从下到上排列

![container](/images/flex/flex-direction.png)

- `flex-wrap`: 决定换行：
    - nowrap(默认)：不换行
    - wrap: 超过主轴空间则换行
    - wrap-reverse: 超过主轴空间则换行,新行在前面

- `flex-flow`: `flex-direction`属性和`flex-wrap`属性的简写形式，默认值为`row nowrap`

- `justify-content`: 这个属性就是决定主轴方向排列方式的，水平居中，等间隔排列都可以用它来实现，应用的比较多，有6个值：
    - flex-start(默认)：左对齐
    - flex-end: 右对齐
    - center: 居中
    - space-between: 两端对齐，项目间的间隔相等
    - space-around: 每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。
    - space-evenly: 项目沿主轴均匀分布。

![container](/images/flex/justify-content.svg)

- `align-items`: 这个属性决定交叉轴方向的排列方式，可以很方便地实现垂直居中，也有5个值，以从上到下的垂直方向为交叉轴方向：
    - flex-start: 上对齐
    - flex-end: 下对齐
    - center: 居中对齐
    - stretch（默认值）: 拉伸，如果未设置高度或者高度设为为`auto`,则填充整个容器高度
    - baseline: 项目的第一行文字的基线对齐

![container](/images/flex/align-items.svg)

- `align-content`: 该属性定义了主轴有多根轴线时，多根轴线在交叉轴的排列方式。只有一根轴线，就是项目没有换行时，该属性不起作用。这里比较复杂的一个地方是多轴线的问题，主轴一旦换行就会形成多个轴线，`align-items`仍然在每个轴线内起作用，而轴线的排列方式则由`align-content`来设置，和`justify-content`相对应。该属性有6个值，以从上到下的垂直方向为交叉轴方向：
    - flex-start：上对齐
    - flex-end: 下对齐
    - center: 居中
    - space-between: 两端对齐，项目间的间隔相等
    - space-around: 每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍
    - stretch(默认值): 轴线占满整个交叉轴,strech属性和其他属性的不同之处在于，strech会自动扩展轴线的高度，而其他属性都是以轴线实际占用的位置进行排列。具体可以看下下面的例子。

![container](/images/flex/align-content.svg)

例子：
```html
<div class="outer">
  <div class="inner">hello</div>
  <div class="inner">nihao</div>
  <div class="inner">wobuhao</div>
</div>
```
```css
.outer{
  display: flex;
  background: blue;
  height: 400px;
  flex-wrap: wrap;
  align-items: stretch;
  align-content: stretch;
}
.inner{
  width: 650px;
  background: orange;
}
```
效果如下：

![container](/images/flex/align-content2.png)

如果我们把`align-content`设置为`space-between`,则效果如下：

![container](/images/flex/align-content3.png)

如果我们把`align-content`设置为`stretch`,同时`align-items`设置为`center`,则效果如下,说明`align-items`在单个轴线内起作用：

![container](/images/flex/align-content1.png)

你可以自己去[试一试](https://codepen.io/huiweiwudi/pen/KGByXb)!

容器上的属性都已经介绍完了，主要就是设置主轴方向和两个轴的排列方式。

#### flex项目

下面我们介绍一下flex项目上的属性，`flex容器`上主要设置了排列方式，`flex项目`上则主要设置了项目的大小。同样`flex-项目`上有6个css属性可以设置：

- `order`: 定义项目的排列顺序。数值越小，排列越靠前，默认为0

- `flex-grow`: 定义项目的放大比例，该值越大，放大的比例越大，0表示不放大,默认值为0。这个属性和下一个`flex-shrink`很重要，需要深入理解。这个有一个很容易误解的点，就是放大比例是参考谁来放大的。注意不是参考主轴宽度，而是参考剩余宽度，也就是容器宽度减去项目总宽度。当然了，如果项目总宽度加起来比容器宽度大，那就不存在放大这一说了，有剩余空间的情况下才会去放大。假如一行有n项目，n个项目的放大比例分别为grow<sub>1</sub>,grow<sub>2</sub>...grow<sub>n</sub>,则第m(m<=n)个项目的放大空间为：

    space = grow<sub>m</sub>/(grow<sub>1</sub>+grow<sub>2</sub>+...grow<sub>n</sub>) *  (`容器宽度`-`项目总宽度`)

    由此也可以知道为什么0表示不放大,知道了明确的换算公式，我们在设置大小时才不会乱。

- `flex-shrink`: 定义项目的缩小比例，该值越大，缩小的比例越大，0表示不缩小，默认为1。缩小比例的参考系则是项目总宽度减去容器宽度，也就是说容器宽度容纳不下项目了，项目才会去缩小。这里和`flex-grow`不一样的地方是，缩小空间不止和`flex-shrink`的值有关，还和项目的宽度有关。假如一行有n个项目，n个项目的`flex-shrink`值分别为shrink<sub>1</sub>,shrink<sub>2</sub>...shrink<sub>n</sub>,n个项目的宽度分别为width<sub>1</sub>,width<sub>2</sub>...width<sub>n</sub>,则第m(m<=n)个项目的缩小空间为：

    space = (shrink<sub>m</sub> `*` width<sub>m</sub>)/(shrink<sub>1</sub> `*` width<sub>1</sub> + shrink<sub>2</sub> `*` width<sub>2</sub> +...shrink<sub>n</sub> `*` width<sub>n</sub>) `*` (`项目总宽度`-`容器宽度`)

- `flex-basis`: 定义了项目在主轴上的所占据的空间大小，默认值为`auto`。我们之前说的放大缩小，都要依赖于`项目总宽度`的计算。这个`项目总宽度`就是依据`flex-basis`算出来的。那如果项目本来就设置了`width`或者`height`,那项目宽度或者高度由谁来决定那？答案是`flex-basis`,`flex-basis`的优先级高于`width`或者`height`,但是低于`min-widht`或者`max-height`,仍受最大最小宽度的限制。比如下面的例子：
```html
<div class="outer">
  <div class="inner">
    
  </div>
</div>
```
```css
.outer{
  display: flex;
  height: 300px;
  background: blue;
}
.inner{
  width: 300px;
  flex-basis: 500px;
  background: yellow;
}
```
效果如下：

![remote](/images/flex/shrink.png)

可以看出项目的宽度是由`flex-basis`决定的。另外当`flex-basis`为`auto`时，则表示项目占据主轴空间的大小为项目本来的大小，此时`width`便会起作用。

- `flex`: flex属性是`flex-grow`,`flex-shrink`和`flex-basis`的简写，默认值为`0 1 auto`。`flex`的值使用起来比较复杂，可以看下面这段代码，基本包含所有情况。
```css
/* Basic values */
flex: auto; /* 1 1 auto，该放大放大，该缩小缩小 */ 
flex: none; /* 0 0 auto，既不缩小，也不放大 */ 

/* One value, unitless number: flex-grow */
flex: 2;

/* One value, width/height: flex-basis */
flex: 10em;
flex: 30px;

/* Two values: flex-grow | flex-basis */
flex: 1 30px;

/* Two values: flex-grow | flex-shrink */
flex: 2 2;

/* Three values: flex-grow | flex-shrink | flex-basis */
flex: 2 2 10%;
```

- `align-self`: 和`align-items`作用一样，只不过该属性设置在项目上，可以对单个项目设置，允许我们设置某些项目的排列方式和其他不同。`align-self`多了一个`auto`属性，表示继承父元素的`align-items`属性。