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

## 6、什么是原型链

先说一下原型和实例

```js
class Person{}
const person = new Person();
```

上面代码中，Person类就是person的原型对象，而person就是Person类的实例。

每一个构造函数的实例对象上有一个proto属性，指向的构造函数的原型对象，构造函数的原型对象也是一个对象，也有proto属性，这样一层一层向上寻找的过程就被称为原型链。

![](http://106.55.171.176:9000/yusen/Snipaste_2022-12-22_11-57-29.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=sttch%2F20221222%2F%2Fs3%2Faws4_request&X-Amz-Date=20221222T035914Z&X-Amz-Expires=432000&X-Amz-SignedHeaders=host&X-Amz-Signature=8469159f68c62d0140d2ca7853a5471e126fcbdf08bec4dcdd1642ac9dfd9b99)

## 7、什么是闭包，闭包有哪些优缺点

- 概念：函数嵌套函数，内部变量能访问外部变量，这个变量称为自由变量

  ```js
  function a() {
              let demo = 123;
              function b(){
                  console.log(demo); // 123
              }
              b();
          }
          a();
          console.log(demo); // 报错
  ```

- 解决的问题：保存变量

- 带来的问题：容易造成内存泄漏

- 闭包的应用：防抖节流

  防抖和节流都是降低程序执行频率的方法，防抖是指**事件函数在一定时间连续触发只执行最后一次**，节流是指**同一单位直接内函数指执行一次**

  下面是代码实现：

```js
       window.onload = function() {
            const debounceBtn = document.getElementById('debounce');
            const thorttleBtn = document.getElementById('thorttle');

            // 防抖
            function Debounce(fn, delay) {
                let timer = null;
                const debounce = function() {
                    // 形成闭包，timer变量不被释放，如果timer不为空则清除延时执行
                    if(timer){
                        clearTimeout(timer);
                    }
                    timer = setTimeout(()=>{
                        fn()
                    },delay);
                }
                return debounce
            }
            debounceBtn.onclick = Debounce(() => { console.log('Debounce')},3000)

            // 节流
            function Thorttle(fn,delay) {
                let timer = null;
                function thorttle(){
                    if(!timer){
                        timer = setTimeout(()=>{
                            fn();
                            timer = null;
                        }, delay);
                    }
                }
                return thorttle;
            }
            thorttleBtn.onclick = Thorttle(() => {console.log('thorttle');},3000);
        }
```

## 8、ES6新特性

- 新建模板字符串（\`${}`）
- 箭头函数
- for-of（可以用来遍历一个数组中的值）
- ES6将Promise对象纳入规范，提供了原生的Promise对象
- 新增let和const来声明变量
- 还有人如module模块化概念

## 9、v-for循环为什么一定绑定key

给每个遍历出来的dom元素加上key作为唯一标识，diff算法可以正确识别这个节点，是页面渲染更迅速。

## 10、组件中data为什么定义成一个函数而不是对象

每个组件都是Vue实例。组件共享data属性，当data的值是同一个引用类型的值是，改变其中一个会影响到其他。