---
title: js事件循环
date: 2022-11-26 10:33
categories: javaScript
---

![](http://106.55.171.176:9000/yusen/Snipaste_2022-11-26_11-15-34.png)

<!-- more -->

### js 是单线程的

js是一种单线程的编程语言，只有一个调用栈，在代码执行的时候，通过将不同函数的执行上下文压入执行栈中来保证代码的有序执行。如果在执行同步代码时遇到了异步事件，js的引擎不会一直等待返回其结果，而是会将这个事件挂起，继续去执行栈中的其它任务。所以js也是一个非阻塞，异步，并发式的编程语言。

### 同步和异步

#### 同步任务

指的是在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。可以理解为在执行完一个函数或方法之后，一直等待系统的返回值或消息，这时候程序是处于堵塞的。只有接收到返回的值或消息后才往下继续执行。

#### 异步任务

指的是不进入主线程，某个异步任务执行了，该任务才会进入主线程执行。执行完函数或方法后，不必阻塞性的等待返回值或消息，只需要向系统委托一个异步过程，当系统接收到返回值或消息时，系统回自动触发委托的异步过程，从而完成一个完整的流程。

### 事件循环(event loop)

```javascript
console.log(1);
setTimeout(() => {
  console.log(2);
}, 0);
setTimeout(() => {
  console.log(3);
}, 0);
setTimeout(() => {
  console.log(4);
}, 0);
console.log(5);
```

输出 **1 5 2 3 4**

#### 事件循环的过程

简单来说就是自顶向下执行，遇到同步任务直接执行，遇到异步任务放到队列中，当同步任务执行过后再执行异步任务。

### 任务队列

在js中，异步任务分为宏任务和微任务。

#### 宏任务

宏任务的范畴很广，包括创建主文档对象，解析HTML，执行主线(或全局)javaScript代码，更改当前URL以及各种事件，如页面加载，输入，网络事件和定时器事件。

#### 微任务

微任务是更小的任务，微任务更新应用程序的状态，重要的一点，**微任务可以插宏任务的队**。

**举几个例子能更深刻的了解事件循环**

```javascript
console.log("script start");
 
setTimeout(function () {
  console.log("setTimeout");
}, 0);
 
Promise.resolve()
  .then(function () {
    console.log("promise1");
  })
  .then(function () {
    console.log("promise2");
  });
 
console.log("script end");
```

输出 

**script start
script end
promise1
promise2
setTimeout**

```javascript
console.log("script start");
 
setTimeout(function () {
  console.log("timeout1");
}, 10);
 
new Promise((resolve) => {
  console.log("promise1");
  resolve();
  setTimeout(() => console.log("timeout2"), 10);
}).then(function () {
  console.log("then1");
});
 
console.log("script end");
```

输出 

**script start
promise1
script end
then1
timeout1
timeout2**

#### async && await

这两个其实就是Generator和Promise的语法糖，async函数和普通函数没什么区别，只是表示这个函数中有个异步方法，返回一个promise对象。

```javascript
// async 和 await 版
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
// Promise版
async function async1() {
  console.log("async1 start");
  Promise.resolve(async2()).then(() => console.log("async1 end"));
}

```

以上两者是完全相同的，所以看下面例子。

```javascript
async function async1() {
    console.log("async1 start");
    await async2();
    console.log("async1 end");
}
async function async2() {
    console.log("async2");
}
async1();
setTimeout(() => {
    console.log("timeout");
}, 0);
new Promise(function (resolve) {
    console.log("promise1");
    resolve();
}).then(function () {
    console.log("promise2");
});
console.log("script end");
```

自顶向下开始执行，**async1 start**为同步任务，直接输出，执行async2(),直接输出**async2**，然后把**async1 end**添加到微任务队列，遇到**timeout**加入宏任务队列，遇到**promise1**直接输出，由于**resolve()**将**promise2**加入微任务队列，遇到**script end**直接执行。然后执行队列中的微任务**async1 end**和**promise2**，最后执行宏任务**timeout**

输出 

**async1 start
async2
promise1
script end
async1 end
promise2
timeout**

### 最终练习

```javascript
console.log('1');

setTimeout(function () {
    console.log('2');
    process.nextTick(function () {
        console.log('3');
    })
    new Promise(function (resolve) {
        console.log('4');
        resolve();
    }).then(function () {
        console.log('5')
        setTimeout(function () {
            console.log('6');
            process.nextTick(function () {
                console.log('7');
            })
            new Promise(function (resolve) {
                console.log('8');
                resolve();
            }).then(function () {
                console.log('9')
            })
        })
    })
})
process.nextTick(function () {
    console.log('10');
})
new Promise(function (resolve) {
    console.log('11');
    resolve();
}).then(function () {
    console.log('12')
    setTimeout(function () {
        console.log('13');
        process.nextTick(function () {
            console.log('14');
        })
        new Promise(function (resolve) {
            console.log('15');
            resolve();
        }).then(function () {
            console.log('16')
        })
    })
})

setTimeout(function () {
    console.log('17');
    process.nextTick(function () {
        console.log('18');
    })
    new Promise(function (resolve) {
        console.log('19');
        resolve();
    }).then(function () {
        console.log('20')
    })
})
console.log('21')
```



输出 **1 11 21 10 12 2 4 3 5 17 19 18 20 13 15 14 16 6 8 7 9**