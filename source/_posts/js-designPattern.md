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

