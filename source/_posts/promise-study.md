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




# 6、手写promise源码

- **Promise**构造函数声明部分

  ```js
  // 声明构造函数
  function Promise(executor) {
      // 添加属性
      this.PromiseState = 'pending';
      this.PromiseResult = null;
      this.callbacks = [];
  
      // 改变this指向
      let self = this;
      // resolve 函数
      function resolve(data) {
          // 判断状态
          if(self.PromiseState !== 'pending') return;
          // 修改对象的状态 (PromiseState)
          self.PromiseState = 'fulfilled'
          // 修改对象的结果值 (PromiseResult)
          self.PromiseResult = data;
          // 调用callback回调函数 处理异步
          setTimeout(() => {
              self.callbacks.forEach((item) => {
                  item.onResolve(data)
              })
          })
      }
      // reject 函数
      function reject(data) {
          // 判断状态
          if (self.PromiseState !== 'pending') return;
          // 修改对象的状态 (PromiseState)
          self.PromiseState = 'rejected'
          // 修改对象的结果值 (PromiseResult)
          self.PromiseResult = data;
          // 调用callback回调函数 // 处理异步
          setTimeout(() => {
              self.callbacks.forEach((item) => {
                  item.onRejected(data)
              })
          })
      }
      try {
          // 同步调用执行器函数
          executor(resolve, reject);
      } catch (error) {
          // 修改对象状态为失败
          reject(error);
      }
  }
  ```

- **then**方法的添加

  ```js
  // 添加 then 方法
  Promise.prototype.then = function (onResolve, onRejected){
      const self = this;
      // 判断回调函数参数
      if(typeof onRejected !== 'function') {
          onRejected = (reason) => {
              throw reason
          }
      }
      if(typeof onResolve !== 'function') {
          onResolve = (value) => {
              return value
          }
      }
      return new Promise((resolve, reject) => {
  
          // 封装函数
          function callback(type) {
              try {
                  // 取得返回值
                  let result = type(self.PromiseResult);
                  if (result instanceof Promise) {
                      // 返回值为Promise
                      result.then((v) => {
                          resolve(v);
                      }, (r) => {
                          reject(r)
                      })
                  } else {
                      // 返回值不为promise
                      resolve(result)
                  }
              } catch (error) {
                  reject(error)
              }
          }
          // 设置回调函数 PromiseState
          if (this.PromiseState === 'fulfilled') {
              setTimeout(() => {
                  callback(onResolve);
              })
          }
  
          if (this.PromiseState === 'rejected') {
              setTimeout(() => {
                  callback(onRejected);
              })
          }
          if (this.PromiseState === 'pending') {
              this.callbacks.push({
                  onResolve: function() {
                      callback(onResolve);
                  },
                  onRejected: function() {
                      callback(onRejected);
                  }
              })
          }
      })
  }
  ```

- **catch**方法添加

  ```js
  // 添加catch方法
  Promise.prototype.catch = function(onRejected) {
      return this.then(undefined, onRejected)
  }
  ```

- **resolve**方法添加

  ```js
  // 添加resolve方法
  Promise.resolve = function(value) {
      return new Promise((resolve, reject) => {
          if(value instanceof Promise) {
              value.then((v) => {
                  resolve(v)
              }, (r) => {
                  reject(r)
              })
          }else{
              resolve(value);
          }
      })
  }
  ```

- **reject**方法添加

  ```js
  // 添加reject方法
  Promise.reject = function(reason) {
      return new Promise((resolve, reject) => {
          reject(reason);
      })
  }
  ```

- **Promise.all方法（重点）**

  ```js
  // 添加Promise.all方法
  Promise.all = function(promises){
      return new Promise((resolve, reject) => {
          // 声明变量
          let count = 0;
          let arr = [];
  
          //循环接收到的数组
          for(let i = 0; i < promises.length; i++){
              // 由于返回的都是promise对象，所以用.then方法来获取
              promises[i].then((v) => {
                  // 每次成功后使count+1做个识别
                  count++;
                  // 返回的数组
                  arr[i] = v;
                  // 判断 即为全部返回成功后改变状态为成功
                  if(count === promises.length) {
                      resolve(arr);
                  }
              }, (r) => {
                  // 如若有一个返回失败，即为失败
                  reject(r)
              })
          }
      })
  }
  ```

- **Promise.race**方法

  ```js
  // 添加Promise.race方法
  Promise.race = function(promises) {
      return new Promise((resolve, reject) => {
          for(let i = 0; i < promises.length; i++) {
              promises[i].then((v) => {
                  resolve(v)
              }, (r) => {
                  reject(r)
              })
          }
      })
  }
  ```

# 7、promise的类封装

resolve、reject、all、race由于不是实例方法，所以需要加static关键字

```
class Promise{
    // 构造方法
    constructor(executor) {
        // 添加属性
        this.PromiseState = 'pending';
        this.PromiseResult = null;
        this.callbacks = [];

        // 改变this指向
        let self = this;
        // resolve 函数
        function resolve(data) {
            // 判断状态
            if (self.PromiseState !== 'pending') return;
            // 修改对象的状态 (PromiseState)
            self.PromiseState = 'fulfilled'
            // 修改对象的结果值 (PromiseResult)
            self.PromiseResult = data;
            // 调用callback回调函数 处理异步
            setTimeout(() => {
                self.callbacks.forEach((item) => {
                    item.onResolve(data)
                })
            })
        }
        // reject 函数
        function reject(data) {
            // 判断状态
            if (self.PromiseState !== 'pending') return;
            // 修改对象的状态 (PromiseState)
            self.PromiseState = 'rejected'
            // 修改对象的结果值 (PromiseResult)
            self.PromiseResult = data;
            // 调用callback回调函数 // 处理异步
            setTimeout(() => {
                self.callbacks.forEach((item) => {
                    item.onRejected(data)
                })
            })
        }
        try {
            // 同步调用执行器函数
            executor(resolve, reject);
        } catch (error) {
            // 修改对象状态为失败
            reject(error);
        }
    }
    // then方法封装
    then(onResolve, onRejected){
        const self = this;
        // 判断回调函数参数
        if (typeof onRejected !== 'function') {
            onRejected = (reason) => {
                throw reason
            }
        }
        if (typeof onResolve !== 'function') {
            onResolve = (value) => {
                return value
            }
        }
        return new Promise((resolve, reject) => {

            // 封装函数
            function callback(type) {
                try {
                    // 取得返回值
                    let result = type(self.PromiseResult);
                    if (result instanceof Promise) {
                        // 返回值为Promise
                        result.then((v) => {
                            resolve(v);
                        }, (r) => {
                            reject(r)
                        })
                    } else {
                        // 返回值不为promise
                        resolve(result)
                    }
                } catch (error) {
                    reject(error)
                }
            }
            // 设置回调函数 PromiseState
            if (this.PromiseState === 'fulfilled') {
                setTimeout(() => {
                    callback(onResolve);
                })
            }

            if (this.PromiseState === 'rejected') {
                setTimeout(() => {
                    callback(onRejected);
                })
            }
            if (this.PromiseState === 'pending') {
                this.callbacks.push({
                    onResolve: function () {
                        callback(onResolve);
                    },
                    onRejected: function () {
                        callback(onRejected);
                    }
                })
            }
        })
    }
    // catch方法封装
    catch(onRejected) {
        return this.then(undefined, onRejected)
    }
    // resolve方法封装
    static resolve(value) {
        return new Promise((resolve, reject) => {
            if (value instanceof Promise) {
                value.then((v) => {
                    resolve(v)
                }, (r) => {
                    reject(r)
                })
            } else {
                resolve(value);
            }
        })
    }
    // reject方法封装
    static reject(reason) {
        return new Promise((resolve, reject) => {
            reject(reason);
        })
    }
    // Promise.all
    static all(promises){
        return new Promise((resolve, reject) => {
            // 声明变量
            let count = 0;
            let arr = [];

            //循环接收到的数组
            for (let i = 0; i < promises.length; i++) {
                // 由于返回的都是promise对象，所以用.then方法来获取
                promises[i].then((v) => {
                    // 每次成功后使count+1做个识别
                    count++;
                    // 返回的数组
                    arr[i] = v;
                    // 判断 即为全部返回成功后改变状态为成功
                    if (count === promises.length) {
                        resolve(arr);
                    }
                }, (r) => {
                    // 如若有一个返回失败，即为失败
                    reject(r)
                })
            }
        })
    }
    // Promise.race
    static race(promises) {
        return new Promise((resolve, reject) => {
            for (let i = 0; i < promises.length; i++) {
                promises[i].then((v) => {
                    resolve(v)
                }, (r) => {
                    reject(r)
                })
            }
        })
    }
}
```

