---
title: vue2数据响应式原理
date: 2023-4-9 19:59
categories: vue
---

# 数据劫持

vue2用Object.defineProperty来完成数据劫持，使用自定义的getter和setter来重写原有的行为。具体如下

```js
let obj = {};
Object.defineProperty(obj, 'a', {
    // getter
    get() {
        console.log('访问obj的a属性');
        return 7;
    },
    // setter
    set(newValue) {
        console.log('改变了obj的a属性', newValue)
    }
})

console.log(obj.a); //访问obj的a属性  7
obj.a = 9    // 改变了obj的a属性  9
console.log(obj.a); // 访问obj的a属性 7
```

上面的代码也确实引申除了问题，就是无法保存变量使得obj里面的a属性完成修改，所以提到保存变量，最先想到的一定是闭包

创建defineReactive()函数来辅助实现闭包

```js
let obj = {};

function defineReactive(data, key, val) {
    Object.defineProperty(data, key, {
        // 可枚举
        enumerable: true,
        // 可以被配置，比如可以被delete
        configurable: true,
        get() {
            console.log('访问obj的a属性');
            return val;
        },
        set(newVal) {
            console.log('改变了obj的a属性');
            // 如果新旧值相等 无需修改
            if(val === newVal) {
                return
            }
            val = newVal;
        }
    })
}
defineReactive(obj, 'a', 1)

console.log(obj.a); // 访问obj的a属性 1
obj.a = 9           // 改变了obj的a属性
console.log(obj.a); // 访问obj的a属性 9
```

如果obj里有多个属性可以新建一个类`Observe`来遍历改对象

```js
class Observer{
    constructor(value) {
        this.value = value;
        this.walk();
    }
    walk() {
        Object.keys(this.value).forEach((key) => defineReactive(obj, key))
    }
}
```

如果obj有嵌套属性，可以用递归来完成数据劫持

```js
function observe(data) {
    if(typeof data !== 'object') return;
    // 调用observe
    new Observer(data);
}

class Observer {
    constructor(value) {
        this.value = value;
        this.walk();
    }
    walk() {
        Object.keys(this.value).forEach((key) => defineReactive(this.value, key))
    }
}

function defineReactive(data, key, val = data[key]) {
    // 监听传过来的值
    observe(val);
    Object.defineProperty(data, key, {
        // 可枚举
        enumerable: true,
        // 可以被配置，比如可以被delete
        configurable: true,
        get() {
            console.log(`访问obj${key}的属性`);
            return val;
        },
        set(newVal) {
            console.log(`改变obj${key}的属性`);
            // 如果新旧值相等 无需修改
            if(val === newVal) {
                return
            }
            val = newVal;
            observe(newVal) // 设置的新值也要被监听
        }
    })
}
const obj = {
    a: 1,
    b: {
        c:2
    }
}
```

那此对象来说，obj进入observe，是对象，进入Observer中执行walk方法执行遍历，defineReactive(obj,a)先执行，obj.a进入observe判断不是对象，直接返回，然后执行defineReactive剩余代码，defineReactive(obj,b)执行，进入observe判断是对象，进入Observer中继续执行walk方法，执行defineReactive(obj.b,c)，进入observe判断不是对象直接返回，执行玩剩余的defineReactive(obj.b,c)代码，而后执行玩剩余的defineReactive(obj,b)，代码执行结束。

综上所述这三者之间是相互调用的。我们的响应式最终要实现的效果是：在数据变化时，只更新与这个数据相关的数据，这就引出了下面的内容`依赖`

# 收集依赖与派发更新

## 依赖

结合对发布订阅模式的理解，创建的Watcher.js内部代码如下

```js
export default class Watcher {
    constructor(data, expression, cb) {
        // data 数据对象 如obj
        // expression 表达式，如b.c，根据data和expression就可以获取watcher依赖的数据
        // cd 依赖变化时触发的回调
        this.data = data;
        this.expression = expression;
        this.cb = cb;
        // 初始化watcher实例时订阅数据
        this.value = this.get();
    }

    get() {
        const value = parsePath(this.data, this.expression);
        return value
    }

    // 当收到数据变化的消息时执行该方法，从而调用cb
    update() {
        this.value = parsePath(this.data, this.expression) //对储存的数据进行更新
        this.cb()
    }
}

function parsePath(obj, expression) {
    const segments = expression.split('.');
    for(let key of segments) {
        if(!obj) return;
        obj = obj[key];
    }
    return obj;
}
```

我们要实现的功能有

- 有一个数组来储存`watcher`
- `watcher`实例需要订阅数据，也就是获取依赖或者收集依赖
- `watcher`的依赖发生变化时触发watcher的回调函数，也就是派发更新

每个数据都应该维护一个属于自己的数组，该数组来存放依赖自己的`watcher`，我们可以在`defineReactive`中定义一个数组`dep`，这样通过闭包，每个属性就能拥有一个属于自己的`dep`

```js
function defineReactive(data, key, val = data[key]) {
    const dep = [] // 增加
    // 监听传过来的值
    observe(val);
    Object.defineProperty(data, key, {
        // 可枚举
        enumerable: true,
        // 可以被配置，比如可以被delete
        configurable: true,
        get() {
            console.log(`访问obj${key}的属性`);
            return val;
        },
        set(newVal) {
            console.log(`改变obj${key}的属性`);
            // 如果新旧值相等 无需修改
            if(val === newVal) {
                return
            }
            val = newVal;
            observe(newVal) // 设置的新值也要被监听
            dep.notify();
        }
    })
}
```

## 依赖收集

我肯可以在`getter`中把当前`watcher`加入到dep数组 中，即可完成依赖收集。

对`getter`进行一些更改

```js
get() {
            dep.push(watcher);
            return val;
        },
```

但是里面的变量`watcher`没有，我们要把watcher实例放到全局，比如`window.target`上。因此，`Watcher`的get方法做如下修改

```js
get() {
        window.target = this;
        const value = parsePath(this.data, this.expression);
        return value
    }
```

这样修改后将getter中的`dep.push(watcher)`修改为`dep.push(window.target)`即可

## 派发更新

派发更新可以在`setter`中进行

```js
set(newVal) {
            console.log(`改变obj${key}的属性`);
            // 如果新旧值相等 无需修改
            if(val === newVal) {
                return
            }
            val = newVal;
            observe(newVal) // 设置的新值也要被监听
            dep.forEach(d => d.update()) 
        }
```

# 优化代码

## Dep类

将dep数组抽象为类

```js
class Dep {
    constructor() {
        this.subs = []
    }

    depend() {
        this.addSub(Dep.target)
    }

    notify() {
        const subs = [...this.subs];
        subs.forEach((s) => s.update())
    }

    addSub(sub){
        this.subs.push(sub)
    }
}
```

`defineReactive`函数做如下更改

```js
function defineReactive(data, key, val = data[key]) {
    const dep = new Dep();
    // 监听传过来的值
    observe(val);
    Object.defineProperty(data, key, {
        // 可枚举
        enumerable: true,
        // 可以被配置，比如可以被delete
        configurable: true,
        get() {
            dep.depend();
            return val;
        },
        set(newVal) {
            console.log(`改变obj${key}的属性`);
            // 如果新旧值相等 无需修改
            if(val === newVal) {
                return
            }
            val = newVal;
            observe(newVal) // 设置的新值也要被监听
            dep.notify();
        }
    })
}
```

## window.target

修改Watcher中的get方法

```js
 get() {
        window.target = this;
        const value = parsePath(this.data, this.expression);
        window.target = null
        return value
    }
```

修改Dep中的depend方法

```js
depend() {
        if(Dep.target) {
            this.addSub(Dep.target)
        }
    }
```

## update方法

```js
update() {
        const oldValue = this.value;
        this.value = parsePath(this.data, this.expression) //对储存的数据进行更新
        this.cb().call(this.data, this.value, oldValue)
    }
```

# 总结

- **Observe**

  Observe方法结合defineReactive和observe来进行循环递归，从而实现将一个对象变为响应式。在组件的生命周期中，此事件在beforeCreate之后，created之前，由于无法动态添加或删除属性，$set和$delete两个实例方法。对于数组，此事件会重写数组中的七个方法：`push, pop, unshift, shift, sort, splice, reverse`。总之此事件就是为了让一个对象属性的读取，修改，内部数组的变化都可以被vue感知到

- **Dep**

  Observe执行过后还有两个问题没有解决，读取数据要做什么，修改数据要做什么。这时就需要Dep。每个Dep都可以做两件事：

  - 记录依赖

    是哪里在调用此数据

  - 派发更新

    数据修改之后通知哪里也去修改对应数据

  也就是在读取响应式对象的某一个属性时，Dep可以收集到那里使用了此数据。在修改响应式数据时可以通知对应的数据进行修改。

  上面提到$set和$delete，vue不建议使用这两个属性，因为在模板中没有用到值，凭空加了一条数据不会重新运行render函数，但是上一级dep发现自身改变了，就会导致重新运行render

- **Watcher**

  上述代码执行之后，dep如何知道是哪里再用数据呢，watcher就解决了这个问题。我们不要去直接执行函数，而是去把函数交给watcher。wtacher会创建一个全局变量，表示当前执行的watcher等于自己，然后再去执行函数，在函数执行过程中，如果有了依赖记录，那么dep就会把这个全局变量记录下来，表示有一个watcher用到了我这个属性

  当dep派发更新时，就会通知所有watcher用到此属性一起改变

  

  