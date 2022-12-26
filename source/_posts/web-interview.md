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

  防抖和节流都是降低程序执行频率的方法，防抖是指**事件函数在一定时间连续触发只执行最后一次**，节流是指**同一单位时间内函数指执行一次**

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

每个组件都是Vue实例。组件共享data属性，当data的值是同一个引用类型的值是，改变其中一个会影响到其他。从而为了保证数据的独立性，才选择定义成一个函数。

## 11、常见的盒子居中的方法

- 子绝父相方法

  ```html
  <style>
      .container{
          width: 300px;
          height: 300px;
          position: relative;
          background-color: yellowgreen;
      }
      .center{
          width: 100px;
          height: 100px;
          background-color: aqua;
          position: absolute;
          top: 50%;
          left: 50%;
          margin-top: -50px;
          margin-left:-50px;
      }
  </style>
  ```

- transform 可以在未知元素宽高情况下实现居中。

  ```html
     <style>
          .container{
              position: relative;
          }
          .conter{
              position: absolute;
              top: 50%;
              left: 50%;
              transform: translate(-50%,-50%);
          }
      </style>
  ```

- 弹性盒子

  ```html
   <style>
          .container{
           display: flex;
           justify-content: center;
           align-items: center;
          }
          .conter{
              
          }
      </style>
  ```

## 12、js数据类型有哪些，区别是什么

- 基本类型：string，number，boolean，null，undefined，symbol，bigInt
- 引用类型：object，array
- 基本类型存储在栈中，空间小，操作频繁
- 引用类型存储在堆中，他的地址在栈中，一般我们访问的就是他的地址

## 13、什么是symbol

是es6引入的新的原始数据类型symbol，表示独一无二的值。

## 14、什么是同源策略

所谓同源策略就是浏览器的一种安全机制，来限制不同源的网站不能通信（域名、协议、端口号相同）

## 15、promise是什么，有什么作用

- promise是一个对象，可以从改变对象获取异步操作信息
- 它可以解决回调地狱问题，也就是异步深层嵌套问题

## 16、什么是递归，递归有哪些优缺点

- 递归：如果函数在内部可以调用其本身，那么整个函数就是递归函数，简单理解就是：自己调用自己的函数就是递归函数。
- 优点：结构清晰，可读性强。
- 缺点：效率低，调用栈可能溢出。

### 17、let和const有哪些特性

- let命令不存在变量提升，如果在let声明前使用，会导致报错，这也就是常说的“暂时性死区”。
- 如果块区域存在let和const就会形成封闭作用域，在块级区域外无法访问let和const声明的变量。
- const定义的是常量，不能修改，但如果定义的是对象，可以修改对象内部的数据。

## 18、vue性能优化

- 函数式组件
- 路由懒加载
- v-for绑定key值
- computed缓存数据和watch keep-alive缓存组件
- v-if和v-for不要同时使用，因为两者作用在同一元素，优先级是不同的
- 设计vue响应式数据时不能设计太深，会做全量递归的计算
- 组件的颗粒度不能设计太细，合理划分，层积越深性能消耗越大
- 防抖节流
- ui组件库按需引入

## 19、mvvm和mvc

- m（数据层）v（视图层）vm（数据视图交互层）简化了大量dom操作，只用于单页面，通过数据来显示视图层而不是节点操作。
- mvc还需要获取dom，使页面渲染性能低且加载速度慢
- 面试可以说的项目优化
- 设计vue响应式数据时不能设计太深，会做全量递归的计算
- 组件的颗粒度不能设计太细，合理划分，层积越深性能消耗越大

## 20、路由模式：hash和history

- 实现的功能：
  - 改变url且不让浏览器向服务器发请求
  - 检测url的变化
  - 截获url地址并解析出需要的信息匹配路由规则
- hash基于url传参会有体积限制，不会包括在http请求中对后端完全没有影响，改变hash不会重新加载页面；history可以在url里放参数还可以将数据存放在一个特定对象中，history模式浏览器白屏解决方法是在服务端加一个覆盖所有情况候选资源，必须要服务端在服务器上有对应的模式才能使用，如果服务器没配置，可以先使用默认的hash

## 21、常用的块与行属性内标签有哪些？有什么特征

- 块标签：div、h1~h6、ul、li、table、p、br、form
- 特征：独占一行，换行显示，可以设置宽高，可以嵌套块与行
- 行标签：span、a、img、textarea、select、option、input
- 特征：只有在行内显示，内容撑开宽、高，不可以设置宽、高（img、input、textarea等除外）

## 22、==和===的区别

- ==是非严格意义上的相等

  - 值相等就相等，会做强制类型转换

    ```js
    1 == "1" //true
    true == 1 //true
    true == true // true
    ```

- ===是严格意义上的相等，会比较两边的数据类型和值的大小

  - 值和引用地址都相等才相等

    ```js
    1==="1" //false
    true === 1 //false
    true === true // true
    ```

## 23、严格模式的限制

- 变量必须声明后再使用
- 函数的参数不能有重名属性，否则报错
- 不能使用with语句
- 禁止this指向全局对象

## 24、git常用指令

- git init 初始化仓库
- git clone 克隆代码
- git status 检查文件状态
- git add . 将文件添加到暂存区(注意空格)
- git commit -m  描述信息

## 25、tcp和udp协议

- tcp安全性更高 http协议是建立在tcp基础上的
- ucp效率比tcp高 容易丢数据

## 26、vuex的 五种状态

- mutations（修改state里面的数据，但是他只能进行同步操作，异步操作必须要放在action里面）
- state（放数据）
- action（执行异步操作）
- getter（计算属性）
- moudel（允许将单一store拆分成多个store并且同时保存在单一的状态中）

##### 传递过程

页面通过mapAction异步提交事件到action。action通过commit把对应参数同步提交到mutations，mutations会修改state中对应的值。最后通过getter把对应值跑出去，在页面的计算属性中，通过mapGetter来动态获取state中的值

## 27、什么是防抖和节流，js如何处理防抖和节流

- 首先防抖就是触发下一个事件时停止掉上一个事件
- 节流是触发当前事件需要在上一个事件结束以后
- 通过设置节流阀（定时器）

## 28、什么是重绘和回流

- 重绘：当元素内容以及布局没有发生改变，只是元素外观发生改变，就是重绘
- 回流：当一部分内容或者布局发生了改变，重新构建页面就会产生回流
- 产生回流一定造成重绘，产生重绘不一定造成回流

## 29、css优先级

!importent>行内(直接在html标签内添加样式)>id>类(class)，伪类(a:hover)，属性(input[type="email"])>标签(div)，伪元素选择器(p::before)>继承和通配符(*)

## 30、如何解决盒子塌陷

- 父盒子设置上边距
- overflow：hidden
- 子盒子脱标
- 父盒子上padding