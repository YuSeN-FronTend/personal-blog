---
title: vue基础
date: 2023-3-17 16:11
categories: vue
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

# Class与Style绑定

操作元素class列表和内联样式都是数据绑定的常见需求。因为他们都是attribute，所以我们可以使用`v-bind`去处理它们: 只需要通过表达式计算出字符串结果即可。表达式类型除了字符串之外，还可以是对象或数组。

## 绑定 HTML Class

### 对象语法

```html
<div :class="{ active: isShow }">vue</div>
```

以上代码确定div是否有`active`样式取决于`isShow`的布尔值是否为真

并且动态添加类名指令也可以和普通类共存，如以下代码

```html
<div class="box" :class="{ active: isShow }">vue</div>
```

动态类还可如以下方式编写

```vue
<template>
    <div class="container">
        <div :class="objStyle">vue</div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            objStyle: {
                active: true,
                isShow: false
            }
        }
    },
}
</script>

<style scoped>
.active {
    color: red;
}

.isShow {
    display: none;
}
</style>

```

同样只会为div添加上`class="active"`, 甚至绑定的值还可为计算属性

### 数组语法

```vue
<template>
    <div class="container">
        <div :class="[activeClass, isShowClass]">vue</div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            activeClass: 'active',
            isShowClass: 'isShow'
        }
    },
}
</script>

<style scoped>
.active {
    color: red;
}

.isShow {
    display: none;
}
</style>

```

以上写法可以添加上两个类名，还可以嵌套使用三元表达式和对象

### 用在组件上

在组件上添加类名会被直接添加上去，并且上述方法依旧有效

## 绑定内联样式

### 对象语法

```vue
<template>
    <div class="container">
        <div :style="{ color: colorStyle, fontSize: fontStyle + 'px'}">vue</div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            colorStyle: 'red',
            fontStyle: 30
        }
    },
}
</script>
```

其中的css property名可以用驼峰式或短横线分隔(记得用引号括起来)

但是通常情况下，直接绑定一个样式对象通常更好，这会让模板更清晰：

```vue
<template>
    <div class="container">
        <div :style="objStyle">vue</div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            objStyle:{
                color: 'red',
                fontSize: '30px'
            }
        }
    },
}
</script>
```

### 数组语法

```html
<div v-bind:style="[baseStyles, objStyle]"></div>
```

数组语法可以将多个样式对象应用到同一个元素上

### 自动添加前缀

当使用动态样式使用添加**浏览器引擎前缀**的CSS property时，如transform，Vue.js会自动侦测并添加响应的前缀。

### 多重值

从2.3.0起，`style`绑定中的property提供一个包含多个值的数组，常用于提供多个带前缀的值：

```html
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

这样写只会渲染数组中最后一个被浏览器支持值

# 条件渲染

## `v-if`

它的意思在于条件性的渲染一块内容，这块内容只在指令表达式返回`truthy`值(真值)的时候被渲染

```vue
<template>
    <div class="container">
        <div v-if="isShow">vue</div>
        <div v-else>look me</div>
        <button @click="isShow = !isShow">change</button>
    </div>
</template>

<script>
export default {
    components: {},
    data() {
        return {
            isShow: true
        }
    },
}
</script>
```

上述代码，每次点击按钮都会显示不同的内容，`v-else`必须与`v-if`连用，当`v-if`为假值时，`v-else`就会渲染

## v-else-if

可以和v-if连用，tab切换时经常用到此属性

## v-show

此方法和`v-if`用法和作用都差不多，都是控制DOM元素的显示与隐藏

## `v-if`  vs  `v-show`

`v-if`是直接销毁DOM元素，重新渲染直接重建，而`v-show`是给DOM元素添加`display: none`属性，只是不显示该DOM元素，而不是销毁他们。

## v-if与v-for一起使用

当`v-if`与`v-for`一起使用时，`v-for`具有比`v-if`更高的优先级(在vue3中是`v-if`比`v-for`更高)

# 列表渲染

## 使用`v-for`把一个数组对应为一组元素

```vue
<template>
    <div>
        <ul>
            <li v-for="item in items" :key="item.id">
                {{ item.msg }}
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    components: {},
    data() {
        return {
            items:[
                {
                    msg: 'msg1',
                    id: 1
                },
                {
                    msg: 'msg2',
                    id: 2
                }
            ]
        }
    },
}
</script>
```

在`v-for`中，我们可以访问到所有父作用域中的property，`v-for`还支持第二个参数即当前项的索引 

## 在`v-for`里使用对象

可以用`v-for`来遍历一个对象property，可以接受三个参数，第一个参数是value，第二个参数是键名，第三个参数是对应的索引

```vue
<template>
    <div>
        <ul>
            <li v-for="item,name,index in object">
                {{ item }}:{{ name }}:{{ index }}
            </li>
        </ul>
    </div>
</template>

<script>
export default {
    components: {},
    data() {
        return {
            object: {
                title: 'How to do lists in Vue',
                author: 'Jane Doe',
                publishedAt: '2016-04-10'
            }
        }
    },
}
</script>

<style scoped></style>

```

## 维护状态

当使用vue中`v-for`进行列表渲染时，除非渲染内容非常简单，否则最好添加key属性，且key的值一定是唯一的，因为这能帮助浏览器更好的识别出每一个被渲染的DOM元素，从而提高性能。

## 数据更新检测

### 变更方法

vue将被侦听的数组的变更方法进行了包裹，所以他们也将会触发视图更新。这些被包裹过的方法包括：

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

### 替换数组

有变更方法就有非变更方法，例如`filter()`、`concat()`和`slice()`。它们不会变更数组，而总是返回一个新数组，当使用非变更方法时，可以用旧数组替换新数组。并且vue为了使得DOM元素得到最大范围的重用而实现了一些智能的启发式方法，所以用一个含有相同元素的数组去替换原来的数组是非常高效的操作。

### 注意事项

由于js的限制，vue不能检测数组和对象的变化

## 在`v-for`中使用范围值

`v-for`可以接受整数，并且会把模板重复对应次数

```html
<div>
  <span v-for="n in 10">{{ n }} </span>
</div>
```

## 在组件上使用`v-for`

其实和在任何普通元素上使用一样，但是需要注意的是，在2.2.0+以及之后的版本，在组件上使用时必须绑定`key`属性

# 事件处理

## 监听事件

可以用v-on来进行监听事件

```vue
<template>
    <div>
        {{ num }}
        <button v-on:click="num++">num+1</button>
    </div>
</template>

<script>
export default {
    components: {},
    data() {
        return {
            num: 0,
        }
    }
}
</script>
```

## 事件处理方法

许多事件的逻辑较为复杂，所以直接将js代码写在模板中是不现实的，因此`v-on`还可以接受一个需要调用方法的名称

```vue
<template>
    <div>
        {{ num }}
        <button v-on:click="add">num+1</button>
    </div>
</template>

<script>
export default {
    components: {},
    data() {
        return {
            num: 0,
        }
    },
    methods:{
        add() {
            this.num++;
        }
    }
}
</script>
```

## 内联处理器中的方法

除了直接绑定方法，还可以在内联javaScript语句中调用方法

```vue
<template>
    <div>
        <button v-on:click="add(2)">math</button>
        <button v-on:click="add(5)">math</button>
    </div>
</template>

<script>
export default {
    methods:{
        add(e) {
            if(typeof e === 'number'){
                alert(e++);
            }
        }
    }
}
</script>
```

## 事件修饰符

在事件处理程序中调用`event.preventDefault()`或`event.stopPropagation()`是非常常见的需求。尽管我们可以在方法中轻松实现这点，但更好的方式是方法只有纯粹的数据逻辑，而不是去处理DOM事件细节，所以vue提供了以下修饰符

- `.stop`

  阻止单击事件继续传播

- `.prevent`

  提交事件不再重载页面

- `.capture`

  内部元素触发的事件现在此处理，然后再交给内部元素进行处理

- `.self`

  事件不是从内部元素触发的

- `.once`(2.1.4新增)

  事件只会触发一次

- .`passive`(2.3.0新增)

  滚动的默认行为将会立即触发，不会等待`onScroll`完成，这其中包含event.preventDefault()的情况。此修饰符尤其能提升移动端的性能

**注意**：修饰符可以连用，但要注意连用的顺序，并且`.passive`和`.prevent`不要一起使用，因为`.prevent`会被忽略且浏览器会展示一个警告

## 按键修饰符

vue还为我们提供了按键修饰符，比如一下代码在聚焦input框按下回车就会触发

```html
<input type="text" v-on:keyup.enter="add">
```

更多按键码可查看[vue官网](https://v2.cn.vuejs.org/v2/guide/events.html)

## 系统修饰符

vue2.1.0新增的。可以用下面的修饰符来实现仅在按下相应按键时才会触发鼠标或键盘事件的监听器

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

### `.exact`修饰符

vue2.5.0新增的，允许自己控制由精确的系统修饰符组合触发的事件，也就是直接绑定.ctrl，再它和alt以前下也会触发，加此属性，只有按下ctrl才会触发

### 鼠标按钮修饰符

vue2.2.0新增的

- `.left`
- `.right`
- `.middle`

这些修饰符会限制处理函数仅响应特定的鼠标按钮

# 表单输入绑定

## 基础用法

vue中可以使用`v-model`来完成双向绑定，但是`v-modal`在内部为不同的输入元素使用不同的property并抛出不同的事件：

- text和textarea元素使用value property和input事件
- checkbox和radio使用checked property 和 change事件
- select字段将value作为prop并将change作为事件

## 修饰符

### `.lazy`

在默认情况下，`v-model`在每次input发生修改时都会进行同步修改，添加lazy修饰符之后，只有每次填写完成(就是失去焦点)时才会同步更新。

```html
<input type="text" v-model.lazy="message">
```

### `.number`

如果想要用户的输入类型为number即可以给`v-model`添加number属性

```html
<input type="text" v-model.number="message">
```

### `.trim`

如果要自动过滤掉前后的空白字符，可以给`v-model`添加`trim`修饰符

```html
<input type="text" v-model.trim="message">
```

