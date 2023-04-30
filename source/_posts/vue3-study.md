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



