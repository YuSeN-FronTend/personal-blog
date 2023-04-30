---
title: vue3学习
date: 2023-4-29 9:20
categories: vue
---
# vue3

## 基础应用创建

脚手架安装过后，再main.js中需做如下配置。每个vue的应用都是通过createApp函数创建的，如果是我们最常用的根文件组件，可以从外部直接导入。最后使用mount('根组件类名')挂载上去

```js
import { createApp } from "vue";
import App from "./App.vue";
const app = createApp(App);
app.mount("#app");
```

## 模板语法

模板语法和vue2类似，不再重复学习

## 响应式基础

### `<script setup>`

由于在vue3中，大部分的配置项方法生命周期都在setup中去书写，并且书写完毕需要大量return，所以这样写可以大幅度的简化代码

### DOM更新时机

在DOM渲染完成后再执行里面的逻辑，就是nextTick()，和vue2中的$nextTick类似，所以不多赘述

### 深层响应式

vue3的响应式默认都是深层次的，也就是不论有多少层，他都是响应式的，即便如下这种错综复杂的结构也可以

```vue
<template>
  <div>
    <button @click="countAdd">{{ obj.b.d.f.g.h }}</button>
  </div>
</template>

<script setup>
      import { getCurrentInstance, onMounted, reactive, ref } from 'vue';
      let obj = reactive({
        a: 1,
        b: {
          c: 2,
          d:{
            e:3,
            f:{
              g:{
                h:29
              }
            }
          }
        }
      })
      function countAdd() {
        obj.b.d.f.g.h++;
      }
</script>
```

### 响应式代理vs原始对象

值得注意的式reactive返回的是一个原始对象的Proxy，它和原始对象是不相等的。并且为了保证访问代理的一致性，在一个Proxy对象上再调用reactive则返回它自己，如下所示

```js
	  let obj = {
        count:123
      }
      let obj1 = reactive(obj)
      console.log(obj === obj1); // false
      console.log(obj1 === reactive(obj)); // true
      console.log(reactive(obj1) === obj1); // true
```

并且嵌套对象无论多少层，每一层都是原始对象的代理

### `reactive()`局限性

它只对数组，对象和Map，Set这样的集合类型有效，对于string，number和boolean这样的原始类型无效。

vue3的响应式系统是通过属性方位进行追踪的，所以我们必须保持对该响应式对象的相同引用，不然就会导致原始引用的响应式连接丢失，如下所示

```vue
<template>
  <div>
    <button @click="countAdd">{{ n }}</button>
  </div>
</template>
<script setup>
	let obj = reactive({
        count: 1
      })
	let n = obj.count;
	function countAdd() {
        n++
      }
</script>
```

### 用`ref()`定义相应式变量

vue3提供ref()方法来允许我们创建可以使用任何值类型的响应式ref。但是有一点需要特别注意，用ref包装的响应式，调取时需要加上.value，否则调取不到，如下所示

```js
	  let count = ref(0);
      console.log(count); // RefImpl对象，里面有value
      console.log(count.value); // 0
```

当ref包装的值为对象类型值时会用reactive自动转换它的.value，并且ref被传递给函数或是从一般对象上被解构时，不会丢失响应性

### ref在模板中的解包

ref在模板中使用时不用加.value，会自动完成解包，如下所示

```vue
<template>
  <div>
    <button @click="countAdd">{{ count }}</button>
  </div>
</template>
<script setup>
	let count = ref(0);
	function countAdd() {
        count.value++
      }
</script>
```

## 计算属性

在模板语法中，可以写一些简单的表达式，并且是非常方便的。但是如果出现太多逻辑，会使得代码非常臃肿，所以vue给我们提供了更好的解决方案——计算属性，请看下述示例

```js
<template>
    <div>
        <span :style="{color: textColor}">你好啊</span>
        <button @click="handleArr">改变颜色</button>
    </div>
</template>

<script setup>
import { computed, reactive } from 'vue';

    let arr = reactive(['red', 'blue', 'green']);

    function handleArr() {
        if(arr.length){
            arr.pop();
        }
    }

    let textColor = computed(() => {
        return arr.length ? 'green': 'red';
    })
</script>
```

创建一个数组，再创建一个方法使方法每次触发都删除数组最后一个元素。将此方法绑定到一个按钮上。在创建一个计算属性，在arr数组长度为0时改变textColor的值，并且把textColor的值当作span中文字的color。当点击三次后，span中的字颜色变了，说明计算属性可以监听值的变化。

### 计算属性缓存vs方法

计算属性是会被缓存的，如果计算属性所监听的值没有发生变化，则无论重新渲染多少次页面，里面的值都不会被重新计算，而是使用先前存储好的值。这样其实可以节省掉很多性能，并且计算属性只会基于响应式依赖缓存，如下代码

```js
let time = computed(() => Date.now())
```

由于`Date.now()`不是响应式的，所以不会更改

### 最佳实践

计算属性默认只是只读的，可以改写成可以被修改，但是没必要。因为计算属性只是基于原始响应式的一个快照，直接修改快照是没有任何意义。所以计算属性应该永远不被更改，想要改变就应该更新他所依赖的原装态以触发新的计算。

## 类与样式绑定

### 绑定类

- 使用v-bind:class="{ active: isActive }"，意思是active类存在取决于isActive的值是否为真
- 使用v-bind:class="[类名, 类名]"，可以绑定多个类名，还可以在数组内使用三元表达式
- 还可以使用上述方法调用子组件时给子组件绑定类名也是可以实现的

### 绑定内联样式

- 使用v-bind:style="{css属性}"，可以直接添加样式，也可以在js部分定义好对象直接调用
- v-bind:style同样可以绑定数组，添加包含多个样式的对象

由于这一部分和vue2中基本没有区别，不做太多赘述

