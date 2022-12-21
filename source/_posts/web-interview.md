---
title: 前端高频面试题总结
date: 2022-12-21 9:53
categories: 面试
---

## 1、vue双向绑定原理（响应式原理）

核心思想就是Object.defineProperty()进行数据劫持。在vue3中改用Proxy(代理)来进行数据劫持。先来看一段代码。

```js
<div id="demo"></div>
    <input type="text" id="inp">
    <script>
        let obj = {};
        let demo = document.getElementById('demo');
        let inp = document.getElementById('inp');
        Object.defineProperty(obj, 'name', {
            get:function() {
                return val;
            },
            set:function(newVal) {
                // newVal就是此时最新的obj.name
                inp.value = newVal;
                demo.innerHTML = newVal;
            }
        })
        inp.addEventListener('input', function(e) {
            // 当输入框中值变化时，就会改变obj.name
            obj.name = e.target.value
        })
    </script>
```

这段代码简单完成了一个双向绑定，看这个结构大家很眼熟，就是MVVM，也就是模型-视图-视图模型结构。

## 2、vue生命周期有哪些

- beforeCreate(创建前)
- created(创建后)
- beforeMount(载入前)
- mounted(载入后)
- beforeUpdate(更新前)
- updated(更新后)
- beforeDestroy(销毁前)
- destroyed(销毁后)

## 3、v-if和v-show的相同点和不同点

- 相同点：两者都能控制DOM元素的显示与隐藏。
- 不同点：
  - v-show：只是给DOM元素加上display: none;属性，DOM元素还在页面中，并未被删除。所以切换不需要重新渲染页面。
  - v-if：直接将DOM元素从页面中删除，再次切换需要重新渲染页面。

## 4、async await是什么？有什么作用

async和await是ES7新增的属性。async用于声明一个函数，await用于拦截一个异步请求，等次异步执行完并返回一个结果后，再去执行后面的函数体。

ps：await只能在async声明的函数中使用。

## 5、数组常用的方法？改变原数组或者不改变原数组的方法

- 改变原数组的方法
  - push(在数组末尾添加一个或多个元素并返回新数组)
  - unshift(在数组的开头添加一个或多个元素并返回新数组)
  - pop(删除数组的最后一个元素并返回新数组)
  - shift(删除数组的第一个元素并返回新数组)
  - reverse(反转数组的元素顺序)
  - sort(对数组的元素进行排序)
  - splice(用于插入、删除或替换数组的元素)
- 不改变原数组的方法
  - concat(合并两个或多个数组)
  - every(检测数组每个元素是否符合检索条件)
  - some(检测数组是否存在元素符合检索条件)
  - filter(检测数组元素并返回所有符合条件的元素的数组)
  - indexOf(搜索数组中的元素并返回它所在的位置)
  - join(将所有数组元素拼接成一个字符串)
  - toString(把数组转换为字符串并返回)
  - lastIndexOf(返回一个指定的字符串最后出现的位置)
  - map(通过指定函数处理数组的每个元素并返回处理后的数组)
  - slice(截取数组的一部分并返回新数组)
  - valueOf(返回数组对象的原始值)