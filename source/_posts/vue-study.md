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

## 指令

指令是带有`v-`特殊前缀的attribute。指令的值预期是单个js表达式。指令就是当表达式的值发生改变时，能够响应式的作用于DOM。

```html
<p v-if="seen">看得见我</p>
```

跟去seen的真假来插入或删除p标签

### 参数

一些指令能够接收一个参数，在指令名称之后以冒号表示。例如**v-bind**指令可以用于响应式地更新HTML attribute，**v-on**指令可以用来监听DOM事件。

### 动态参数

从vue 2.6.0开始，可以用方括号括起来地javaScript表达式作为一个指令的参数：

```html
<!-- 在data函数中定义的str如果值为value，则v-bind:[str]等同于v-bind:value -->
<input type="text" v-bind:[str]="bol">
```

当然v-on也同样可以动态绑定

```html
<!-- 同样的，在data函数中定义的str如果值为click，则v-on:[str]等同于v-on:click -->
<button v-on:[str]="num++">num+1</button>
```

**对动态参数的值的约束**

动态参数预期会求出一个字符串，异常情况下值为`null`。这个特殊`null`值可以被显示地用于移除绑定。任何其他非字符串类型地值都将会触发一个警告。

**对动态参数表达式的约束**

动态参数表达式有一些语法约束，因为某些字符，如空格和引号，放在HTML attribute名里是无效的。并且还需要注意避免用大写字符来命名键名，因为浏览器会把attribute名全部转换为小写。

### 修饰符

修饰符是以符号`.`指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。例如`.prevent`修饰符是告诉`v-on`指令对于触发的事件调用`event.preventDefault()`:

```html
<form v-on:submit.prevent="onSubmit">...</form>
```

## 缩写

vue为`v-bind`和`v-on`两个最常用的指令都提供了简写方式

- `v-bind`简写为`:`
- `v-on`简写为`@`

# 计算属性和帧听器

如果直接在模板中写入过多的逻辑如以下所示

```html
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

这就会违背了模板的初衷(用于简单运算)，并且在模板中放入太多逻辑会使代码可读性变差并且难以维护。所以`computed`计算属性应运而生。

```js
computed: {
    sum(){
      return this.num+1
    }
  },
```

## 计算属性和监听属性

vue提供了一种更通用的方式来观察和响应vue实例上的数据变动：**侦听属性**，当有一些数据需要随着其他数据的改变而改变时，我们很容易滥用`watch`，所以有如此情况，还是推荐使用计算属性。

```js
// 此段监听代码不如计算属性，如下段代码
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```

```js
// 比上段代码更轻量
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```

## 计算属性的setter

计算属性默认只有getter，但在需要时也可以提供一个setter

```js
computed: {
    fullName:{
      get: function() {
        return this.firstName + ' ' + this.lastName;
      },
      set: function(newValue) {
        let names = newValue.split(' ');
        this.firstName = names[0];
        this.lastName = names[names.length - 1]
      }
    }
  },
```

## 侦听器

虽然计算属性更加常用，但是一些特定的场景也确实需要自定义的侦听器，这也是vue提供`watch`方法的原因。

```html
 <button @click="num++">num+1</button>
```



```js
watch:{
    num(newValue, oldValue) {
      console.log(newValue, oldValue);
    }
  },
```

上述代码当每次点击按钮，num都会+1，并且watch监听着num的变化，所以每次num改变时，控制台都会打印响应数据。