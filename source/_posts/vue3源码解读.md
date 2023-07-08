---
title: vue3源码解读
date: 2023-7-8 14:40
categories: vue
---

# 响应式系统

肯定是使用proxy代理进行的数据劫持，但是还有很多细节需要剖析

## 简单实现

使用proxy进行数据劫持，当监听的值改变后响应执行副作用函数，即可将数据变为响应式，如下代码所示

```js
let box = document.getElementById('box');
let data = {
    test: 'Hello World'
}
let bucket = new Set();
let obj = new Proxy(data, {
    get(target, property) {
        bucket.add(effect);
        return target[property]
    },
    set(target, property, newValue) {
        target[property] = newValue;
        bucket.forEach((item) => item())
        return true;
    }
})

function effect() {
    box.innerText = obj.test;
}
effect();
```

但此代码还存在很多细节上的问题，比如如果副作用函数不叫effect，则无法捕获到副作用函数

## 设置一个全局变量函数

通过设置一个全局变量函数，接收一个匿名函数即可完成对任意函数的监听，并不局限于函数名，见如下代码

```js
let box = document.getElementById('box');
let data = {
    test: 'Hello World'
}
let bucket = new Set();
let obj = new Proxy(data, {
    get(target, property) {
        if(activeEffect){
            bucket.add(activeEffect);
        }
        return target[property]
    },
    set(target, property, newValue) {
        target[property] = newValue;
        bucket.forEach((item) => item())
        return true;
    }
})
let activeEffect;
function effect(fn) {
    activeEffect = fn;
    fn()
}
effect(
    () => {
        box.innerText = obj.test;
    }
);
```

但我们还没有在副作用函数于被操作的目标字段之间建立明确的联系，这时我们就需要重新建立桶的数据结构

## 修改桶的数据结构

首先需要使用WeakMap来代替Set来作为桶的数据结构，因为WeakMap对key是弱引用，当它内部的值没有被引用时，不会影响垃圾回收机制，可以保证内存不溢出，如下代码所示

```js
let box = document.getElementById('box');
let data = {
    test: 'Hello World'
}
let activeEffect;
let bucket = new WeakMap();
let obj = new Proxy(data, {
    get(target, property) {
        track(target, property)
        return target[property]
    },
    set(target, property, newValue) {
        target[property] = newValue;
        trigger(target, property)
    }
})
function track(target, property) {
    if (!activeEffect) {
        return target[property]
    }
    let depsMap = bucket.get(target);
    if (!depsMap) {
        bucket.set(target, (depsMap = new Map()))
    }
    let deps = depsMap.get(property);
    if (!deps) {
        depsMap.set(property, (deps = new Set()))
    }
    deps.add(activeEffect);
}
function trigger(target, property) {
    const depsMap = bucket.get(target);
    if (!depsMap) return;
    const effects = depsMap.get(property);
    effects && effects.forEach(fn => fn())
}
function effect(fn) {
    activeEffect = fn;
    fn()
}
effect(
    () => {
        box.innerText = obj.test;
    }
);
```

## 分支切换

在分支切换过程中，也就是三元表达式中，如果被判断的值为false，则前面判断为true会触发的值永远不会触发，但它的改变依旧会影响副作用函数，所以我们考虑重写副作用函数

```js
function effect(fn) {
    const effectFn = () => {
        cleanup(effectFn)
        activeEffect = effectFn
        fn()
    }
    effectFn.deps = [];
    effectFn()
}
function cleanup(effectFn) {
    for(let i = 0; i < effectFn.deps.length; i++) {
        const deps = effectFn.deps[i];
        deps.delete(effectFn);
    }
    effectFn.deps.length = 0;
}
```

