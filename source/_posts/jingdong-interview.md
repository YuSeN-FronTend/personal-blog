---
title: 面经总结
date: 2023-3-8 10:55
categories: 面试
---

# 1、href和src的区别

## href

href是指向网络资源所在位置，建立和当前元素或当前文档之间的链接，用于超链接。表示超文本应用，用在**link**和**a**等元素上，**href**是引用和页面关联，是在当前元素和引用资源之间建立联系

## src

src是指向外部资源的位置，指向的内部会迁入到文档中当前标签所在的位置；在请求**src**资源时会将其指向的资源下载并应用到当前文档中，例如js脚本，img图片和frame等元素。

## 区别

当浏览器遇到href会并行下载资源并且不会停止对当前文档的处理。(同时也是为什么建议用link方式加载CSS，而不是使用@import方式)。

当浏览器解析到src，会暂停其他资源的下载和处理，直到该资源加载或执行完毕。(这也是script标签为什么放在底部而不是头部的原因)

# 2、HTML5语义化

## 概念

HTML5的语义化指的是合理正确的使用语义化标签来创建页面结构

## 语义化标签

- header

  此元素表述了文档的头部区域

- nav

  此元素标签定义导航链接的部分(在header下面)

- section

  此标签定义了文档中的接，比如章节、页眉、页脚或文档中的其他部分

- article

  标签定义独立的内容(包含section)

- aside

  标签定义页面主区域内容之外的内容(比如侧边栏)

- figcaption

  标签定义figure元素的标题，它应该被至于figure元素的第一个或最后一个子元素的位置

- figure

  标签规定独立的流内容(图像、图片、照片、代码等等)

- footer

  元素描述了文档的底部区域

## 语义化的优点

- 在没有CSS样式的情况下，页面整体也会呈现很好的结构效果
- 代码结构清晰，易于阅读
- 利于开发和维护 方便其他设备解析(如屏幕阅读器)根据语义渲染网页
- 有利于搜索引擎的优化(SEO)，搜索引擎爬虫会根据不同的标签来赋予不同的权重

# 3、!DOCTYPE

它是一种标准通用标记语言的文档类型声明，它的目的是告诉浏览器用什么解析器来解析当前语言。

# 4、CSS盒子模型

## 盒子的组成

盒子从内到外分为四个部分：

- margin(外边距)
- border(边框)
- padding(内边距)
- content(内容)

可以看出前三个时CSS属性，因此可以通过这三个属性来控制盒子的这三个部分，而content则是HTML元素的内容

## 盒子的大小

盒子的大小指的是盒子的宽度和高度，但是并不是width和height，而是以下等式

> 盒子的宽度 = width + padding-left + padding-right + border-left + border-right + margin-left + margin-right
> 盒子的高度 = height + padding-top + padding-bottom + border-top + border-bottom + margin-top + margin-bottom

上面是默认情况下的计算方法，另一种情况，width和height设置的就是盒子的宽度和高度，宽度和高度的计算方式由box-sizing

box-sizing有以下值可选

- content-box

  默认值，width和height分别应用到元素的内容框。在宽度和高度和高度之外绘制元素的内边距、边框、外边距。

- border-box

  告诉浏览器盒子的宽度(width)包括盒子的边框(border)和内边距(padding)，但是不包括margin。举个例子，你给一个盒子添加一个属性并且给他设置宽度为300px，那么添加padding和border都是在300px以内添加的。

- inherit

  表示从父盒子来继承box-sizing的值

# 5、BFC

BFC(Block Formatting Context) 块格式化上下文。说通俗一点，BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。

## 常见创建BFC的方式：

- 浮动元素(元素的float不是none，指定float为left或者right就可以创建BFC)
- 绝对定位元素(元素的position为absolute或fixed)
- display: inline-block, display: table-cell, display: flex, display: inline-flex
- overflow指定除了visible的值

## BFC的特点

- 在BFC中，块级元素是从顶端开始垂直的一个接一个排列。(不在BFC中的块级元素也会垂直排列)
- 如果两个块级元素都在同一个BFC中，他们上下的margin会重叠，以较大的为准。但是如果两个块级元素分别在不同的BFC中，它们的上下边距就不会重叠了而是两者之和
- BFC的区域不会与浮动的元素区域重叠，也就是说不会与浮动盒子产生交集，而是紧贴浮动边缘。
- 计算BFC高度时，浮动元素也参与计算，BFC可以包含浮动元素。(利用这个特性可以清除浮动)
- BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。

## BFC的应用

### 解决外边距折叠问题

此问题只会发生在同一BFC块级元素之间

html: 

```html
<div class="div1"></div>
<div class="div2"></div>
```

css:

```css
.div1{
        width: 100px;
        height: 100px;
        background-color: aqua;
        margin-bottom: 10px;
    }
    .div2{
        width: 100px;
        height: 100px;
        background-color: cadetblue;
        margin-top: 20px;
    }
```

设置了两个外边距，但是只会显示更大的那个外边距，也就是说两个盒子间隔为20px

我们可以用BFC来解决这个问题，让两个盒子在不同的BFC中，原理就是BFC在页面上是一个隔离的独立容器，容器里面的元素不会对外边产生影响。

html:

```html
  <div class="wrap">
        <div class="div1"></div>
    </div>
    <div class="div2"></div>
```

css:

```css
.wrap{
        overflow: hidden;
    }
    .div1{
        width: 100px;
        height: 100px;
        background-color: aqua;
        margin-bottom: 10px;
    }
    .div2{
        width: 100px;
        height: 100px;
        background-color: cadetblue;
        margin-top: 20px;
    }
```

需要注意的是，如果指定position属性为absolute和fixed，或者float指定为left、right也可以创建BFC，但是这个元素会从文档流中移除，不占据页面空间，并且可以和其他元素重叠，导致下面的div把上面的div覆盖。

### 制作两栏布局

BFC的区域不会与浮动元素的区域重叠，利用这个特性可以制作两栏布局

html:

```html
<div class="left"></div>
<div class="right"></div>
```

css:

```css
.left{
        width: 200px;
        height: 100px;
        background-color: green;
        float: left;
    }
    .right{
        height: 100px;
        background-color: red;
        overflow: hidden;
    }
```

### 清除元素内部的浮动

这里清除浮动的意思并不是清除你设置的元素的浮动属性，而是清除设置了浮动属性之后给别的元素带来的影响。例如我们给子元素设置浮动，那么父元素的高度就撑不开了。

原理是计算BFC的高度时，浮动元素也参与计算，利用这个特性可以清除浮动

html:

```html
<div class="div1">
        <div class="son1">a</div>
        <div class="son2">b</div>
    </div>
```

css:

```css
.div1{
        width: 150px;
        border: 1px solid red;
        overflow: hidden;
    }
    .son1,.son2{
        width: 100px;
        height: 100px;
        background-color:cadetblue;
        float: left;
    }
    .son2{
        background-color: aqua;
    }
```

