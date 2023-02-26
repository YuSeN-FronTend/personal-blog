---
title: promise
date: 2023-2-16 8:53
categories: javaScript
---

# 1、Promise是什么

- Promise是一门新的技术(ES6规范)
- Promise是JS中进行异步编程的新解决方案 (旧解决方案是单纯使用回调函数)
- 从**语法**上来讲：Promise是一个构造函数
- 从**功能**上来说：Promise对象用来封装一个异步对象，并可以获取其成功/失败的结果值

# 2、为什么要用Promise

##### 支持链式调用，可以解决回调地狱问题

- **回调地狱**就是回调函数嵌套调用，外部回调函数异步执行的结果是嵌套的回调执行的条件
- 回调地狱的缺点：
  - 不便于阅读
  - 不便于异常处理

# 3、Promise的状态改变

三种状态：

- **pending**(未决定的)
- **resolve**(成功)
- **reject**(失败)

状态改变：

- **pending**转**resolve**
- **pending**转**reject**

说明：

状态转变只有这两种，且promise对象只能改变一次，无论变为成功还是变为失败，都会有一个结果数据，成功的结果数据一般称为**value**，失败的结果数据一般称为**reason**

# 4、Promise的方法

- **resolve()**

  如果传入的参数为非Promise类型的对象，则返回的结果就是成功的promise对象

  如果传入的参数是Promise类型的对象，则参数的结果决定了resolve的结果

- **reject()**

  无论参数传入的是什么，结果始终会返回失败

- **all()**

  参数为包含很多promise对象的数组，只要数组中有一个promise对象返回失败，直接失败，除非都成功才成功。

- **race()**

  参数同样为很多promise对象的数组，但是状态由数组中第一个promise对象来决定，顾名思义就是"赛跑"

# 5、Promise需要注意的问题

- 改变promise状态的三种方法

  - **resolve**
  - **reject**
  - **thorw**

  ```js
  let p = new Promise((resolve, reject) => {
              // 1.调用resolve
              // resolve('ok')
              // 2.调用reject
              // reject('err')
              // 3.利用throw抛出错误
              // throw '123';
          })
  ```

  

- 一个promise指定多个函数的回调函数的时候，当promise为对应状态时会全调用的

```js
let p = new Promise((resolve, reject) => {
            resolve('ok')
        })

        p.then((value) => {
            console.log(value); // ok
        })

        p.then((value) => {
            console.log(value + 1); // ok1
        })
```

- 修改promise状态和.then的先后执行顺序是都有可能的，如下列代码加个定时器就是.then先执行

  ```js
  let p = new Promise((resolve, reject) => {
              setTimeout(() => {
                  resolve('ok');
              }, 1000)
          })
  
          p.then((value) => {
              console.log(value);
          })
  ```

- 在执行.then时也可以修改状态，见如下代码

  ```js
  let p = new Promise((resolve, reject) => {
              resolve('ok')
          })
  
          let result = p.then((value) => {
              // console.log(value);
              // 1.抛出错误
              // throw '出了问题'; // 失败
              // 2.返回结果是非Promise类型的对象
              // return "521" // 返回成功
              // 返回的结果是Promise对象
              return new Promise((resolve, reject) => {
                  // resolve('success'); // 成功
                  // reject('error'); // 失败
              })
          }, (reason) => {
              console.warn(reason);
          })
          console.log(result);
  ```

- promise的串联实现

  ```js
  let p = new Promise((resolve, reject) => {
              setTimeout(() => {
                  resolve('ok')
              }, 1000) 
          })
  
          p.then((value) => {
              console.log(value); // ok
              return new Promise((resolve, reject) => {
                  resolve('success')
              })
          }).then((value) => {
              console.log(value); // success
          }).catch((reason) => {
              console.log(reason); // undefined
          })
  ```

- 异常穿透(即结尾的catch可以捕获到以上任意一个reject)

  ```js
  let p = new Promise((resolve, reject) => {
              setTimeout(() => {
                  reject('ok')
              }, 1000) 
          })
  
          p.then((value) => {
              return new Promise((resolve, reject) => {
                  resolve('success')
              })
          }).then((value) => {
              console.log(value);
          }).catch((reason) => {
              console.log(reason); // ok
          })
  ```

- 中断promise

  ```js
  let p = new Promise((resolve, reject) => {
          setTimeout(() => {
              resolve('ok')
          }, 1000)
      })
  
      p.then((value) => {
          console.log(value);  // ok
          return new Promise(() => {})
      }).then((value) => {
          console.log(value); // 由于中断没有执行
      })
  ```

  