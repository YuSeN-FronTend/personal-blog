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

​	








