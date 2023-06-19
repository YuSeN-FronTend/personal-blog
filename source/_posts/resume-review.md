---
title: 重要知识点复习
date: 2023-6-16 9:55
categories: 面试
---
# vue源码部分

## 模板编译

大体的结构分为三个部分：

- 先将模板字符串转换为抽象语法树(解析器)

  这里需要创建一个parse函数来完成对模板字符串的解析，通过正则表达式来抓取标签的开始和结束，将模板字符串转换为一个js对象，方便后续操作。

- 对抽象语法树进行静态节点标记，主要用来做虚拟DOM的优化(优化器)

  对抽象语法树添加属性，一定要添加key，因为再之后的diff算法比较时，可以更加快速且清晰的找到相同的节点，且判断是否需要改变

- 使用抽象语法树生成render函数代码字符串(代码生成器)

  将标记过后的语法树按照规则进行拼接即可

## 响应式原理

在响应式原理中，`Observe`、`Dep`、`Watcher`这三类是构成完整原理的主要成员。

- Observe

  响应式原理的入口，根据数据类型处理观测逻辑。(如果是基本数据类型，即可直接处理，如果是数组或者对象，则可能需要遍历递归处理)

- Dep

  依赖收集器，属性都会有一个Dep，方便发生变化时能够找到对应的依赖触发更新

- Watchar

  用于执行更新渲染，组件会拥有一个渲染Watcher，上面Dep是依赖收集器，收集的就是Watcher

### Observe包含的功能

- 为观测的属性添加`__ob__`，它的值相当于this，即当前`Observe`实例
- 为数组添加重写数组方法，比如: `push`、`unshift`、`splice`等方法，重写目的是在调用这些方法时，进行更新渲染。
- 观测数组内的数据，`observe`内部会调用`new Observe`形成递归观测
- 观测对象数据，Object.defineProperty为数据定义getter或者setter，即数据劫持

### Dep包含的功能

- 数据收集依赖的主要方法，Dep.target是一个watcher实例

- 添加watcher到数组中，也就是添加依赖
- 属性在变化时会调用notify方法，通知每一个依赖进行更新
- Dep.target用来记录watcher实例，是全局唯一的，主要目的是为了在收集依赖的过程中找到对应的watcher

### Watcher包含的功能

- watcher存储dep，dep也存储watcher，进行双向记录
- 触发更新，每次数据发生更改都会通知watcher来进行数据更新

### 大体流程

数据在初始化的时候会通过`observe`来调用`Observe`，初始化时observe拿到的参数data就是我们在data参数内返回的对象。observe只对object类型数据进行观测，观测过的数据都会被添加上`__ob__`属性，通过判断该属性是否存在，防止重复观测。创建Observe实例，开始处理观测逻辑。进入到Observe内部，由于初始化的是一个对象，则在defineReactive中使用Object.defineProperty，这里defineReactive只是一个函数的名字，不是API。如果是对象，则需要调用observe进行递归观测，这里的dep就是上面讲到的每一个属性都会有一个dep，它作为闭包存在，负责收集依赖和更新。在依赖收集时，会先对name属性定义get和set，然后初始化会执行一次watcher.run执行页面，获取data.name，触发get收集依赖，当修改data.name，触发set函数，调用run更新视图。所以在vue3中使用Proxy代替Object.defineProperty也就不足为奇了，因为属性是对象类型的话，递归观测是很消耗性能的，而proxy代理的是整个对象，只要属性发生变化就会触发回调。

## 虚拟DOM和diff算法

虚拟DOM其实就是模板编译，将DOM结构的字符串转换为一个vnode对象来模拟DOM节点。

### diff算法

将虚拟DOM转化成DOM就需要经过diff算法来检测是否是同一节点，如果是则不需要重新生成，这样就会提升页面性能，当项目庞大起来，只更改一个text造成页面重新渲染，这对性能消耗无法想象。所以diff算法的思想和实施是非常有必要的，具体分为几下几点情况。

- 如果oldVnode和newVnode不是同一个节点(同一个sel)，删除旧的，插入新的。

- 如果是同一个节点，还需要分为以下情况

  - 如果是同一个对象没什么都不做

  - 如果newVnode和oldVnode中都有text属性且相等，什么都不做。如果不相等就用新节点text来替换老节点的text

  - 如果老节点没有children而新节点有，则删除老节点的text并添加新节点的children

  - 如果老节点有children，新节点也有，则才真正的用到了diff算法。

    diff算法分为四种情况，设立四个指针，新前、新后、旧前、旧后，按照以下的规则比较移动指针或插入节点。

    - 新前和旧前

      如果相等，新前和旧前同时下移

    - 新后和旧后

      如果相等，新后和旧后同时上移

    - 新后和旧前

      如果相等，新后指向的节点插入到旧后之后，旧前所指向的对应节点变为undefined，新后上移，旧前下移

    - 新前和旧后

      如果相等，新前指向的节点插入到新前之前，旧后所指向的对应节点变为undefined，新前下移，

    如果是旧节点先循环完毕，则新节点剩余的就是需要添加的节点，如果新节点先循环完毕，则老节点剩余的就是需要删除的节点。

    按照如上顺序依此比较，如果都不满足，则需要循环去比较newVnode和oldVnode的剩余节点，但出现的概率不大。

# JS设计模式部分

## 单例模式

单例模式是指在一个类只能有一个实例，即便多次实例化该类，也只返回第一次实例化后的实例对象。

单例模式不仅能减少不必要的内存开销，并且在减少全局的函数和变量冲突也具有重要的意义。

### 简单的单例模式

最简单的命名方式就是对象字面量命名

```js
let timeTool = {
    name: '单例模式',
    getISODate: function() {
        console.log(this.name);
    },
    getUTCDate: function() {}
}
```

这里的timeTool就已经是一个实例了，并且ES6中的let和const不允许重复声明的特性，确保了timeTool不能被重新覆盖。

但如此声明的单例模式，所有的属性和方法都暴露在外面，很容易造成命名空间的污染，所以说并不是特别的完善。

将上面代码做简单的优化，控制它暴露的属性：

```js
let mySingleton = function () {
    let privatename = '单例'
    function showPrivate() {
        console.log(privatename);
    }
    return {
        publicMethod: function() {
            showPrivate();
        },
        publicVar: '公有变量'
    }
}

let single = mySingleton();

single.publicMethod()
console.log(single.publicVar);
```

调整过后，代码相对合理许多。但是单例模式是要保证一个类只有一个实例，实现的方法一般是先判断实例存在与否，如果存在直接返回，如果不存在就创建了再返回，来确保一个类只有一个对象。所以上面的代码还不是特别完善，还应该做一些修改：

```js
function Singleton(name) {
    this.name = name;
    this.instance = null;
}

Singleton.prototype.consoleName = function() {
    console.log(this.name);
}

Singleton.getInstance = function(name) {
    // 如果没有被实例化就创建这个类的实例化
    if(!this.instance) {
        this.instance = new Singleton(name);
    }
    // 已经实例化过了就直接返回之前实例化对象的引用
    return this.instance;
}

let a = Singleton.getInstance('单例模式')
let b = Singleton.getInstance('修改单例')

a.consoleName() // 单例模式
b.consoleName() // 单例模式
```

这就实现了单例模式。

以上是ES5的写法，有了ES6后，可以写出更加优雅的代码：

```js
class Singleton {
    constructor(name) {
        this.name = name;
        this.instance = null;
    }
    // 创建一个静态方法
    static getInstance(name) {
        if(!this.instance) {
            this.instance = new Singleton(name);
        }
        return this.instance;
    }
    consoleName() {
        console.log(this.name);
    }
}
let a = Singleton.getInstance('单例模式');
let b = Singleton.getInstance('修改单例');

a.consoleName()
b.consoleName()
```

总而言之，单例模式的核心是确保只有一个实例，并且可以提供全局访问。就是我们只需要一个浏览器对象window，使用jQuery的$对象不需要第二个。

单例模式的优点：

- 单例模式在创建后只存在一个实例，节约了内存开支和实例化的性能开支，可以节约更多的资源。
- 单例模式可以解决对资源的多重占用，比如写文件操作时，因为只有一个实例，可以避免对一个文件进行同时操作。
- 只是用一个实例，可以减少垃圾回收机制的压力，表现在浏览器中就是系统卡顿减少，操作更流畅。

单例模式的缺点：

- 单例模式一般不容易扩展，因为单例模式一般自行实例化，没有接口。
- 与单一职责原则冲突，一个类应该只关心内部逻辑，不关心外面如何实例化引用。

## 策略模式

定义一系列的算法，将他们一个个封装起来，是他们直接可以相互替换。

正常书写代码：

```js
        let level = ''
        const btn1 = document.getElementById('btn1');
        const btn2 = document.getElementById('btn2');
        const btn3 = document.getElementById('btn3');
        const btn4 = document.getElementById('btn4');
        const text = document.getElementById('text');
        function levFun () {
            if (level === 'A') {
                text.innerText = 'jfz'
            } else if (level === 'B') {
                text.innerText = 'zj'
            } else if (level === 'C') {
                text.innerText = 'zpz'
            } else if (level === 'D') {
                text.innerText = 'ys'
            }
        }
        btn1.onclick = function() {
            level = 'A'
            levFun()
        }
        btn2.onclick = function() {
            level = 'B'
            levFun()
        }
        btn3.onclick = function() {
            level = 'C'
            levFun()
        }
        btn4.onclick = function() {
            level = 'D'
            levFun()
        }
```

用策略模式来修改levFun函数中的if else语句

```js
        let obj = {
            'A': function() {
                text.innerText = 'jfz'
            },
            'B': function () {
                text.innerText = 'zj'
            },
            'C': function () {
                text.innerText = 'zpz'
            },
            'D': function () {
                text.innerText = 'ys'
            },
        }
        function levFun () {
            return obj[level]()
        }
```

这样一来，代码显得清晰简介，代码的复用和弹性也会随之变强。

当时在写可视化大屏项目的时候，在高德地图上添加点位，并且捕获点击不同点位都要弹出不一样的模态框，当时并没有了解过js设计模式，所以就用if else语句强行实现功能。之后学习到策略模式时，就萌生重写的想法，改过之后的代码，非常的清晰明了，再配合上模态框复用，使相同echarts图形的模态框都使用同一个，这样一来，就减少了不少的项目体积。现在除了比较少量的if else，其余我已经都用策略模式代替了，越多的判断语句，策略模式的好处越能体现，使代码更易于i切换和扩展。

## 发布订阅模式

它定义了一种一对多的关系，多个订阅者同时监听某一个主题对象，当主题对象发生变化时，它会通知所有订阅者对象，使它们能够自动更新。

### 优点

- 实现了发布者和订阅者之间的解耦，提高了代码的可维护性和复用性
- 支持异步处理，可以支持事件的延迟触发和批量处理
- 支持多对多通信，可以实现广播和组播的功能

### 缺点

- 可能会造成内存泄漏，如果订阅者对象没有及时的取消订阅，就会一直存在于内存中
- 可能会导致程序的复杂性增加，如果订阅者对象过多或者依赖关系不清晰，就会增加程序的调试难度
- 可能会导致信息的不一致性，如果发布者在通知订阅者之前或之后发生了变化，就会造成数据的不同步

### 举例

设置一个空数组，可以向里面添加订阅者，然后最后发布时遍历通知到每一个订阅者

```js
let fnList = [];

class Journal{
    subscribe(fn){
        const index = fnList.indexOf(fn);
        if(index !== -1) return fnList;
        fnList.push(fn);
        return fnList;
    }
    unsubscribe(fn){
        const index = fnList.indexOf(fn);
        if(index === -1) return fnList;
        fnList.splice(index, 1);
        return fnList;
    }
    notify(){
        fnList.forEach((item) => {
            item.update();
        })
    }
}

const o = new Journal();

class Observer {
    constructor(name, age) {
        this.name = name;
        this.age = age
    }
    update(){
        return console.log(`${this.name}喜欢${this.age}`);
    }
}

let obj1 = new Observer('小张', '打篮球')
let obj2 = new Observer('小鞠', '游泳');

o.subscribe(obj1);
o.subscribe(obj2);

o.unsubscribe(obj1);

o.notify(); // 小鞠喜欢游泳
```

# mustache模板引擎

它的实现原理就是把模板字符串编译成tokens形式(嵌套数组的形式，也就是模板字符串的JS表示，里面有规定的属性)，然后再和数据进行组合拼接，完成后生成最终的DOM字符串，再显示到页面上。

mustache使模板引擎的开山鼻祖，再vue之前很久就存在mustache，所以vue自然也是借鉴了mustache中的一些思想。在还没有存在mustache的时候，将数据变为视图起初是适用纯DOM方法，它是非常笨拙地，就是适用JS语句直接操作DOM。数组join法，顾名思义可以将数组转换为字符串，再用于拼接，这个在当时是非常流行的。ES6反引号法时候来新增的，可以更便捷的拼接字符串和数据。最优雅的当然还是模板引擎。具体的手写代码[点击这里](https://yusen.club/2023/04/07/mustache-study/)方可查看

# Ajax的原理以及实现

Ajax全称为Async JavaScript adn XML，也就是异步JS和XML。

它的原理是**通过XMLHttpRequest对象来向服务器发送异步请求，从服务器获取数据，然后用js来操作DOM更新页面。**

下面进行简单的实现：

- 创建一个XHR构造函数

  ```JS
  const xhr = new XMLHttpRequest();
  ```

- 调用xhr.open()方法简历与服务端的连接，它有5个参数，最重要的两个分别是method(请求的方法)和url(请求的地址)

  ```js
  xhr.open('GET', 'http://127.0.0.1:7001/index');
  ```

- 调用xhr.send()方法传递参数body，如果是get请求没有参数body，传入null即可

  ```js
  xhr.send(null);
  ```

- 调用xhr.onreadystatechange监听事件，判断readyState的值，它的值有如下可能：

  - 0：open()未调用
  - 1：open()未调用
  - 2：send()已经调用，响应头和响应状态已返回
  - 3：响应体正在下载，responseText(接收的服务器响应的结果)获取到部分的数据
  - 4：整个请求过程已经完毕

  只要readyState属性值发生了改变，onreadystatechange事件就会触发，详情获取见如下代码

  ```js
          xhr.onreadystatechange = function() {
              if(xhr.readyState === 4) {
                  if(xhr.status >= 200 && xhr.status < 300) {
                      console.log(xhr.responseText);
                  } else if(xhr.status >= 400) {
                      console.log('错误信息' + xhr.status);
                  }
              }
          }
  ```

- 全部完成后，自行处理数据渲染DOM即可

# Node.js

node.js是一个跨平台的js运行环境，使开发者可以搭建服务器端的js应用程序。node.js运行在V8js引擎上(chrome)，并在web浏览器之外执行js代码。

**优点：**

- 高并发(这个是它最重要的优点)
- 适合I/O密集型应用

**缺点：**

- 不适合CPU密集型应用；因为由于js单线程的原因，如果有长时间运行的计算，将会导致CPU时间片不能释放，导致后续I/O无法发起

  解决：分解大型运算任务为多个小任务，使得计算能够适时释放，不阻塞I/O调用的发起

- 支支持单核CPU，不能充分利用CPU

- 可靠性比较低，一旦代码某个环节崩溃，整个系统都崩溃，因为js是单进程，单线程

  解决：Nnigx反向代理，负载均衡，开多个进程，绑定多个端口

- 开源组件库只能参差不齐，更新快，不兼容(这个在我们写项目的时候也经常遇到)

- Debug不方便

node.js几乎可以实现我们想要实现的任何应用，但要从多方面考虑它是否真的适合。

# ES6起个版本的新特性

## ES6(ES2015)

- class
- 模块化：导入export、导出import
- 箭头函数：this指向定义时所在的对象，而不是使用时所值的对象
- 函数参数默认值：function app(age=25) {...}
- 模板字符串
- 解构赋值：let [a,b] = [2,3] // a:2 b:3
- 扩展运算符：...
- 对象属性简写：{name:name} ===> {name}
- promise对象
- let, const

## ES7(ES2016)

- 新增数组的includes属性，检测数组中是否存在与传入参数一样的值
- 引入了(**)指数操作符

## ES8(ES2017)

- 新增async await使得异步改同步称为可能，避免代码书写的来回嵌套
- 新增`Object.values()`获取对象中所有的value值
- 新增`Object.entries()`将键值对包在一个数组里面再全部放在一个大数组中
- 新增字符串填充(padStart，padEnd)
- 允许函数参数列表结尾存在逗号
- 添加Object.getOwnPropertyDescriptors():获取一个对象的所有自身属性的描述符，如果没有任何自身属性，则返回空对象
- 新增ShareArrayBuffer对象：用来表示一个通用的，固定长度的原始二进制数据缓冲区
- 新增Atomics对象：提供了一组静态方法用来对SharedArrayBuffer对象进行原子操作

## ES9(ES2018)

- 允许异步迭代：await可以和for...of循环一起使用，已串行的方式运行异步操作
- 新增Promise.finally()
- 修改了正则表达式的一些属性

## ES10(ES2019)

- 新增数组flat()和flatMap()方法，用于摊平数组
- 修改了try catch的使用
- 增加字符串的trimStart， trimEnd方法，分别去掉首尾空格
- 增加Object.fromEntries方法，可以把对应数组转成对象

