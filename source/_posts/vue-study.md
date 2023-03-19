---
title: vue学习
date: 2023-3-17 16:11
categories: javaScript
---

# vue介绍

vue是一套用于构建用户界面的渐进式框架。vue的核心库只关注视图层，使用的是MVVM(model-view-viewmodel)体系结构。另一方面，vue也完全有能力驱动采用单文件组件和vue生态系统支持的库开发的复杂单页面应用。

# vue知识点概况

## 创建一个vue实例

```js
var vm = new Vue({
// 选项
})
```

## 数据和方法

当一个vue实例被创建完成时，他将data对象中所有的`property`都加入到Vue的响应式系统中，当这些`property`的值发生改变时，视图将会产生响应，即匹配为最新的值。

```js
// 内部的数据都是响应式的
data() {
    return {
      str: 'abc',
      num: 123,
      bol: true,
      und: undefined,
      nu: null,
      obj: {},
      arr: [],
      list: ['孙悟空', '猪八戒', '123']
    }
  },
```

但是如果使用`Object.freeze()`，传递一个响应式参数，他就不会再追踪变化。(但是只针对于对象)

## 生命周期

熟悉生命周期，就可以在最恰当的时候去书写我们的逻辑，让代码在合理的地方触发从而最大限度地节省性能并且维护性更高。

vue的生命周期一共有8个：

- beforeCreate

- created

  数据初始化完成，方法页面调用，但是DOM没有渲染

- beforeMount

- mounted

  DOM和方法全部挂载完成

- beforeUpdate

- updated

- beforeDestroy

- destroyed

详情见[vue官网](https://v2.cn.vuejs.org/v2/guide/instance.html)

# 模板语法

vue.js使用了基于HTML的模板语法，允许开发者声明式地将DOM绑定至底层vue实例的数据。所有vue.js的模板都是合法的HTML，所以能被遵循规范的浏览器和HTML解析器解析。

在底层的实现上，vue将模板编译成虚拟DOM渲染函数。结合响应系统，vue能够智能的计算出最少需要重新渲染多少组件，并把DOM操作次数减到最少。

如果熟悉虚拟DOM并且偏爱js的原始力量，可以不用模板，**直接写渲染(render)函数**，使用可选的JSX语法。(例如拖拽生成工单，把工单中的元素封装起来变成JSON文件，再把JSON文件渲染成JSX语法，用render函数渲染成工单，从而实现动态生成工单并与后端完成交互。)

## 插值

### 文本

数据绑定最长用的语法就是`mustache`语法的文本插值

```html
<span>{{msg}}</span>
```

`mustache`标签将会被替代为对应数据对象上的msg property的值，无论何时，只要数据对象上的msg property发生了改变，插值处的内容也会随之更新。

如果需要一次性插值，在元素上添加`v-once`属性即可，但是要注意这可能会影响到该节点上的其他数据绑定。

```html
<span v-once>{{msg}}</span>
```

### 原始HTML

双大括号会将数据解释为普通文本，而非HTML代码。为了输入真正的HTML，需要属于`v-html`，但是我们不能直接使用v-html来复合局部模板，因为vue不是基于字符串的模板引擎。

> 在站点动态渲染任意的HTML可能会非常危险，容易受到XSS攻击，所以一定只对可信内容使用HTML插值，绝不要对用户提供的内容使用插值。**v-html为了防止XSS攻击，只能识别html语法，element等封装好的组件无法被指别**

### Attribute

mustache语法不能应用HTML attribute上，遇到这种情况应该使用v-bind指令

```html
<span v-bind:id="boxId"></span>
```

这就是我们常说的动态绑定，大部分属性想要绑定动态值都可以使用此方法，所以由于v-bind使用的极其频繁，所以一般缩写为`:`

### 使用javaScript表达式

```js
{{ number + 1}}
{{ ok ? 'YES' : 'NO'}}
```

但是不太推荐此方法，逻辑最好还是封装在method里