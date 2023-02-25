---
title: 前端高频面试题总结
date: 2022-12-21 9:53
categories: 面试
---

## 1、vue双向绑定原理（响应式原理）

核心思想就是Object.defineProperty()进行数据劫持。在vue3中改用Proxy(代理)来进行数据劫持。先来看一段代码。

- vue2

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
<!-- more -->
- vue3

  ```js
  let obj = {
              name:'yusen'
          };
          const p = new Proxy(obj,{
              get(target,propName) {
                  console.log('get');
                  return Reflect.get(target,propName)
              },
              set(target,propName,value) {
                  console.log('set');
                  Reflect.set(target,propName,value);
              },
              deleteProperty(target,propName){
                  console.log('delete');
                  return Reflect.deleteProperty(target, propName)
              }
          })
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

![](http://106.55.171.176:9000/yusen/Snipaste_2022-12-22_11-57-29.png)

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

  防抖和节流都是降低程序执行频率的方法，防抖是指**事件函数在一定时间连续触发只执行最后一次**，节流是指**同一单位时间内函数只执行一次**

  下面是代码实现：

```js
       window.onload = function() {
            const debounceBtn = document.getElementById('debounce');
            const throttleBtn = document.getElementById('throttle');

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
            function Throttle(fn,delay) {
                let timer = null;
                function throttle(){
                    if(!timer){
                        timer = setTimeout(()=>{
                            fn();
                            timer = null;
                        }, delay);
                    }
                }
                return thorttle;
            }
            throttleBtn.onclick = Throttle(() => {console.log('throttle');},3000);
        }
```

## 8、ES6新特性

- 新建模板字符串（\`${}`）
- 箭头函数
- for-of（可以用来遍历一个数组中的值）
- ES6将Promise对象纳入规范，提供了原生的Promise对象
- 新增let和const来声明变量
- 还有如module模块化概念

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
- udp效率比tcp高 容易丢数据

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

## 31、清除浮动的方法

#### 一共五种方法

- 父盒子设置高度
- overflow: hidden;
- 伪元素
- 双伪元素
- 在父盒子末尾添加一个空盒子，设置 clear: both;

## 32、split() 和 join() 的区别？

- split 字符串转换为数组，参数为以某个字符串分割

  ```js
  let str = '123-456';
  console.log(str.split('-')); // 输出：['123', '456']
  ```

- join 数组转换为字符串，参数表示转换为字符串以什么连接

  ```js
  let arr = ['123','456'];
  console.log(arr.join('-')); // 输出：123-456
  ```

## 33、数组去重

- 利用双for循环，配合数组方法splice方法去重(es5及之前常用)

- set去重: 准备一个数组，数组解构new set，再准备一个函数存放数组变量作为函数判断值return

  ```js
  let arr = [1,3,2,3,4,5,5,1,2,3];
  let test = new Set(arr);
  console.log(Array.from(test)); // 输出：[1, 3, 2, 4, 5]
  ```

- 数组方法indexOf

- 数组方法 sort Obj[a] - Obj[b]

## 34、什么原因会造成内存泄漏

- 全局变量使用不当(没有声明的变量)
- 闭包使用不当
- 定时器/延时器没有清理
- 没有清理的DOM元素引用(dom清空或删除时，事件未清除)

## 35、Vue中第一次加载页面会触发哪几个钩子函数

- beforeCreate
- created 数据初始化完成，方法页面调用，但是DOM未渲染
- beforeMount
- mounted DOM和数据挂载完成

## 36、get和post区别

#### 相同点

get请求和post请求底层都是基于TCP/IP协议实现的，使用二者中的任意一个，都可以实现客户端和服务端的双向交互

#### 最本质的区别

- 约定和规范：
- 规范：定义GET请求是用来获取资源的，也就是进行**查询**操作的，POST请求是用来传输实体对象的，用于**增删改**操作
- 约定：GET请求将参数拼接到URL上进行参数传递 POST请求将参数写入请求正文中传递

#### 非本质不同

- 缓存不同，get会缓存
- 参数长度限制不同，get请求的参数是通过URL传递的，而URL的长度是有限制的，通常为2K；post请求参数存放在请求正文中，没有大小限制
- 回退和刷新不同，get请求可以直接回退和刷新，不会对用户和程序产生影响；post请求如果直接回滚和刷新，数据将会再次提交
- 历史记录不同，get请求的参数会保存在历史记录中，post请求的参数不会
- 书签不同，get请求的地址可以被收藏为书签，post不会

## 37、跨域

- 跨域原因：浏览器出于安全考虑保护资源，同源策略。（协议、域名、端口号）
- 解决跨域：
- jsonP 但只能使用get原理将请求的接口设置给script标签的src属性传递一个参数给后台实现跨域。后台响应的是一个函数调用。
- cors：最常用。
- 反向代理：本地前端发送到本地后端，不会跨域，（同源）本地后端接受请求后会转发到其他服务器（服务器和服务器之间不会跨域）代理是需要路径中的特殊标志。

## 38、三种存储的区别

- cookie 设置过期时间删除，即使窗口或浏览器关闭
- localStorage(本地存储) 存储量大，存储持久数据，浏览器关闭后数据不会丢失除非手动删除
- sessionStorage(会话存储) 临时存储，关闭浏览器时存储内容自动删除

#### 存储大小：

- cookie 数据大小不能超过4K
- sessionStorage和localStorage虽然也有存储大小的限制，但比cookie大得多，可以达到5m或更大

## 39、dom如何实现浏览器内多个标签页之间的通信

- webSokcet，SharedWoeket
- 也可以调用localStorage、cookies等本地存储方式；localStorage另一个浏览器上下文里被添加、修改或删除时，他都会触发一个事件，我们通过监听事件，控制他的值进行页面信息通信；
- 注意quirks：Safari在无痕模式下设置localStorage值时会抛出，quotaExceededError的异常

## 40、vue.cli项目src目录每个文件夹和文件的用法

- assets文件夹是放静态资源
- components是放组件
- router是定义路由相关的配置
- view是视图
- app.vue是应用主组件
- main.js是入口文件

## 41、$route和$router的区别

- router为VueRouter的实例，相当于一个全局的路由器对象，里面含有很多属性和子对象，例如history对象等。经常用的跳转链接就可以使用this.$router.push，和router-link跳转一样
- route相当于当前正在跳转的路由对象，可以从里面获取name，push，params，query等

## 42、虚拟DOM实现原理

- 用javaScript对象模拟真实DOM树，对真实DOM进行抽象
- diff算法：比较两棵虚拟树的差异
- pach算法：将两个虚拟DOM对象的差异应用到真实的DOM树

## 43、普通函数和箭头函数的区别

- 箭头函数没有原型，原型是undefined
- 箭头函数this指向全局对象，而函数指向引用对象
- call，apply，bind方法改变不了箭头函数的this指向

## 44、怎样理解vue单项数据流

数据总是从父组件到子组件，子组件没有权利修改父组件传过来的数据，只能请求父组件对原数据进行修改

## 45、slot插槽

- slot插槽，可以理解为slot在组件模板中提前占据了位置，当复用组件时，使用相关的slot标签时，标签里的内容就会自动替换组件模板中对应slot标签的位置，作为承载分发内容的出口
- 主要作用是：复用和扩展组件，做一些定制化组件的处理

## 46、vue常见指令

- v-model 多用于表单元素实现双向数据绑定
- v-bind：简写为：，动态绑定一些元素属性，类型可以是：字符串、对象或数组
- v-on:click 给标签绑定函数，可以缩写为@click，例如绑定一个点击函数，函数必须写在methods里面
- v-for 格式：v-for="字段名 in(of) 数组/json"  (要记得绑定key值)  
- v-show 显示内容
- v-if：取值为true/false，控制元素是否被渲染
- v-else：和v-if指令搭配使用，没有对应的值。当v-if的值为false，v-else才会被渲染出来
- v-else-if 必须和v-if连用
- v-text 解析文本
- v-html 解析html标签
- v-bind:class 三种绑定方法
  - 对象型 '{red: isred}'
  - 三元型 'isred' ? 'red' : 'blue'
  - 数组型 '[{red: "isred"}, {blue: "isblue"}]'
- v-once 进入页面时 只渲染一次 不再进行渲染
- v-cloak 防止闪烁
- v-pre 把标签内部的元素原位输出

## 47、vue中keep-alive的作用 

- \<keep-alive> 是Vue的内置组件，能在组件切换过程中将状态保留在内存中，防止重复渲染DOM
- \<keep-alive> 包裹动态组件时，会缓存不活动的组件实例，而不是销毁他们
