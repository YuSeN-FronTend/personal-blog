---
title: 面经总结
date: 2023-2-20 10:53
categories: 面试
---

# 1、defer标签和async标签有什么区别？

- defer指的是外部文件和当前html页面**同时加载**，但是要在当前页面解析之后再去执行js代码
- async指的是外部文件和当前html页面**同时加载**，当js代码执行完毕之后，**直接执行。**

**需要注意的是defer和async只对外部脚本才会有效果**

代码示例

```javascript
	<script src="./test.js" defer="defer"> // defer
    </script>
    <script src="./test1.js" async="async"> // async
    </script>
    <script src="./test2.js"> // 1
    </script>
    <script src="./test3.js"> // 3
    </script>
    <script src="./test4.js"> // 4
    </script>
    // 1 async 3 4 defer
```
<!--more-->
# 2、说一下闭包及其应用

父函数中包含子函数，父函数无法获取子函数中的变量，子函数可以获取到父函数中的变量，这样就形成了闭包。

- 优点：**保存变量**
- 缺点：如果使用不规范容易造成**内存泄漏**
- 闭包的应用：**防抖和节流**

**防抖**是指单位时间事件连续执行只执行最后一次，**节流**是指单位时间内事件只会执行一次

```js
		const debounce = document.getElementById('debounce');
        const throttle = document.getElementById('throttle');
        // 防抖
        function Debounce(fn, delay) {
            let timer = null;
            const debounce = function() {
                if(timer) {
                    clearTimeout(timer);
                }
                timer = setTimeout(() => {
                    fn();
                }, delay)
            }
            return debounce;
        }
        debounce.onclick = Debounce(() => {console.log('debounce');}, 1000);
		// 节流
        function Throttle(fn, delay) {
            let timer = null;
            const throttle = function() {
                if(!timer){
                    timer = setTimeout(() => {
                        fn()
                        timer = null;
                    }, delay);
                }
            }
            return throttle;
        }
        throttle.onclick = Throttle(() => {console.log('throttle');}, 1000)
```

# 3、简述一下原型和原型链

每一个实例对象上都有一个\__proto__属性(隐式原型)，指向此实例对象原型的prototype属性(显示原型)，而原型也有\_\_proto\_\_属性，这样一层一层向上查找的过程称为原型链。每一个对象都会从原型中继承属性。                          

**注：**instanceof是用于检测构造函数的prototype属性是否出现在某个实例对象的原型链上。但是对原始数据(2,'2',true等)是没有效果的。虽然原始数据也有原型链，但是是js帮我们去设置的，并非真正意义上存在的。

### 代码演示：

```js
let s1 = '2';
let s2 = new String('2');

console.log(s1 instanceof String); // false
console.log(s2 instanceof String); // true
```

# 4、继承的几种方式并展开描述

js继承方式分为六种：

- **原型链继承**(主要继承方式) 

  - **核心**：将父类的实例作为子类的原型

  - **特点**：

    1. 非常纯粹的继承关系，实例是子类的实例也是父类的实例
    2. 父类新增原型方法或者原型属性，子类都能够访问到
    3. 简单，易于实现

  - **缺点**：

    1. 要想为子类新增属性和方法，必须要在 new Animal()这样的语句后执行，不能放到构造器中
    2. 无法实现多继承
    3. 来自原型对象的所有属性被所有实例共享(来自原型对象的引用属性也是所有实例共享的)
    4. 创建子类实例时，无法向父类构造函数传参

  - **代码示例**：

    ```js
    function SuperType() {
        this.property = true;
    }
    
    SuperType.prototype.getSuperValue = function() {
        return this.property;
    }
    
    function SubType() {
        this.subproperty = false;
    }
    
    // SuperType
    SubType.prototype = new SuperType();
    
    SubType.prototype.getSubValue = function() {
        return this.subproperty;
    }
    
    let instance = new SubType();
    console.log(instance.getSuperValue()); //true
    ```

- **构造函数继承**(盗用构造函数)(借助call或apply方法)

  为了解决原型包含引用值导致的继承问题，此种继承方法应运而生，也被称作"对象伪装"或"经典继承"。

  - **核心**：在子类构造函数中调用父类构造函数。

  - **特点**：

    1. 解决原型链继承中，子类实例共享父类引用属性的问题
    2. 创建子类实例时，可以向父类传递参数
    3. 可以实现多继承(call多个父类对象)

  - **缺点**：

    1. 实例并不是父类的实例，只是子类的实例
    2. 只能继承父类的实例属性和方法，不能继承原型属性/方法
    3. 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

  - **代码实现**：

    ```js
    function SuperType() {
        this.colors = ['red', 'blue', 'green'];
    }
    
    function SubType() {
        // 继承SuperType
        SuperType.call(this);
    }
    
    let instance1 = new SubType();
    instance1.colors.push('black');
    console.log(instance1.colors); // [ 'red', 'blue', 'green', 'black' ]
    
    let instance2 = new SubType();
    console.log(instance2.colors); // [ 'red', 'blue', 'green' ]
    ```

- **组合继承**

  综合了原型链继承和盗用构造函数继承，将两者的优点集中了起来。

  - **核心**：通过调用父类构造，继承父类的属性并保留传参的有点，然后通过父类实例作为子类原型，实现函数复用

  - **特点**：

    1. 弥补了盗用构造函数继承的缺陷，可以继承实例的属性和方法，也可以继承原型属性和方法
    2. 既是子类的实例，也是父类的实例
    3. 不存在引用属性共享问题
    4. 可传参
    5. 函数可复用

  - **缺点**：

    调用了两次父类构造函数，生成了两份实例（子类实例将子类原型上的那份屏蔽了）

  - **代码实现**：

    ```js
    function SuperType(name) {
        this.name = name;
        this.colors = ['red', 'blue', 'green'];
    }
    
    SuperType.prototype.sayName = function() {
        console.log(this.name);
    }
    
    function SubType(name, age) {
        // 继承属性
        SuperType.call(this, name);
    
        this.age = age;
    }
    
    // 继承方法
    SubType.prototype = new SuperType();
    
    SubType.prototype.sayAge = function() {
        console.log(this.age);
    }
    
    let instance1 = new SubType('Nicholas', 29);
    instance1.colors.push('black');
    console.log(instance1.colors); // "red, blue, green, black"
    instance1.sayName(); // Nicholas
    instance1.sayAge(); // 29
    
    let instance2 = new SubType('Greg', 27);
    console.log(instance2.colors); // red, blue, green
    instance2.sayName(); //Greg
    instance2.sayAge(); // 27
    ```

    

- 原型式继承

  ES5里面的Object.create方法，这个方法接收两个参数：一是用作新对象原型的对象、二是为新对象定义额外属性的对象(可选参数)

  - **代码实现**：

    ```js
    let parent = {
        name: 'jufengze',
        friends: ['zz', 'ttt', 'yd'],
        getName: function() {
            return this.name
        }
    }
    
    let parent1 = Object.create(parent);
    parent1.name = 'zouzhiheng';
    parent1.friends.push('AL');
    let parent2 = Object.create(parent);
    parent2.push = 'sitansen';
    parent2.friends.push('B+');
    
    console.log(parent.name); // jufengze
    console.log(parent.friends); // [ 'zz', 'ttt', 'yd', 'AL', 'B+' ]
    console.log(parent.name === parent.getName()); // true
    console.log(parent1.friends); // [ 'zz', 'ttt', 'yd', 'AL', 'B+' ]
    console.log(parent2.friends); // [ 'zz', 'ttt', 'yd', 'AL', 'B+' ]
    ```

    

- **寄生式继承**

  使用原型式继承可以获得一份目标对象的浅拷贝，然后利用这个浅拷贝的能力再进行增强，添加一些方法，这样的继承方式就叫作寄生式继承。

  - **代码实现**:

    ```js
    let parent = {
        name: 'parent',
        friends: ['p1', 'p2', 'p3'],
        getName: function() {
            return this.name;
        }
    }
    
    function clone(original) {
        let clone = Object.create(original);
        clone.getFriends = function() {
            return this.friends;
        }
        return clone
    }
    
    let person = clone(parent);
    console.log(person.getName()); // parent
    console.log(person.getFriends()); // [ 'p1', 'p2', 'p3' ]
    ```

    

- **寄生式组合继承**

  结合原型式继承中提及的继承方法，解决普通对象的继承问题的Object.create 方法，我们在前面这几种继承方式的优缺点基础上进行改造，得出了继承组合式的继承方式，这也是所有继承方式里面相对最优的继承方式。

  - **代码实现**

    ```js
    function Father(name) {
        this.name = name;
        this.hobby = ['唱', '跳', 'rap', '篮球', 'music'];
    }
    
    Father.prototype.getName = function() {
        console.log(this.name); 
    }
    
    function Son(name, age) {
        Father.call(this, name);
        this.age = age;
    }
    
    Son.prototype = Object.create(Father.prototype);
    
    let Y = new Son('zz', '22');
    let S = new Father('aa');
    console.log('寄生组合1', Y); // 寄生组合1 Father {name: 'zz',hobby: ['唱', '跳', 'rap', '篮球', 'music'],age: '22'}
    console.log('寄生组合2', S); // 寄生组合2  Father { name: 'aa', hobby: [ '唱', '跳', 'rap', '篮球', 'music' ] }
    ```

  - **注**： ES6中新增的extends属性底层就是基于寄生式组合继承实现的

# 5、简述一下跨域问题

跨域是指浏览器不能执行其它网站的脚本，它是由浏览器的同源策略造成的，防止恶意攻击，是浏览器对JavaScript实施的安全限制

- **cors(最常用的跨域方法)**

  cors是基于http1.1的一种跨域解决方案，全称为**Cross-Origin Resource Sharing**，也就是**跨域资源共享**。

  针对不同的请求，cors规定了三种不同的交互模式，分别是

  - **简单请求**

    1. 请求方式要满足下列三种方式之一
       - HEAD
       - GET
       - POST
    2. http请求头必须是下面几种字段
       - Accept
       - Accept-Language
       - Content-Language
       - Last-Event-ID
       - Content-Type: 只限于三种值 **application/x-www-form-urlencoded**，**multipart/formdata**，**text/plain**

    对于简单请求就是浏览器直接发出CORS，具体来时就是请求头添加Origin字段

  - **非简单请求**

    不满足上述要求的都为非简单请求

  ##### cors设置的响应头字段

  - Access-Control-Allow-Origin: 必选
  - Access-Control-Allow-Credential: 可选
  - Access-Control-Expose-Headers: 可选

- **JsonP(只能发送get请求)**

  JsonP是JSON with padding简写，实现原理是动态创建script标签，然后利用script里的src不受同源策略约束通过get请求获取数据，利用callback回调函数接收服务器响应数据

- **反向代理**

  本地前端发送一个请求给本地后端，本地后端接受请求后在传递给其他服务器

# 6、Symbol是什么

symbol是ES6中新增加的基本数据类型，用来表示独一无二的值。

```js
const s = Symbol('foo');
const y = Symbol('foo');
console.log(s === y); // false
```

**方法**：

- Symbol.for()

  使用给定的key来搜索现有的Symbol，如果有则会返回现有的Symbol，如果没有，则会在全局Symbol注册表中创建一个新的Symbol

  ```js
  const s = Symbol('foo');
  
  console.log(Symbol.for('foo')); // Symbol(foo)
  console.log(Symbol.for('too')); // Symbol(too)
  ```

- Symbol.keyfor()

  从全局的注册表中，为给定的symbol检索一个共享的Symbol key

  ```js
  const s = Symbol.for('foo');
  console.log(Symbol.keyFor(s)); // foo
  ```

# 7、类数组和数组有什么区别

- **相同点**：都具有length属性和索引元素

- **不同点**：类数组没有任何Array属性和Array方法(比如push)

- **原因**:

  最常用的类数组是**arguments**：

  ```js
  function test() {
      console.log(typeof arguments); // object
  }
  test();
  ```

  可以发现输出的类型是object，说明它有数据的一些特性，而不是数组的类型。但是Array继承于Object，Array是儿子，Object是父亲，Function是母亲。

- ### 如何将类数组转换为数组

  - 方法一：**Array.from()**

    ```js
    function test() {
        const args = Array.from(arguments);
        console.log(args);
    }
    test();
    ```

  - 方法二：拓展运算符

    ```js
    function test() {
        const args = [...arguments];
        console.log(args); // []
    }
    test();
    ```

  - 方法三：Array.prototype.slice.call

    ```js
    function test() {
        const args = Array.prototype.slice.call(arguments);
        console.log(args); // []
    }
    test();
    ```

  - 方法四：Array.prototype.splice.call

    ```js
    function test() {
        const args = Array.prototype.splice.call(arguments, 0);
        console.log(args); // []
    }
    test();
    ```

  - 方法五：Array.prototype.concat.apply

    ```js
    function test() {
        const args = Array.prototype.concat.apply([],arguments);
        console.log(args); // []
    }
    test();
    ```

  - 方法六：[].slice.call

    ```js
    function test() {
        const args = [].slice.call(arguments)
        console.log(args); // []
    }
    test();
    ```


# 8、vue响应式原理

也就是数据双向绑定。大致原理如下：

首先我们需要通过Object.defineProperty()方法把数据(data)设置为getter和setter的访问形式，这样在数据被修改时在setter方法设置监视修改页面信息，也就是每次页面被修改，都会触发对应的set方法，然后我们可以在set方法中去调用操作dom的方法。

由于js的限制，vue不能检测数组和对象的变化，但是vue也提供了解决方案，使用vue.set(Object, 'propsName', value) 或者 this.$set(Object, 'propsName',value) 即可实现。

# 9、模块化是什么，有哪些

模块化是将一个复杂程序，依据一定的规则封装成一个或多个块，并进行组合在一起。块内部的数据与实现是私有的，只是向外部暴露一些接口与外部其他模块通信。

- **commonJS**

  2009年node.js发布，采用的就是commonJS模块规范

  - 每个文件都是一个模块实例，代码运行在模块作用域，不会污染全局作用域
  - 文件内通过require对象引入指定模块，通过exports对象来向外暴露API，文件内定义的函数，变量都是私有属性，对于其他文件不可见
  - 所有的文件加载都是同步完成的，加载的顺序，按照其在代码中出现的顺序
  - 有一缺点是模块化同步加载，资源消耗和等待事件，是用于服务器编程

- **ES6 Module**

  2015年，ES6规范中，终于将模块化纳入JS标准，从此js模块化被ECMA官方所认可。ES6的模块化在CommonJS的基础上有所不同，关键字有import, export, default, as, from。

  **这两者的区别**：

  - CommonJS模块输出的是一个值的拷贝，即原来模块中的值改变不会影响已经加载的该值。
  - ES6模块输出的是值的只读引用，模块内值改变，引用也改变。
  - CommonJS模块是运行加载，加载的是整个模块，即将所有的接口全部加载进来。
  - ES6模块是编译时输出接口，可以单独加载其中的某个接口。

- **AMD/RequireJS**

  AMD是“Asynchronous Module Definition”的缩写，也就是**异步模块定义**。它采用异步方式加载模块，模块的加载不会影响后面的语句执行。等待所有依赖都加载完成后，这个包含所有模块语句的回调函数才会执行

  RequireJS是用于客户端模块管理的一个工具库，它的模块管理遵守AMD规范，它的基本思想是：通过define方法将代码定义为模块，通过require方法实现代码的模块加载

- **CMD/SeaJS**

  ​	全名是"Common Module Definition" ，该规范借鉴了Commonjs和AMD的规范，在两者基础上稍加改进。

  ​	特点：

  - define定义模块，require加载模块，exports暴露变量
  - 不同于AMD的依赖前置，CMD推崇依赖就近
  - 推崇api功能单一，一个模块干一件事

  **AMD、CMD区别**

  - AMD推崇依赖前置，是异步的
  - CMD推崇依赖就近，是同步的

# 10、css3动画如何实现

- ​	常规方法 利用**@keyframes  from to** 即可完成简单动画 见如下代码(只展示css部分)

  ```css
  div{
          width: 100px;
          height: 100px;
          background-color: red;
          animation: myFirst 5s;
      }
      @keyframes myFirst {
          from{
              background-color: red;
          }
          to{
              background-color: yellow;
          }
      }
  ```

- 进阶方法(帧动画) 利用%做分段式动画，见如下代码

  ```css
  div{
          width: 100px;
          height: 100px;
          position: relative;
          background-color: red;
          animation: myFirst 5s;
      }
       @keyframes myFirst {
          0% {
              background: red; left: 0px; top: 0px;
          }
          25% {
              background: yellow; left: 200px; top:0px;
          }
          50% {
              background: blue; left: 200px; top: 200px;
          }
          75%{
              background: green; left: 0px; top: 200px;
          }
          100%{
              background: red; left: 0px; top: 0px;
          }
      }
  ```

- 属性

  - **@keyframes**  用来规定动画

  - **animation**   所有动画属性的简写属性

  - **animation-name**   规定@keyframes动画的名称

  - **animation-duration**   规定动画完成一个周期所花费的秒或毫秒

  - **animation-timing-function**   规定动画的速度曲线，默认是"ease"

    - **ease**  动画以低速开始，然后加快，在结束前变慢
    - **linear**   动画从头到尾是相同的
    - **ease-in**   动画以低速开始
    - **ease-out**   动画以低速结束
    - **ease-in-out**   动画以低速开始和结束

  - **animation-fill-mode**   规定当动画不播放时(规定当动画完成时，或当动画有一个延迟未开始播放时)，要应用到元素的样式

  - **animation-delay**  规定动画何时开始，默认是0

  - **animation-iteration-count**   规定动画被播放的次数 默认是1

    - **infinite**   播放无限次

  - **animation-direction**   是否在下一个周期逆向的播放 默认是normal

    - **normal**   动画正常播放
    - **reverse**   动画反向播放
    - **alternate**   动画在奇数次正向播放，在偶数次反向播放
    - **alternate-reverse**  与上述正相反

  - **animation-play-state**   规定动画是否正在运行或暂停  默认是running

    - **running**   指定正在运行的动画
    - **paused**   指定暂停动画


# 11、图片懒加载实现方法

#### 原理：

- 存储图片的真实路径，把图片的真实路径绑定给一个以data开头的自定义属性data-url即可，页面中的img元素如果没有src属性，浏览器就不会发出请求去下载图片。
- 初始化img的时候，src不能是真实的图片地址（会一次性的发送请求），也不可以是空地址或者坏地址（会出现出错图标）。
- 设置img的默认src为一张1px*1px，很小很小的gif透明图片（所有的img都用这一张，只会发送一次请求），之所以需要时透明的，是需要透出通过background设置的背景图
- 需要一个滚动事件，判断元素是否在浏览器窗口，一旦进入视口才进行加载，当滚动加载的时候，就把这张透明的1px.gif图片替换为真正的url地址(也就是data-url里保存的值)
- 等到图片进入视口后，利用js提取data-url的真实图片地址赋值给src属性，就会去发送请求加载图片，真正实现了按需加载。

#### 实现方法：

代码均为核心逻辑代码

- 滚动监听+scrollTop+offsetTop+innerHeight

  ```js
         // 获取视口高度和内容的偏移量
             let clietH = window.innerHeight || document.documentElement.clientHeight || document.body.clientHeight;
             var scrollTop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop;
             console.log(clietH, scrollTop);
             for (let i = 0; i < imgs.length; i++) {
                 let x = scrollTop + clietH - imgs[i].offsetTop //当内容的偏移量+视口高度>图片距离内容顶部的偏移量时，说明图片在视口内
                 if (x > 0) {
                     imgs[i].src = imgs[i].getAttribute('data-url'); //从dataurl中取出真实的图片地址赋值给url
                 }
             }
  ```

- 滚动监听+getBoundingClientRect()

  ```js
  			// 获取视口高度和内容的偏移量
                    let offsetHeight = window.innerHeight || document.documentElement.clientHeight
                    Array.from(imgs).forEach((item, index) => {
                    let oBounding = item.getBoundingClientRect() //返回一个矩形对象，包含上下左右的偏移值
                    console.log(index, oBounding.top, offsetHeight);
                     if (0 <= oBounding.top && oBounding.top <= offsetHeight) {
                       	item.setAttribute('src', item.getAttribute('data-url'))
                       }
                     })
  ```

- intersectionObserve()

  ```js
  let imgs = document.getElementsByTagName('img')
              // 1. 一上来立即执行一次
              let io = new IntersectionObserver(function (entires) {
                  //图片进入视口时就执行回调
                  entires.forEach(item => {
                      // 获取目标元素
                      let oImg = item.target
                      // 当图片进入视口的时候，就赋值图片的真实地址
                      if (item.intersectionRatio > 0 && item.intersectionRatio <= 1) {
                          console.log(item);
                          oImg.setAttribute('src', oImg.getAttribute('data-url'))
                      }
                  })
              })
              Array.from(imgs).forEach(element => {
                  io.observe(element)  //给每一个图片设置监听
              });
  ```

  

# 12、简述jwt

全称为 JSON Web Token，在HTTP接口调用的时候，服务端需要对调用方做验证，以保证安全性，一种常见的方法就是jwt。

### 应用场景

- **认证**

  认证是JWT的最常用的场景，只要用户完成登录，其随后的请求都会包含jwt，以允许用户访问经由当前jwt授权的路由，服务或者是资源。由于开销小且能被简单应用在跨域访问上，jwt在分布式站点上所支持的**单点登录**(SSO)已经是当前它被广泛应用的一个特性

- **信息交换**

  JWT是一种在各参与方之间安全传递信息的良好方法。由于JWT可以被签名，因而可用于确认发送者自称的身份。除此之外，由于signature使用header和payload进行计算，也可以验证内容没有被篡改。

​	jwt的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其他业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

### jwt结构

在紧凑形式下，JSON Web Token 由以下三部分组成：

- Header(头部)
- Payload(载荷)
- Signature(签名)

因此一个典型的JWT形式如：xxx.yyy.zzz，由两个点间隔

# 13、微任务和宏任务

js分为同步任务和异步任务，而异步任务又分为宏任务和微任务，其中异步任务属于耗时任务。

### 微任务和宏任务都有哪些

- 宏任务
  - 整体代码script
  - setTimeout
  - setInterval
  - setImmediate
  - i/o操作（输入输出，比如读取文件操作，网络请求）
  - ui render
  - 异步Ajax
- 微任务
  - Promise(then、catch、finally)
  - async/await
  - process.nextTick
  - Object.observe(用来实时检测js中对象的变化)
  - MutationObserver(监听DOM树的变化)

### 执行顺序

- 宏任务执行顺序

  setImmediate ---> setTimeout ---> setInterval ---> i/o操作 ---> 异步ajax

- 微任务执行顺序

  process.nextTick ---> Promise

await会等待右侧函数代码执行之后阻塞当前async函数中后面的代码跳出执行外面的代码，async声明的函数如果没有await，等同于普通的同步函数

举个例子：

```js
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}

async function async2() {
    console.log('async2 start');
    await async3();
    console.log('async2 end');
}
async function async3() {
    console.log('async3');
    process.nextTick(() => {
        console.log('2')
    })
}
new Promise((resolve, reject) => {
    resolve('ok')
}).then((value) => {
    console.log(value);
})

process.nextTick(() => {
    console.log('1')
})

async1();


// 执行顺序为
// async1 start
// async2 start
// async3
// 1
// 2
// ok
// async2 end
// async1 end
```

这里面三个async声明函数执行顺序不难理解，但是在执行完毕后，先async2 end, 后async1 end，这是因为await阻塞当前async函数后面代码，async1函数先被阻塞，所以后面都执行完毕才会执行。第二个疑点就是async3函数内部的process.nextTick和在外部的process.nextTick才回去执行，promise由于在微任务中执行顺序在process之后，所以在他们后面执行。

# 14、简述WebSocket

WebSocket是一种**网络传输协议**，可在单个TCP连接上进行全双工通信，位于OSI模型的应用层。

http协议是只能客户端向服务端发送请求，然后服务端返回一个response，无法获取实时的信息传递。但是ajax轮询，long doll可以是http协议完成实时信息传递，但实现的原理就是频繁的发送请求。

WebSocket只需要和服务端建立一次连接，然后客户端只要有数据，就会源源不断的推送过来。

# 15、元素居中办法

- 绝对定位

  ```css
  div{
          height: 300px;
          width: 300px;
          background: red;
          position: absolute;
          top: 0;
          left: 0;
          right: 0;
          bottom: 0;
          margin: auto;
      }
  ```

  ```css
  div{
          height: 300px;
          width: 300px;
          background: red;
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
      }
  ```

  ```css
  div{
          height: 300px;
          width: 300px;
          background: red;
          position: absolute;
          top: 50%;
          left: 50%;
          margin-top: -150px;
          margin-left: -150px;
      }
  ```

- 弹性盒子

  ```css
  /* 父盒子上添加 */
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  ```


# 16、迭代器和生成器

### 迭代器

循环是迭代机制的基础，这是因为它可以指定迭代的次数，以及每次迭代要执行什么操作。每次循环都会在下一次迭代开始之前完成，而每次迭代的顺序都是事先定义好的。迭代会在一个有序集合上进行。数组是js中有序集合的最典型的例子。

任何实现Iterable接口的数据结构都可以被实现Iterable接口的结构“消费”。**迭代器**是按需创建的一次性对象。每个迭代器都会关联一个**可迭代对象**，而迭代器会暴露迭代其关联可迭代对象的API。

实现Iterable接口要求同时具备两种能力：支持迭代的自我识别能力和创建实现Iterable接口的对象的能力。以下内置类型实现了Iterable接口：

- 字符串

- 数组

- 映射

- 集合

- arguments对象

- NodeList等DOM集合类型

  ```js
  let str = 'abc';
  let arr =  ['a', 'b', 'c'];
  let map = new Map().set('a', 1).set('b', 2).set('c', 3);
  let set = new Set().add('a').add('b').add('c');
  
  console.log(str[Symbol.iterator]()); // Object [String Iterator] {}
  console.log(arr[Symbol.iterator]()); // Object [Array Iterator] {}
  console.log(map[Symbol.iterator]()); // [Map Entries] { [ 'a', 1 ], [ 'b', 2 ], [ 'c', 3 ] }
  console.log(set[Symbol.iterator]()); // [Set Iterator] { 'a', 'b', 'c' }
  ```

接收可迭代对象的原生语言特性包括：

- for-of循环
- 数组解构
- 扩展操作符
- Array.from()
- 创建集合
- 创建映射
- Promise.all()接收由期约组成的可迭代对象
- Promise.race()接收由期约组成的可迭代对象
- yield*操作符，在生成器中使用

next()方法返回的迭代器对象IteratorResult包含两个属性：done和value。done是一个布尔值，表示是否还可以再次调用next()取得下一个值；value包含可迭代对象的下一值(done为false)，或者undefined(done为true)。done: true状态称为"耗尽"。

#### 提前终止迭代器

可选的return()方法用于指定的迭代器提前关闭时执行的逻辑。执行迭代的结构在想让迭代器知道它不想遍历到可迭代对象耗尽时，就可以关闭"迭代器"。以下是可能的情况：

- for-of 循环通过break、continue、return或throw提前退出
- 解构操作并未消费所有的值

return()方法必须返回一个有效的IteratorResult对象。简单情况下，可以只返回{ done: true }。因为这个返回值只会在生成器的上下文中。 

### 生成器

生成器的形式是一个函数，函数名称前面加一个星号(*)表示它是一个生成器。只要是可以定义函数的地方，就可以定义生成器。

```js
// 生成器函数声明
function* generatorFn();
// 生成器函数表达式
let generatorFn = function* () {};
// 作为对象字面量方法的生成器函数
let foo = {
    * generatorFn() {}
}
// 作为类实例方法的生成器函数
class Foo{
    * generatorFn() {}
}
// 作为类静态方法的生成器函数
class Bar{
    static * generatorFn() {}
}
```

**注**：箭头函数不能用来定义生成器函数。

调用生成器函数会产生一个**生成器对象**，生成器对象一开始出于暂停执行的状态。与迭代器相似，生成器对象也实现了Iterator接口，因此具有next()方法。调用这个方法会让生成器开始或恢复执行。函数体为空的生成器函数中间不会停留，调用一次next()就会让生成器到达done: true。

```js
function * generatorFn(){}
const g = generatorFn();

console.log(g); // Object [Generator] {}
console.log(g.next()); // { value: undefined, done: true }
```

#### 通过yield中断执行 

yield可以让生成器停止和开始执行，也是生成器最有用的地方。生成器函数再遇到yield关键字之前会正常执行。遇到这个关键字后，执行会停止，函数作用域的状态会被保留。停止执行生成器函数只能通过生成器对象上调用next()方法来执行。

```js
function* generatorFn() {
    yield;
}

let generatorObject = generatorFn();
console.log(generatorObject.next()); // { value: undefined, done: false }
console.log(generatorObject.next()); // { value: undefined, done: true }
```

#### 提前中止生成器

与迭代器类似，生成器也支持“可关闭”的概念。一个实现Iterator接口的对象一定有next()方法，还有一个可选的return()方法用于提前终止迭代器。生成器对象除了有两个方法，还有第三个方法: thorw()

```js
function* generatorFn() { }

const g = generatorFn();
console.log(g); // Object [Generator] {}
console.log(g.next); // [Function: next]
console.log(g.return); // [Function: return]
console.log(g.throw); // [Function: throw]
```

# 17、js作用域链详解

作用域链包括**作用域**，**词法作用域**，**作用域链**

## 作用域

作用域就是变量(变量作用域又被称作上下文)和函数生效(能被访问)的区域或集合。也就是作用域决定了代码区块中变量和其他资源的可见性。

作用域一般分为以下三种:

- 全局作用域

  任何不在函数或是大括号中的变量，都是在全局作用域下，全局作用域中声明的变量可以在程序的任意位置访问

- 局部作用域

  如果一个变量被声明在函数内部，它就是在局部作用域，这个变量只能在函数内部访问，不能在函数以外去访问

- 块级作用域

  ES6引入了let和const关键字，和var关键字不同，在大括号中使用let和const声明的变量存在于块级作用域中，在大括号之外不能访问这些变量

## 词法作用域

词法作用域又被称为静态作用域，变量被创建时就确定好了，而非执行阶段确定的，也就是说我们写好代码的时候它的作用域就已经确定了，js所遵循的就是词法作用域

```js
var a = 2;
function foo(){
    console.log(a);
}

function bar() {
    var a = 3;
    foo(); // 输出 2
}
bar();
```

以上代码中，由于两个函数都是在全局作用域内，所以它们不能访问到互相的局部作用域内的值，所以foo函数还是返回2

## 作用域链

当在js中使用一个变量的时候，首先js引擎会尝试在当前作用域下去寻找，如果找不到，在上层作用域中去找以此类推，直到找到为止，但是如果直到全局作用域仍然找不到该变量，他就会在全局范围内隐式声明该变量(非严格模式下)或者直接报错。

# 18、前端SSR技术

SSR指的是服务端渲染(Server-side Rendering)，是在服务端渲染网页内容，并且把渲染后的HTML发送给浏览器，而不是在浏览器端进行渲染。

### 优点：

- **更快的首屏加载速度**

  因为浏览器不需要等待数据加载和渲染，因此可以提高用户的首屏体验。

- **更好的搜索引擎优化(SEO)**

  因为搜索引擎可以更容易地爬取完整的HTML内容，因此SSR可以更好的改善SEO

- **更好的用户体验**

  因为页面的内容在服务端进行渲染，因此用户可以更快地看到完整地页面，从而提高用户的使用体验

### 缺点：

- **增加服务器的负载**

  因为服务器要渲染HTML页面，因此可能会增加服务器的负载

- **更复杂的环境技术**

  因为需要设计到服务端代码，因此需要更复杂的技术环境

- **更高的开发复杂度**

  因为需要服务端和客户端同时开发，因此可能会更高的开发复杂度

对于复杂的页面来说，可能会造成性能问题：因为服务器需要处理大量的数据和渲染，因此造成性能问题。

### 总结

是否使用SSR取决于业务场景和需求，如果有大量的静态内容的简单页面，客户端渲染是一个更好的选择。但是对于需要从服务器加载动态数据的复杂页面，SSR可能是一个更好的选择。

