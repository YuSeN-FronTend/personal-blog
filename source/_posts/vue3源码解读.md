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

# 渲染器

顾名思义渲染器就是将拼接好的dom字符串动态的渲染到页面上，下面是一个简单的渲染函数实现

```js
let app = document.getElementById('app');
let domStr = '<div>123</div>'
renderer(domStr, app);
function renderer(domString, container) {
    container.innerHTML = domString;
}
```

## 和响应式做结合

由于响应式我们在上文中已经实现过，所以使用@vue/reactivity来实现，如下

```html
<script src="https://unpkg.com/@vue/reactivity@3.0.5/dist/reactivity.global.js"></script>
    <script>
        const { effect, ref } = VueReactivity;
        function renderer(domString, container) {
                container.innerHTML = domString;
            }
        const count = ref(1);
        effect(() => {
            renderer(`<h1>${count.value}</h1>`,document.getElementById('app'))
        })
        count.value++;
    </script>
```

这样就完成了基本的结合

## 更新render

如果多次调用render，则会更新节点，称之为打补丁，如下

```js
        const { effect, ref } = VueReactivity;
        function createRenderer() {
            function render(vnode, container) {
                if(vnode) {
                    patch(container._vnode, vnode, container);
                } else {
                    if(container._vnode) {
                        container.innerHTML = "";
                    }
                }
                container._vnode = vnode
            }
            return {
                render
            }
        }
        const renderer = createRenderer();
        renderer.render(vnode1, document.getElementById('app'))
        renderer.render(vnode2, document.getElementById('app'))
        renderer.render(null, document.getElementById('app'))
```

在第一次渲染中，vnode1会被渲染为真实DOM,并且将vnode1变为旧vnode1，第二次渲染旧vnode存在，则将会把两个vnode都传入patch进行补丁，再把vnode2当作旧vnode，第三次渲染中由于传入空值，则执行清空容器的操作，但此时的操作存在些问题，后续会做响应修改，并且我们还没有编写patch函数，这就来

## patch和挂载函数的简单实现

```js
function createRenderer(options) {
            const { createElement, setElementText, insert } = options;
            function mountElement(vnode, container) {
                const el = createElement(vnode.type);
                if(typeof vnode.children === 'string') {
                    setElementText(el, vnode.children)
                }
                insert(el, container);
            }
            function patch(n1,n2, container) {
                if(!n1) {
                    // 若n1不存在，则直接挂载
                    mountElement(n2, container)
                } else {
                    // 若存在即为打补丁，后续再说
                }
            }
            function render(vnode, container) {
                if(vnode) {
                    patch(container._vnode, vnode, container);
                } else {
                    if(container._vnode) {
                        container.innerHTML = "";
                    }
                }
                container._vnode = vnode
            }
            return {
                render
            }
        }
const renderer = createRenderer({
            // 用于创建元素
            createElement(tag) {
                return document.createElement(tag);
            },
            // 用于设置文本节点
            setElementText(el, text) {
                el.textContent = text;
            },
            // 用于在给定的parent下添加指定元素
            insert(el, parent, anchor = null) {
                parent.insertBefore(el, anchor)
            }
        });
```

不仅实现了两个函数，还将dom上的api封装好并且在创建函数的时候当作参数传入，这样可以使代码维护性更强，代码复用率也更高

## patch函数children为数组的情况处理

需要增设一个判断是否为数组，具体代码如下

```js
if(typeof vnode.children === 'string') {
      setElementText(el, vnode.children)
} else if(Array.isArray(vnode.children)) {
      vnode.children.forEach(child => {
          patch(null, child, el)
	  })
}
```

## 为节点添加属性的简单实现

直接使用setAttribute方法添加属性

```js
// 如果vnode.props存在就去处理它
                if(vnode.props) {
                    // 遍历 vnode.props
                    for(const key in vnode.props) {
                        // 调用setAttribute将属性设置到元素上
                        el.setAttribute(key, vnode.props[key])
                    }
                }
```

这只是比较基础的方法，一定会存在很多的缺陷

## 理解HTML Attributes与 DOM Properties

HTML Attributes就是如下标签中所存在的属性如下

```html
<input id="my-input" type="text" value="foo" />
```

DOM Properties是通过获取dom元素，然后它所存在存在的属性，虽然有很多属性和HTML Attributes是重名的，但它们的名字不总是一模一样的

所以原则就是HTML Attributes的作用是设置与之对应的DOM Properties的初始值

根据此规则处理后的代码如下

```js
function createRenderer(options) {
            const { createElement, setElementText, insert, patchProps } = options;
            function mountElement(vnode, container) {
                const el = createElement(vnode.type);
                // 判断children类型来判断如何处理
                if(typeof vnode.children === 'string') {
                    setElementText(el, vnode.children)
                } else if(Array.isArray(vnode.children)) {
                    console.log(Object.prototype.toString.call(vnode.children).slice(8, -1));
                    vnode.children.forEach(child => {
                        patch(null, child, el)
                    })
                }
                // 如果vnode.props存在就去处理它
                if(vnode.props) {
                    // 遍历 vnode.props
                    for(const key in vnode.props) {
                        patchProps(el, key, null, vnode.props[key])
                    }
                }
                insert(el, container);
            }
            function patch(n1,n2, container) {
                if(!n1) {
                    // 若n1不存在，则直接挂载
                    mountElement(n2, container)
                } else {
                    // 若存在即为打补丁，后续再说
                }
            }
            function render(vnode, container) {
                if(vnode) {
                    patch(container._vnode, vnode, container);
                } else {
                    if(container._vnode) {
                        container.innerHTML = "";
                    }
                }
                container._vnode = vnode
            }
            return {
                render,
            }
        }
        const renderer = createRenderer({
            // 用于创建元素
            createElement(tag) {
                return document.createElement(tag);
            },
            // 用于设置文本节点
            setElementText(el, text) {
                el.textContent = text;
            },
            // 用于在给定的parent下添加指定元素
            insert(el, parent, anchor = null) {
                parent.insertBefore(el, anchor)
            },
            // 将属性设置相关操作封装到patchProps函数中，并作为渲染器选项传递
            patchProps(el, key, prevValue, nextValue) {
                function shouldSetAsProps(el, key, value) {
                    // 特殊处理
                    if (key === 'form' && el.tagName === 'INPUT') {
                        return false;
                    }
                    // 兜底
                    return key in el;
                }
                if (shouldSetAsProps(el, key, nextValue)) {
                    const type = typeof el[key];
                    // 如果是布尔类型并且value是空字符串，则将值矫正为true
                    if (type === 'boolean' && nextValue === '') {
                        el[key] = true
                    } else {
                        el[key] = nextValue
                    }
                } else {
                    // 如果要设置的属性没有对应的DOMProperties， 则使用setAttribute函数设置属性
                    el.setAttribute(key, nextValue)
                }
            }
        });
```

## class的处理

挂载class有三种方法，setAttribute，className，classList，经过性能测试发现className渲染的性能是最优的，所以我们需要将setAttribute改成className即可。

## 卸载操作

我们看完了挂载，就要响应的看一下卸载操作，由于前面提到直接将innerHTML清空来完成卸载是不在合适的，原因有如下三点：

- 容器的内容可能由某个或多个组件渲染，当卸载操作发生时，应该正确的调用这些组件的beforeUnmount、unmounted等生命周期函数
- 即使内容不是由组件渲染的，有的元素存在自定义指令，我们应该在卸载操作发生时正确执行对应的指令钩子函数
- 使用innerHTML清空容器元素内容的另一个缺陷时，它不会移除绑定在DOM元素上的事件处理函数

```js
function unmount(vnode) {
                const parent = vnode.parentNode;
                if(parent) {
                    parent.removeChild(vnode.el);
                }
            }
```

使用removeChild来卸载即可

## 添加事件

添加事件只需截取添加addEventListener即可，但是还要注意移除事件，为了性能更好，则需要使用一个invoker伪造事件函数，具体代码如下

```js
if(/^on/.test(key)) {
                    let invoker = el._vei
                    const name = key.slice(2).toLowerCase()
                    if(nextValue) {
                        if(!invoker) {
                            // 如果没有，则伪造一个
                            invoker = el._vei = (e) => {
                                invoker.value(e);
                            }
                            // 将真正的事件处理函数赋值给 invoker.value
                            invoker.value = nextValue// 绑定invoker作为事件处理函数
                            el.addEventListener(name, invoker);
                        } else {
                            // 存在意味着更新
                            invoker.value = nextValue;
                        }
                    } else if(invoker) {
                        // 心得事件绑定函数不存在，且之前绑定的invoker存在，则移除绑定
                        el.removeEventListener(name, invoker)
                    }
                }
```

## 解决事件冒泡问题

```js
01 const { effect, ref } = VueReactivity
02
03 const bol = ref(false)
04
05 effect(() => {
06   // 创建 vnode
07   const vnode = {
08     type: 'div',
09     props: bol.value ? {
10       onClick: () => {
11         alert('父元素 clicked')
12       }
13     } : {},
14     children: [
15       {
16         type: 'p',
17         props: {
18           onClick: () => {
19             bol.value = true
20           }
21         },
22         children: 'text'
23       }
24     ]
25   }
26   // 渲染 vnode
27   renderer.render(vnode, document.querySelector('#app'))
28 })
```

如果我们按照上述添加代码，则会发现已开始bol的值为false，div的props属性为空，则没有点击事件，但我们点击p标签之后，更改了bol的值，由于bol为响应式的，会触发副作用函数导致父元素div的点击事件会执行，那么我们可以调整patchProps函数关于事件的代码，屏蔽所有绑定时间晚于事件触发时间的事件处理函数的执行，如下代码

```js
invoker = el._vei[key] = (e) => {
    // e.timeStamp是事件发生的时间
    // 如果事件发生的事件早于事件处理函数绑定的时间，则不执行事件处理函数
    if(e.timeStamp < invoker.attached) return;
    if(Array.isArray(invoker.value)){
        invoker.value.forEach(fn => fn(e));
    } else {
        // 直接作为函数调用
        invoker.value(e);
    }
}
// 将真正的事件处理函数赋值给 invoker.value
invoker.value = nextValue// 绑定invoker作为事件处理函数
// 添加invoker.attached属性，存储事件处理函数被绑定的时间
invoker.attached = performance.now();
el.addEventListener(name, invoker);
```

## 更新子节点(还没包含diff算法)

```js
           function patchChildren(n1,n2,container) {
                // 判断新子节点的类型是否是文本节点
                if(typeof n2.children === 'string') {
                    // 旧子节点一共有三种可能：没有子节点、文本子节点、一组子节点
                    // 只有当旧子节点为一组子节点的时候，才需要逐个卸载，其他时候不用做操作
                    if(Array.isArray(n1.children)) {
                        n1.children.forEach((c) => unmount(c));
                    }
                    // 最后将心得文本节点内容设置给容器元素
                    setElementText(container, n2.children)
                } else if(Array.isArray(n2.children)){
                    // 此时说明新子节点是一组子节点
                    // 判断旧子节点是否也是一组子节点
                    if(Array.isArray(n1.children)) {
                        // 代码到这说明新旧子节点都是一组子节点,这就到了核心diff算法
                        // 由于还没有学习diff算法,使用一种傻瓜方式即可,即删除原来的一组子节点,再挂载新的一组子节点
                        n1.children.forEach(c => unmount(c))
                        n2.children.forEach(c => patch(null, c, container))
                    } else {
                        // 此时的旧子节点要么是文本子节点,要么不存在
                        // 无论哪种,我们都只需要将容器清空然后将心得一组子节点逐个挂载
                        setElementText(container, "")
                        n2.children.forEach(c => patch(null, c, container))
                    }
                }else{
                    // 代码运行到这里证明新子节点不存在
                    // 如果旧子节点是一组子节点,逐个卸载即可
                    if(Array.isArray(n1.children)) {
                        n1.children.forEach(c => unmount(c));
                    } else if(typeof n1.children === 'string') {
                        // 旧子节点是文本子节点,清空内容即可
                        setElementText(container, '')
                    }
                    // 如果旧子节点也没有,则不需要做
                }
            }
            function patchElement(n1,n2) {
                const el = n2.el = n1.el;
                const oldProps = n1.props;
                const newProps = n2.props;
                // 第一步：更新props
                for(const key in newProps) {
                    if(newProps[key] !== oldProps[key]) {
                        patchProps(el, key, oldProps[key], newProps[key])
                    }
                }
                for(const key in oldProps) {
                    if(!(key in newProps)) {
                        patchProps(el, key, oldProps[key], null)
                    }
                }
                // 第二步：更新children
                patchChildren(n1, n2, el);
            }
```

## Fragment标签

```js
// 处理Fragment类型vnode
                    if(!n1) {
                        // 如果旧vnode存在,只需要将Fragment的children逐个挂载即可
                        n2.children.forEach(c => patch(null, c, container))
                    } else {
                        // 如果旧vnode存在,则只需要更新Fragment的children
                        patchChildren(n1,n2,container)
                    }
```

由于Fragment标签本身不渲染任何内容，之渲染子标签，实现逻辑比较简单。再vue2中实现递归导航栏由于需要有一个根节点，设置成div会出现缩进导航栏导航栏文字图标不消失问题，当时需要安装vue-fragment插件，模拟根节点解决问题，而vue3中引入了fragment并且完美的解决了这个问题

## 简单diff算法

diff算法核心思想就是最小量更新。

- 首先就是当标签没变，里面的内容有改变时，一定要保留标签，只改变内容。
- 如果发生移动，则一定要根据key值来判断是否有可以复用的标签，并且查看是否新节点的text改变了
- 通过索引方式可以找出移动，如果新节点相比于旧节点是递增索引，则不需要移动，反之则需要移动。也就是再旧children中寻找具有相同key值得节点的过程中，遇到的最大索引值，只需定义一个最大索引变量，遍历的同时更新变量比较即可
- 移动时需要获取新节点中当前节点的前一个虚拟节点，然后使用insert完成节点的移动即可
- 添加元素就是根据上述方法搜索出新节点比旧节点多出了那个元素，并且拿到新节点中需要被新增节点的前一个节点也就是prevVnode，然后插入即可
- 删除元素则更清晰，当更新结束也就是新节点遍历完成之后，需要查找就节点是否存在与新节点key值相同的元素，如果没找到，则删除该元素即可

## 双端diff算法

和简单diff算法不同的是，简单diff算法只靠索引来更换位置，而双端diff算法就是我们常见的四指针，也就是vue2diff算法中的核心具体实现在之前学习vue2源码中已经记录过，不再重复记录了。

## 快速diff算法

快速diff算法和双端不同的是他会做前置节点更新和后置节点更新，这样可以极大的提升性能

- 前置节点更新就是使用while循环判断新旧节点的平行节点是否相同，遇到不相同则停止
- 后置节点更新和前置节点更新类似，只不过是从后面向上循环的
- 判断完后如果新节点中剩余节点，旧节点已经没了，则新增节点即可，也就是找到对应锚点新增即可
- 如果判断完旧节点中剩余节点，新节点已经没了，则需要删除节点，剩余哪个删除那个即可
- 如若遇到比较复杂的情况，则需要进行上述比较完之后，计算新数组的长度并且创建一个source数组并且如果旧子节点中存在相同元素则添加对应索引，如果没有则添加-1，但如此需要for循环中嵌套for来完成，时间复杂度为n²。所以我们想到可以给新子节点建立一个索引表，key为新子节点对应的key，value为新子节点的索引，然后旧子节点去新子节点寻找是否存在和自身的key相同的节点，如果有就把自己添加到source，如果没有则将自己卸载即可，如此一来时间复杂度会变为n，虽然还是两个for循环，但是不用嵌套了
- source填充好之后，就要移动了，但需要判断如果已经更新过的节点数应该小于新的一组子节点中需要更新的节点数，如果大于则说明有多余的节点，应该将其卸载。移动是需要使用的是依靠source最长递增子序列，它是非常重要的，因为这个最长递增子序列说明所代表的元素是不需要移动的，然后需要创建两个索引值，一个指向新子节点的最后一个节点，另一个指向最长递增子序列的最后一个元素，然后判断i !==seq[s],，如若不相等，则说明该节点对应的真实DOM需要移动，否则说明当前项不需要移动。

梳理： 如若遇到复杂情况，则需要进行一开始的比较，然后进行找出通过比较新旧节点的key值来填充source数字，然后建立两个索引，一个指向最长子序列最后一项，一个指向剩余新子节点的最后一项，然后开始判断一下两步

- source[s]是否为-1，如果不是，则说明不用更新，只需打补丁即可，然后走一下判断
- i !== seq[s]，如果成立则说明需要移动，不成立则不需要移动

经过判断后，需要新增的新增即可，需要移动的移动到指定位置即可，至此快速diff算法完毕

## 三种重要的内置组件标签

- keep-alive

  缓存组件功能，在切换组件时，可以使用此标签在切换的过程中不需要重新渲染页面，直接把当前dom结构移到另一个容器中暂且保管，里面还有include，exclude两个属性，还能设置最大缓存区域，会优先缓存最近访问的内容。

- teleport

  将包裹的dom可以移动到任意位置，书写dialog模态框时特别的常见，因为需要遮罩当前页面，移动到body即可。具体实现是通过渲染时选择to属性传入的元素然后挂载指定位置即可

- transition

  包裹的dom切换时可以定义动画，常用是套router-view，具体实现是通过在属性中定义几个css可操作的名字，然后根据规则设置过渡动画即可

## 同构渲染

即服务端渲染(SSR)和CSR(客户端渲染)同时进行。

- 因为SSR更加支持SEO(搜索引擎优化)，所以需要使用这部分就会采用SSR
- 首屏加载CSR可能会白屏，我们就使用SSR
- 在切换页面的时候，由于SSR会每次向服务端请求，所以速度较慢，体验不好，使用CSR
- 两者一起使用对于服务端的资源占用来说，一定比单独使用SSR一种要少

