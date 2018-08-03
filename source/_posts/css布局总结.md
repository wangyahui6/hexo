---
title: css布局基础总结
date: 2018-07-16 15:55:58
description: 前端css布局知识繁杂，实现方式多种多样。想写出高效、合理的布局，必须以深厚的css基础为前提。为了方便记忆和复习，将css布局要点记录如下。内容较多，应用方面说的不太详细，但都是很实用的点。
categories: css
tags: css 布局
---

所谓布局，就是指将元素设置为我们想要的大小，放置于我们想要的位置,`位置`、`尺寸`是核心两要素。这些元素其实就是一些方块，页面就是由各种方块拼凑而成。现在布局方式主要分为三种：

- 传统css布局方案（position,float,line-height等配合）。实现复杂，需要多种属性配合使用，兼容性最好。
- flex布局方案。弹性布局，实现方便，兼容性较好。
- gird布局方案。

使用哪种布局方式，看项目具体要求，如果不需要兼容IE，建议使用flex或者grid，这两种是未来趋势。如果要考虑兼容，则最好使用传统css布局方案。

## 传统css布局方案

### css标准盒模型

前面提过，一个html元素，就是一个方块，讲究一点的话就是盒模型。盒模型长这样：
![盒模型](/images/css_layout/css_box.png)

我们可以在chrome开发者工具的styles中查看：
![盒模型](/images/css_layout/chrome_css_box.png)

css盒模型，分为4部分：

- content：显示元素内容区域，包含子孙元素的地方
- padding：内容区域到边框的距离，也称内边距
- border：显示自身轮廓
- margin: 用于设置元素自身和同级元素或者父级元素的距离

### 规则

无规矩不成方圆。所有的html元素都是按照一定规则去排列渲染的。在html中，不只有一种规则，是多种规则的混合。了解这些渲染规则和这些规则生效的条件对于我们理解css有很大帮助。
#### 文档流
文档流可以理解为所有元素的默认渲染规则。它的规则很简单：元素按照自己的`类型`的布局特性从左到右，从上往下依次排列。

元素在布局上分为三种类型：
- 块级元素：默认占据一整行，即时设置宽度也还是占据一整行，由margin来补充（霸道总裁），有block、table、flex、grid、list-item
- 内联元素：宽度由内容撑开，只占据自己的位置，即时设置了宽度也不起作用,margin也是左右起作用，上下不起作用（大家闺秀）,有inline
- 内联块级元素： 宽度由内容撑开，只占据自己的位置，可以设置自己的宽度（两个人的孩子？）,有inline-block、inline-table、inline-flex、inline-grid

上面内联元素其实说的不太正确，这里需要特殊说一下，内联元素又分为两种：

- 可置换内联元素：该元素展示的内容不在css作用域范围之内，典型比如img、video、object、input、textarea等，展示内容由src、value等决定，这种元素可以设置宽高
- 不可置换内联元素：普通的内联元素，比如span、a、i、em等

有两种方式可以脱离文档流，绝对定位和浮动，我们下面再详细说。

#### BFC

BFC全称是Block Formatting Context,块级格式化上下文。在BFC环境中的元素按照如下规则渲染：

- 和文档流一样，元素按照自己`类型`的布局特性从左到右，从上往下依次排列。
- BFC是一个独立、封闭的渲染区域。子元素的样式对BFC外部产生影响。
- BFC可以识别浮动子元素。
- BFC可以识别浮动的同级元素。利用这一点可以制作弹性布局。

那么什么样的渲染区域是一个BFC哪？下列几种方式可以显示声明一个BFC的渲染区域：
- 根元素或包含根元素的元素
- float的值不为none
- position为absolute或者fixed
- overflow的值不为visible
- display为inline-block、table-cell、table-caption

### 尺寸

元素的尺寸有两种情况：

- 默认情况： 块级元素宽度默认为100%，高度由内容撑开；内联元素和内联块级元素宽度和高度默认由内容撑开。
- 开发者设置： 主动设置width、height、line-height等

我们先来明确一下元素的尺寸概念，我们都知道元素的盒模型，元素由content、padding、border、margin四个区域组成。那么尺寸具体是指什么？我们可以把尺寸分为三类：

- 元素尺寸：由content、padding、border组成，在原生的DOM API中用offsetWidth、offsetHeight获取
- 元素内部尺寸： 由content、padding组成，在原生的DOM API中用clientWidth和clientHeight获取
- 元素外部尺寸： 由content、padding、border、margin组成，没有对应的DOM API,但是理解这个，对布局很有帮助

而我们在css中设置的width、height代表哪部分区域是有歧义的，我们通过设置box-sizing，可以切换width、height所代表的区域。

#### box-sizing

box-sizing，可以切换width、height所代表的区域。box-sizing主流浏览器有两个值：

- content-box: width、height设置的是content的宽高
- border-box: width、height设置的是content、padding、border加起来的宽高

box-sizing的这两个属性，造成width、height的二义性，而且很多情况下是全局设置的，隐蔽性很强，对于新手来说很容易懵。这两个属性的产生有一定的历史原因，最开始的盒模型默认采用的border-box，早期的IE就是这种，也叫IE盒模型。后来W3C觉得content-box更好，又把盒模型默认改为content-box模式。后来的后来，随着弹性布局的流行，border-box的优势越来越明显，大家都更愿意使用border-box来布局，W3C又想把border-box搞回来。但是已经有好多基于content-box的网站，为了兼容性也不能随便改。于是，W3C就想出了box-sizing这种方式，支持了border-box，但是默认还是content-box。使用box-sizing应注意的几点：

- 加入要全局设置box-sizing: border-box，要注意对第三方组件库的侵入，因为第三方组件很可能是基于content-box来布局的
- 设置所有元素的box-sizing属性为inherit：
    ```css
    *, *:before, *:after {
        box-sizing: inherit;
    }
    ```
    这样我们可以设置父元素的box-sizing，就可以控制所有子元素的box-sizing属性，对于我们封装组件，设置整个组件环境很方便，只有我们在封装组件时强设置box-sizing，就不怕全局的样式侵入了。
- content-box模式在设置弹性布局时，可以使用calc属性辅助实现，或者弄两层div实现

#### 尺寸的百分比设置

##### 包含块
我们知道width, height都是可以设置百分比，那这个百分比的参照物是谁？这里引出一个概念，叫做包含块(CB, Contanining Block),一个元素的包含块就是该元素的width、height百分比的参照物。

很多新手同学，在设置宽高百分比的时候，有时候觉得莫名其妙，各种奇怪现象，怎么设置都不起作用。其实就是没有包含块的概念。一个元素的包含块是谁，主要取决于该元素的position属性，总结如下：

- position为static和relative的元素，包含块为其父元素的content-box
- position为absolute的元素，包含块为其最近的定位非static的祖先元素的padding-box，如果没有定位非static的祖先元素，则为初始包含块（后面解释）
- position为fixed的元素，包含块为视口viewport
- position为absolute和fixed时，包含块也可能是由满足以下条件的最近父级元素的padding-box：
    1. A transform or perspective value other than none
    2. A will-change value of transform or perspective
    3. A filter value other than none or a will-change value of filter (only works on Firefox)
   这条比较特殊，遇到的情况比较少，单独拿出来 

除了width, height的百分比相对于包含块设置外，margin、padding的百分比也是相对于包含块设置，只不过margin、padding百分比相对于包括块的宽度设置（水平模式下）。绝对定位的偏移属性top、left、right、bottom也是相对于包含块设置，后面再详细说。
##### 初始包含块

元素的包含块都是自己的祖先元素，那`hmtl`元素没有祖先元素，它的百分比设置相对于谁那？就是初始包含块，根元素(`hmtl`)所在的包含块是一个被称为初始包含块的矩形。这个矩形的大小就是浏览器视口的大小。有一点需要注意，那些没有定位非static祖先元素的参照物是初始包含块，而非html元素。

#### margin

因为margin在布局中有一些重要特性和特殊情况，所以单独拿出来讲一下。

##### margin:auto

我们知道，块级元素即使设置了宽度，也会占满一行，为什么会这样？剩余的空间被谁占了？

这里要明确一点，块级元素占据一行，是指块级元素的外部尺寸占据一行，就是margin-box。当margin设置为auto的时候，margin会自动占满剩余空间。

- margin-left: 默认为0，为auto时，自动充满剩余空间
- margin-right: 默认为0，为auto时，自动充满剩余空间
- margin-top: 默认为0，为auto时，值还是为0
- margin-bottom: 默认为0，为auto时，值还是为0

当margin-left和margin-right同时为auto，就会平分剩余空间，这就是`margin:auto`会使元素水平居中的原因。然后`margin: auto`却不能使元素垂直居中，这是因为在垂直方向上，块级元素不会自动扩充，它的外部尺寸没有自动充满父元素，也没有剩余空间可说。如果我们在父元素上设置`writing-mode: vertical-lr`,这时`margin:auto`就会使子元素垂直居中，而水平居中无效。

那有没有什么办法使用`margin:auto`让元素同时水平垂直居中？答案是有的，就是绝对定位的情况下。
```css
.father {
    width: 300px; height:150px;
    position: relative;
}
.son {
    position: absolute;
    top: 0; right: 0; bottom: 0; left: 0;
    width: 200px;
    height: 200px;
    margin: auto;
}
```
此时的son的外部尺寸，就会自动充满它的包含块，效果和块级元素类似。这时候设置.son元素的宽高，就会有剩余空间出来，`margin:auto`会自动平分剩余空间，使.son水平垂直居中。

##### margin对布局影响

margin对元素的影响有2个：
- 影响内部尺寸
- 影响外部尺寸

###### 影响元素内部尺寸

块级元素这种宽度方向自动充满空间的布局特性，当没有设置宽度，设置margin-left和margin-right为对元素的内部空间有影响。margin为正值时，`元素尺寸`缩小；margin为负值时，`元素尺寸`增大。因为元素的外部尺寸大小已经定了，就是其包含块的尺寸，而`外部尺寸`等于`元素尺寸`加上margin，如果margin为负，`元素尺寸`自然就增大了。利用内部尺寸增大、外部尺寸不变这个特点，我们可以进行等间隔列表的布局。比如我们实现一个一行三列，两侧无间隙，中间间隙为20px的布局：
```css
.father{
    margin-right: -20px;
}
.son{
    width: calc( calc(100% - 20px * 3) / 3);
    margin-right: 20px;
}
```

###### 影响元素外部尺寸

在元素宽高固定的时候，相当于`元素尺寸`固定了，margin开始影响元素的`外部尺寸`。正margin会使元素元素的`外部尺寸`增大，负margin会使元素元素的`外部尺寸`缩小。正值很好理解，就不说了。主要说一下负值，这时候margin-top、margin-left和margin-right、margin-bottom的表现看起来是有区别的，实质是一样。margin-top、margin-left为负值时，表现为元素向上或者向左移动，margin-right、margin-bottom为负值时，表现为其右侧元素或者下面元素都会相应地向左或者向上移动。仔细想想，这都是元素`外部尺寸`缩小的表现。margin-top、margin-left使左侧上上侧尺寸缩小，就会自然向左或者向上移动。margin-right、margin-bottom使右侧和下侧尺寸缩小，旁边和下面的元素自然就会往左或者往上占位。

##### margin合并

块级元素的上边距和下边距在某些场景下会发生合并行为。需要注意的是浮动元素和绝对定位元素不会发生合并，且合并只发生在垂直方向。有以下3中合并场景

- 相邻兄弟元素
- 父元素和第一个/最后一个元素
- 元素内容为空时，height为0，自己的上边距和下边距会发生合并

margin合并的计算规则

- 正正取大
- 正负相加
- 负父取小(-20px, -50px合并取-50px)

阻止margin合并的方法

- 把合并元素变成BFC，比如设置display: inline-block,overflow: hidden, float: left等
- 父子合并/空元素合并时，给父元素/空元素设置border、padding都能阻止合并

#### 内联元素尺寸
主要想说`line-height`,`vertical-align`两个属性，这两个属性对高度有重要的影响，而且如果没有理解这两个属性的作用，会经常碰到一些不好解决的诡异的布局问题。

##### 幽灵空白节点

这个名字是张鑫旭大神定义的，在W3C规范中也是可以找到根据的。在《CSS世界》这本书中，为了证明幽灵空白节点的存在，举了一个例子：

```css
div {
    background-color: #cd0000;
}
span {
    display: inline-block;
}
<div><span></span></div>
```
你可能猜到了，div的高度不为0，我在chrome下试了试高度为21px，这个高度猜测应该和div的字体、字体大小、行高有关。这个幽灵空白节点的存在，会引起一些怪异现象，比如：

```css
<div><img></img></div>
```
div的高度总是比img高一些，img下总有一个间距。这也是一个幽灵空白节点在起作用。
##### line-height

line-height翻译过来就是行高。它指的是指`行框`的高。那什么是`行框`那？

行框是由内联元素或者内联块级元素组成的一行。通过尝试，发现一行形成行框的两个前提是，要么有文字内容，要么有内联块级元素，如果其中是一堆空的内联元素，比如span，是没有形成行框的。

line-height就是指定行框占据的高度。只有形成了行框，line-height才会起作用。

这里可能有一些容易误解的情况，比如：
```css
.father{
    line-height: 300px;
}
<div class="father">
    <div class="son">
        你好
    </div>
</div>
```
这时候.fahter高度是300px，这里注意的是，不是line-height对.son起作用了，是.son内部形成了一个行框，line-height对这个行框起作用了，.son的高度是300px，撑开了.father。

##### vertical-align

vertical-align用来设置内联元素或者内联块级元素在行框内的对齐方式或者说垂直位置。verticle-align有四类值，我们在这里直说其中对布局比较重要的两类：

- 线类： baseline（默认值）、top、middle、bottom
- 数值类：比如20px、20em

##### 基线

先来看一张图：

![基线](/images/css_layout/baseline.png)

vertical-align默认是基线对齐的，基线的位置很重要。而基线的定义是：`字母x的下边缘`。中线的位置是基线往上 1/2 x-height 高度，就是x交叉的地方。这里有一个需要注意的地方是，纯文字的中线是x交叉的地方，但是内联元素的中线却是内联元素的正中间位置。我们来看一个例子：
```css
div{
  line-height: 100px;
  background: yellow;
}
<div>
  <span class="world">xhello</span>
</div>
```
效果如下：
![正常高度](/images/css_layout/normal_height.png)

一切正常，div高度100px，文字也是近似垂直居中。但是如果我们在.world元素上加`vertical-align: middle`，我们会发现文字向下偏移，且div高度变成了102px，效果如下：
![不正常高度](/images/css_layout/abnormal_height.png)

这是怎么回事那，vertical-align设置的到底跟谁对齐？答案是幽灵空白节点，幽灵空白节点是个很神奇的存在，看不见，但却实实在在地在起作用，它的作用就相当于一个x字符的作用，用来提供对齐基准。我们上面说到的，div元素中有一个图片的例子，img是可替换内联元素，它的baseline就是它的底部，默认是baseline对齐，所以图片对齐的是幽灵空白节点x的基线，x基线位置往下是一段空白的，这就是这段空白的来源。为了更直观，我们在上面这个例子的span元素前面加一个x字符，当做幽灵空白节点，看一下效果：
![不正常高度](/images/css_layout/base_height.png)

本来，`x`和`xhello`中的`x`是对齐的。设置vertical-align之后，我们可以看到xhello字符向下挪了一段位置，这是因为span元素的垂直中线位置，比`xhello`中`x`交叉点要高，现在要和前面的`x`交叉点对齐，就往下移动了。又因为`x`和`xhello`的行高都是100px，但是现在因为由于错位，造成整体高度多了2px。

##### 垂直居中应用

利用line-height和vertical-align可以设置多种场景下的垂直居中。

- 单行居中

```css
div{
    line-height: 任意值
}
<div>hello world</div>
```
- 父元素定高垂直居中，这个在.box上设置line-height实际上设置了幽灵空白节点的行高是120px，然后.content和幽灵空白节点middle对齐就实现了垂直居中。

```css
.box {
    line-height: 120px;
    background-color: #f0f3f9;
}
.content {
    display: inline-block;
    line-height: 20px;
    margin: 0 20px;
    vertical-align: middle;
}
<div class="box">
    <div class="content">多行文字...</div>
</div>
```
- 父元素不定高垂直居中,原理其实和父元素定高情况相同，但是因为不定高，你没法设置父元素的line-height,这时候我们可以设置一个父元素的伪元素，设置成高度100%，vertical:middle,因为元素是从左上向下排列，所以vertical:middle会把幽灵空白节点的x拉到父元素垂直中心位置。这时候，我们再设置子元素的vertical-align:middle就达到了目的。

```css
.container {
    position: fixed;
    top: 0; right: 0; bottom: 0; left: 0;
    background-color: rgba(0,0,0,.5);
}
.container:after {
    content: '';
    display: inline-block;
    height: 100%;
    vertical-align: middle;
}
.dialog {
    display: inline-block;
    vertical-align: middle;
}
<div class="container">
    <div class="dialog"></dialog>
</div>
```
### 位置

元素的位置由position属性和float属性决定。

#### position属性

position有4个值：

- static: 默认值，按照css规则排列
- relative：相对定位，可设置偏移量，top,left,right,bottom,偏移量百分比大小的参照物是其包含块，偏移量的位置参照物是自身位置
- absolute：绝对定位，可设置偏移量，top,left,right,bottom,偏移量百分比大小和位置的参照物是其包含块
- fixed：固定定位，可设置偏移量，top,left,right,bottom,偏移量百分比大小和位置的参照物是其包含块viewport

需要注意的地方有：

- position为relative、absolute、fixed时，如果不设置偏移量，默认位置为文档流的正常位置
- position为absolute、fiexed的元素，脱离文档流，文档流中的元素不识别该元素，不识别它的占位和宽高度（不是自家孩子）
- position为absolute、fiexed的元素，宽度的渲染规则和内联块级元素相同，宽度默认为由内容撑开，可设置宽高


#### float属性

float属性可以使元素脱离文档流，向左或者向右移动，直至碰到包含块的padding或者碰到另一个浮动元素。注意：

- position为absolute、fixed会让浮动属性失效
- float会使元素块状化，dispaly的值，inline-table变为table，剩下的都变为blcok，但是元素宽度的渲染规则和内联块级元素相同，宽度默认为由内容撑开，可设置宽高

float最著名的就是它的高度坍塌，之所以有高度坍塌，和float设计的初衷有关。float属性最初是设计用来实现文字环绕效果的。
```
<div class="img-wrap">
    <img style="float: left"/>
</div>
<p>我是文字，我要环绕你</p>
```
.img-wrap因为高度塌陷，实际上不识别float元素，高度为0，p元素与其重合，从而实现文字环绕效果。

高度塌陷特性在实现布局的时候是不合理的，而现在文字环绕的场景很少，float基本都用来布局使用了。这现在实际上已经成为了一个“bug”的存在。所以css又提供了一个属性用来清除浮动：clearfix。

##### clearfix

clearfix在mdn上的解释为指定一个元素是否可以在它之前的浮动元素旁边（我觉得这里用重合更贴切），或者必须向下移动(清除浮动) 在它的下面。

注意上面这句话，`指定一个元素`表示clearfix的作用目标是元素自身，`在它之前的浮动元素`表示clearfix的作用条件是它前面有浮动元素，而clearfix的作用效果是让元素换行显示。所以clearfix的作用是，如果元素前面有浮动元素，可以指定该元素换行显示，或者说是识别前面的浮动元素。clearfix有3个值：

- left: 清除左浮动
- right: 清除右浮动
- both: 清除左右浮动

一般情况下采用clearfix: both。清除浮动一个重要应用就是解决float高度塌陷的问题。通用的解决方案是设置.clearfix类：
```css
.clearfix {
    display: block;
    zoom: 1;
    &:after {
        content: "";
        display: block;
        font-size: 0;
        height: 0;
        clear: both;
        visibility: hidden;
    }
}
```
在包含浮动元素的父元素上设置该类即可。

### 参考

- 张鑫旭 《CSS世界》
- https://mp.weixin.qq.com/s/8eAfz_I5xIhh7oFRifxaFw