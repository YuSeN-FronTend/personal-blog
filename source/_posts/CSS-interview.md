---
title: css部分面试总结
date: 2023-3-29 15:33
categories: 面试
---

# CSS篇

css直接的面试问答题相对较少，更多的是需要能够当场敲代码实现功能，一般来说备一些常见的布局，熟练掌握flex基本就没什么问题

## 什么是BFC

Block Formatting context  块格式化上下文

BFC是一个独立的的渲染区域，相当于一个容器，这个容器内的样式布局不会受到外界影响

比如浮动元素、绝对定位、overflow除visble以外的值、display为inline/tabel-cells/flex都能构建BFC

常常用于解决

1. 处于同一个BFC的元素外边距会产生重叠(此时需要将它们放在不同BFC中)
2. 清除浮动(float),， 受用BFC包裹浮动的元素即可
3. 组织元素被浮动元素覆盖，应用于两列式布局，左边宽度固定，右边内容自适应宽度(左边float，右边overflow)

## 伪类和伪元素的使用场景

### 伪类

定义：当元素处于特定状态时才会运用的特殊类

开头为冒号的选择器，用于选择处于特定状态的元素。比如`:first-child`选择第一个子元素；`:hover`悬浮在元素上会显示；`:focus`用键盘选定元素时激活；`:link` + `:visted`点击过的链接的样式；`:not`用于匹配不符合参数选择器的元素；`:disabled`匹配禁用的表单元素

### 伪元素

定义：伪元素创建一些不再文档书中的元素，并为其添加样式

比如我们可以通过`::before`来在一个元素前增加一些文本，并为这些文本添加样式，虽然用户可以看到文本，但是这些文本根本不在文档书中。

示例代码

`::before`在被选元素前插入内容。需要使用content属性来指定要插入的内容。被插入的内容实际上不在文档树中

```css
h1::before{
	content: "Hello";
}
```

`::first-line`匹配元素中第一行的文本，下面代码更改元素第一行文本的颜色

```css
h1::first-line{
    color: red
}
```

## src和href区别

- href是Hypertext Reference的简写，表示超文本引用，指向网络资源所在位置。href用于在当前文档和引用资源之间确立联系。
- src是source的简写，目的是把文件下载到html页面中去，src用于替换当前内容
- 浏览器的解析方式
  - 当浏览器遇到href会并行下载资源并且不会停止对当前文档的处理(同时也是为什么建议使用link方式加载CSS，而不是使用@import方式)
  - 当浏览器解析到src，会暂停其他资源的下载和处理，直到将该资源加载会执行完毕。(这也是script标签放在底部而不是头部的原因)

## 不定宽高元素的水平垂直居中

- flex

  ```css
  <div class="wrapper flex-center">
          <p>center</p>
      </div>
  
  .wrapper{
          width: 900px;
          height: 300px;
          border: 1px solid #ccc;        
      }
      .flex-center{
          display: flex;
          justify-content: center;
          align-items: center;
      }
  ```

- flex + margin

  ```css
  <div class="wrapper">
          <p>center</p>
      </div>
  .wrapper{
          width: 900px;
          height: 300px;
          border: 1px solid #ccc;
          display: flex;     
      }
      .wrapper > p{
          margin: auto
      }
  ```

  

- transform + absolute

  ```css
  <div class="wrapper">
      <img src="test.png">
  </div>
  
  .wrapper {
      width: 300px;
      height: 300px;
      border: 1px solid #ccc;
      position: relative;
  }
  .wrapper > img {
      position: absolute;
      left: 50%;
      top: 50%;
      tansform: translate(-50%, -50%)
  }
  ```

  该方法只对行内元素(a、img、label、br、select等)，用于块级元素会有问题。

- display: table-cell

  ```css
  <div class="wrapper">
          <p>center</p>
      </div>
  .wrapper{
          width: 900px;
          height: 300px;
          border: 1px solid #ccc;
          /* 让元素像表格一样 */
          display: table-cell;
          /* 占父盒子高度的中间部位(垂直) */
          vertical-align: middle;
          /* 水平居中 */
          text-align: center;
      }
  ```

  