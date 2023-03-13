---
title: js设计模式
date: 2023-3-11 13:16
categories: javaScript
---

# js设计模式

## 观察者模式(发布/订阅模式)

观察者模式又被称作发布订阅模式，它定义了一种一对多的关系，所有的订阅者同时观察一个主题对象，当主题对象发生变化时，就会通知所有的订阅者，使订阅者们能够自动更新自己。

**优点：解耦**

- 时间上的解耦

  注册的订阅行为有发布方来决定何时调用，订阅者不用持续关注，当消息发生时发布者会通知订阅者。

- 对象上的解耦

  发布者不需要知道消息的接收者是谁，只需要遍历所有订阅者将消息发出去即可。

**缺点：**

- 增加消耗: 创建结构和缓存订阅者这两个过程都需要消耗性能和内存资源，即使订阅后之中没有接收到消息，订阅者也始终会在内存中
- 增加复杂度：订阅者被缓存在一起，如果订阅者和发布者层层嵌套，程序将变得难以追踪和调试
- 缺点主要在于理解成本、运行效率、资源消耗，特别是在多级发布-订阅时，情况会变得更复杂

```js
function Journal() {
    const fnList = [];
    return {
        // 订阅
        subscribe: (fn) => {
            const index = fnList.indexOf(fn);
            // 如果订阅列表存在，无需重复添加
            if (index !== -1) return fnList;
            fnList.push(fn);
            return fnList;
        },
        // 退订
        unsubscribe: (fn) => {
            const index = fnList.indexOf(fn);
            // 如果订阅链表存在，无需退订
            if(index === -1) return fnList;
            fnList.splice(index, 1);
            return fnList;
        },
        // 发布
        notify: () => {
            fnList.forEach((item) => {
                item.update();
            })
        }
    }
}

const o = new Journal();

// 创建订阅者
function Observer(person, data) {
    return {
        update: () => {
            console.log(`${person}:${data}`);
        }
    }
}

const f1 = new Observer('jfz', '123');
const f2 = new Observer('zpz', '456');
const f3 = new Observer('zj', '789');

// f1,f2,f3订阅了
o.subscribe(f1);
o.subscribe(f2);
o.subscribe(f3);

// f1去取消了订阅
o.unsubscribe(f1)

o.notify();
// zpz:456
// zj: 789
```

# 策略模式

定义：定义一系列算法，把它们一个个封装起来，并且是他们可以相互替换。

策略模式的优缺点：

- 优点：
  - 策略之间相互独立，但策略可以自由切换，这个特点给策略模式增加很多灵活性，也增加了策略的复用率
  - 如果不采用策略模式，那么在选策略时一般会采用多重的条件判断，采用策略模式可以避免多重条件判断，增加可维护性
  - 可扩展性好，策略可以很方便的进行扩展
- 缺点：
  - 策略相互独立，因此一些复杂的算法逻辑无法共享，造成一些资源浪费
  - 如果用户想采用什么策略，必须了解策略的实现，因此所有策略都需向外暴露，这是违背迪米特法则(最少知识原则的)，也增加了用户对策略对象的使用成本

策略模式的适用场景：

- 多个算法只在行为上稍有不同的场景，这时可以使用策略模式来动态选择算法
- 算法需要自由切换场景
- 有时需要多重条件判断，那么可以使用策略模式来规避多重条件判断情况

```js
//不用策略模式
let CLfun = function (props, salary) {
    if (props === 'Z') {
        return salary * 4;
    }
    if (props === 'F') {
        return salary * 3;
    }
    if (props === 'J') {
        return salary * 2;
    }
};
CLfun('J', 20000); // 输出：40000
CLfun('Z', 6000); // 输出：24000

// 使用策略模式优化
let obj = {
    'J': function(salary){
        return salary*2;
    },
    'F': function(salary) {
        return salary*3;
    },
    'Z': function(salary) {
        return salary*4;
    }
}

let CLfun = function(props, salary) {
    return obj[props](salary)
}

CLfun('J', 2000); // 输出: 2000

```

# 单例模式

创建一个单独的实例化对象，输入参数来创建一个对象。

特点:

- 单例类只能有一个实例
- 单例类必须自己创建自己的唯一实例
- 单例类必须给其他对象提供这一实例

```js
let box;
const createBox = (_a, _b) => {
    if(!box) {
        box = {};
    }

    box.a = _a;
    box.b = _b;
    return box;
}

let obj1 = createBox(3, 6);
console.log(obj1); // { a: 3, b: 6 }

let obj2 = createBox(10, 20);
console.log(obj1); // { a: 10, b: 20 }
console.log(obj2); // { a: 10, b: 20 }
```

由上面代码可见，如果已经创建了对象，再次创捷则会影响上一次创建的值，所以单例模式一般只创建一个实例。
