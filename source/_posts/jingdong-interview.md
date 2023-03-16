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

# 6、arguments详解

## 概念

arguments是一个对应传递给函数的参数的类数组对象(有长度但是并没有数组的内置方法)

```js
function foo(){
    console.log(arguments); // [Arguments] { '0': 1, '1': 'a', '2': true, '3': [ 'abc', 2 ] }
}

foo(1,'a',true,['abc',2])
```

## 将arguments转换为数组的方法

- Array.from()

  ```js
  function foo() {
      console.log(Array.from(arguments)); // [ 1, 'a', true, [ 'abc', 2 ] ]
  }
  
  foo(1, 'a', true, ['abc', 2])
  ```

- 扩展运算符

  ```js
  function foo() {
      console.log([...arguments]); // [ 1, 'a', true, [ 'abc', 2 ] ]
  }
  
  foo(1, 'a', true, ['abc', 2])
  ```

## 属性

- arguments.callee

  调用函数自己本身，类似于递归，不同的是此方法可以应用在匿名函数中，不需要知道函数名。

  ```js
  let a = 0;
  function foo(num) {
      if(num){
          a += num;
          arguments.callee(--num);
      }else{
          console.log(a); // 15
      }
  }
  foo(5)
  ```

  上面代码是一个简单的实现

- arguments.length

  ```js
  function foo() {
     console.log(arguments.length); // 4
  }
  
  foo(1, 'a', true, ['abc', 2])
  ```

  这也是arguments被称作类数组的理由，因为有长度

- arguments[@@iterator]

  语法为  arguments[Symbol.iterator]

  ```js
  function foo() {
     console.log(arguments.length); // 4
  }
  
  foo(1, 'a', true, ['abc', 2])function foo() {
     for(let i of arguments) {
         console.log(i);// 1   a   true   [ 'abc', 2 ]
     }
  }
  
  foo(1, 'a', true, ['abc', 2])
  ```

  能进行for of循环就说明它会返回一个新的Array迭代器

# 7、父子组件的生命周期

父组件的beforeCreate --> 父组件的created --> 父组件的beforeMount --> 子组件的beforeCreate --> 子组件的created --> 子组件的beforeMount --> 子组件的mounted --> 父组件的mounted --> 父组件的beforeUpdate --> 子组件的beforeUpdate --> 子组件的updated --> 父组件的updated --> 父组件的beforeDestroy --> 子组件的beforeDestroy --> 子组件的destroyed --> 父组件的destroyed

# 8、组件间通信

vuex这里不做记录

组件间通信一般分为三种

- 父子组件之间通信
- 兄弟组件之间通信
- 隔代组件之间通信

## 父子组件之间通信

- props

  父组件直接给子组件传值，子组件用props去接收

  ```html
  <!-- 父组件 -->
  <template>
      <div class="father">
          <son :text="text"></son>
      </div>
  </template>
  
  <script>
  import son from './son.vue';
  export default {
      components: {
          son
      },
      data() {
          return {
              text: '123'
          }
      },
  }
  </script>
  ```
  
  ```js
  // 子组件
  <template>
      <div class="son">
          {{ text }}
      </div>
  </template>
  
  <script>
  export default {
      props:['text'],
      data() {
          return {
          }
      }
  }
  </script>
  ```
  
- $emit父组件给子组件传值

  ```vue
  <!-- 父组件 -->
  <template>
      <div class="father">
          <son @titleChange="titleChange"></son>
          <div>{{ title }}</div>
      </div>
  </template>
  
  <script>
  import son from './son.vue';
  export default {
      components: {
          son
      },
      data() {
          return {
              title: ''
          }
      },
      methods: {
          titleChange(title) {
              this.title = title
          }
      }
  }
  </script>
  ```

  ```vue
  <!-- 子组件 -->
  <template>
      <div class="son">
          <button @click="handleClick">传值</button>
      </div>
  </template>
  
  <script>
  export default {
      methods: {
          handleClick() {
              this.$emit('titleChange', 'new Title')
          }
      }
  }
  </script>
  ```

  

## 兄弟间组件通信

### 状态提升方法

就是给两个兄弟组件提供一个父组件负责处理数据，来解决兄弟组件间的通信

```vue
<!-- 父组件 -->
<template>
    <div class="father">
        <son :title="title" :change-title="changeTitle"></son>
        <son1 :title="title1" :change-title="changeTitle1"></son1>
    </div>
</template>

<script>
import son from './son.vue';
import son1 from './son1.vue';
export default {
    components: {
        son,
        son1
    },
    data() {
        return {
            title:'',
            title1:''
        }
    },
    methods:{
        changeTitle(){
            this.title = 'son'
        },
        changeTitle1(){
            this.title1 = 'son1'
        }
    }
}
</script>
```

```vue
<!-- 子组件1 -->
<template>
    <div class="son">
        {{ title }}
        <button @click="changeTitle">change sonTitle</button>
    </div>
</template>

<script>
export default {
    props:{
        title:{
            type: String
        },
        changeTitle: Function
    },
}
</script>
```

```vue
<!-- 子组件2 -->
<template>
  <div class="son1">
    {{ title }}
    <button @click="changeTitle">change son1Title</button>
  </div>
</template>

<script>
export default {
 components: {},
 props:{
    title:{
        type:String
    },
    changeTitle: Function
 },
}
</script>
```

## 隔代组件之间通信

- $attrs

  ```vue
  <!-- 父组件 -->
  <template>
      <div class="father">
          <son :title="title"></son>
      </div>
  </template>
  
  <script>
  import son from './son.vue';
  export default {
      components: {
          son,
      },
      data() {
          return {
              title:'123',
          }
      },
  }
  </script>
  ```

  ```vue
  <!-- 子组件1 -->
  <template>
      <div class="son">
          {{ title }}
          <grandson v-bind="$attrs"></grandson>
      </div>
  </template>
  
  <script>
  import grandson from './grandson.vue';
  export default {
      inheritAttrs: true,
      components:{
          grandson
      },
  }
  </script>
  ```

  ```vue
  <!-- 孙子组件 -->
  <template>
    <div class="son1">
      {{ title }}
    </div>
  </template>
  
  <script>
  export default {
   components: {},
   props:{
      title:{
          type:String
      },
   },
  }
  </script>
  ```

- $parent/$children

  但是并不推荐使用。

# 9、虚拟DOM

## 虚拟DOM的概念

- 起源

  虚拟DOM最先是facebook团队提出的，最先运用在react中，vue在vue2.0版本引入了虚拟DOM的概念

- 虚拟DOM的含义

  **虚拟DOM是相对于浏览器渲染出来的真实DOM**

  以往，我们改变更新页面，只能通过首先查找DOM对象，再进行修改DOM的方式来达到目的。但这种方式**相当消耗计算资源**。(因为每次查询DOM都需要遍历整棵DOM树)

  虚拟DOM树就是用对象的方式来描述真实的DOM，并且对象与真实DOM建立了一对一对应的关系，那么每次DOM修改，我通过找到对应的对象，也就找到了响应的DOM节点，在对其进行更新。这样的话，就是节省性能，因为**js对象的查询，比起对整个DOM树的查找，所消耗的性能要少。**

## 虚拟DOM的优缺点

优点：

- 降低浏览器性能消耗

  因为JavaScript的运算速度远大于DOM操作的执行速度，因此，运用patching算法来计算出真正需要更新的节点，最大限度地减少DOM操作，从而提高性能。

- diff算法 减少重绘和回流

  通过diff算法，优化遍历，对真实DOM进行打补丁式地新增，修改，删除，实现局部更新，减少回流和重绘。

  **扩展**：当页面内容和布局没有发生改变，只是元素外观发生改变就会产生重绘。当页面地一部分内容或布局发生改变，重新构建页面就会产生回流。产生重绘不一定会造成回流，产生回流不一定会造成重绘。

- 跨平台

  虚拟DOM本质上是一个javaScript对象，而DOM与平台强相关，相比之下虚拟DOM更适合跨平台开发。例如: **服务器渲染(SSR)、week开发**等等。

缺点：

- 首次显示要慢些

  首次渲染大量DOM是，由于多了一层虚拟DOM地计算，会比innerHTML插入慢

- 无法进行极致优化

  虽然虚拟DOM+合理地优化，足以应对绝大部分应用的性能需求，但在一些性能要求极高地应用中，无法进行针对性地机制优化。

  

# 10、a == 1 && a == 2 && a == 3常用解法

先说以下双等号的比较规则：如果操作数是之一是对象，另一个是数字或字符串，会尝试使用对象的valueOf和toString方法将对象转换为原始值。

## 隐式转换之重写valueOf() / toString()

这种是**最常见**的解法，直接重写valueOf或toString。

经过分析可以不难看出，a一定不是一个原始数据类型，那a只能是一个复杂数据类型，常见的又对象，数组。

思路：构造一个对象，有一个属性i=1，改写valueOf方法，每次返回i，并让i++。以下是代码实现。

```js
const a = {
    i: 1,
    valueOf(){
        return this.i++;
    }
}

if(a == 1 && a == 2 && a == 3) {
    console.log(a); // { i: 4, valueOf: [Function: valueOf] }
}
```

## 使用Object.defineProperty改写数组join方法

这里对象也可以是数组，因此数组与数字的比较也会尝试调用valueOf()和toString()方法，如下面例子

```js
let arr = [1,2,3]
let str = '1,2,3';
console.log(arr == str); // true
```

实际上，数组上的toString方法和join方法存在一定的联系，具体表现为：

- `Array.prototype.toString() === Array.prototype.join()`(对于任何数组，默认方法也成立，且方法无参数)
- 数组在调用`toString`方法的时候，如果`toString`方法没有被重写，而是`join`方法被重写了，则数组就会去调用`join`方法

比如我们改写数组的join方法，没有改变toString方法，那么调用toString方法就会执行join方法，如下列代码所示：

```js
let arr = [1,2,3];
arr.join = () => { console.log('join被重写了'); };
arr.toString(); // join被重写了
```

以上分析表明：对于数组而言，隐式调用toString相当于执行了join方法。对于数组我们无法在内部直接添加方法，所以我们使用`Object.defineProperty()`

### Object.defineProperty() 

##### 语法

> Object.defineProperty(obj, prop, descriptor);

##### 参数

- `obj`: 要定义属性或方法的目标对象
- `prop`: 要定义的属性或方法名
- `descriptor`: 一个对象，包含value等属性

思路：让数组的`join`方法赋值给`shift`方法，这样每次与数组比较都执行了`join`方法，间接执行了`shift`，弹出数组中的第一个元素。

```js
let a = Object.defineProperty([1,2,3], 'join', {
    get() {
        return () => this.shift();
    }
})

if(a == 1 && a == 2 && a == 3) {
    console.log(a); // []
}
```

## 使用Proxy代理构造get捕获器

ES6的新特性Proxy，能够代理对象的一些行为，例如在获取目标对象的一些属性和方法的时候进行拦截或者进一步处理后再返回结果。

思路：和第一种改写`valueOf`方法然后每次递增`i`的方法思想是一样的，只是换成了`proxy`代理模式，下面是代码实现

```js
let a = new Proxy({ v: 1}, {
    get(target, property, receiver){
        // 隐式转换会调用Symbol.toPrimitive，这是一个函数
        if (property === Symbol.toPrimitive){
            // 函数属性，所以要返回一个函数，会被执行
            return () => target.v++
        }
    }
})

if(a == 1 && a == 2 && a == 3) {
    console.log(a); //{ v: 4 }
}
```

# 11、js基本类型之——bigInt

`bigInt`是ES11引入的新的基本数据类型。`bigInt`数据类型的目的是比`Number`数据类型支持的范围更大的整数值，以任意精度表示整数，使用`bigInt`解决了之前`Number`整数溢出的问题

## 表示方式

只需在普通整形后加n即可

```js
let a =  521n;
console.log(a, typeof a); // 521n bigint
```

## BigInt函数

可以将普通整数值转化为大整型的值，但是需要注意的是不能使用浮点数进行转换

```js
console.log(BigInt(521)); // 521n
console.log(BigInt(0.2)); // RangeError: The number 0.2 cannot be converted to a BigInt because it is not an integer at BigInt(<anonymous>)
```

## 大数值运算

```js
let max = Number.MAX_SAFE_INTEGER; // Number的最大值
console.log(max); // 9007199254740991
console.log(max + 1); // 9007199254740992
// 超过最大范围，运算就会出错
console.log(max + 2); // 9007199254740992

// BigInt数据类型不能直接和普通数据类型进行运算
console.log(BigInt(max) + BigInt(2)); // 9007199254740993n
console.log(BigInt(max) + BigInt(50)); // 9007199254741041n
```

