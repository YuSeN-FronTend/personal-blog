---
title: vue部分面试总结
date: 2023-3-29 8:54
categories: 面试
---

# vue和react的区别

## 数据可变性

- react推崇函数式编程，数据不可变以及单项数据流，只能通过`setState`和`onchange`来实现视图更新
- vue基于数据可变，设计了响应式数据，通过监听数据的变化自动更新视图

## 写法

- react推荐使用jsx+inline style的形式，也就是把html和css全写进js中
- vue是单文件组件(SFC)形式，在一个组件内分模块(template/script/style)，当然vue也支持jsx形式，可以在开发vue的ui组件库时使用

## diff算法

- vue2采用双端比较，vue3采用快速比较
- react主要使用diff队列保存需要更新哪些DOM，得到patch树，再统一操作批量更新DOM。需要使用shouldComponentUpdate()来手动优化react的渲染

# vue组件通信方式

- props/$emit
- ref/$refs
- parent/root
- attrs/listeners
- eventBus/vuex/pinia/localStorage/sessionStorage/Cookie/window
- provide/inject

# vue渲染列表为什么要加上key

v-for遍历的时候，绑定key，可以让DOM识别到每个节点，最大程度实现对已有节点的复用，从而提升性能，减少DOM操作的性能开销。

如果v-for遍历常量或者子节点时诸如纯文本这类没有状态的节点，是可以不加key的。但实际的开发中还是推荐统一都加上key，能够实现更广泛场景的同时，避免可能发生的状态更新错误。

# vue3相对vue2的响应式优化

vue2使用的是`Object.defineProperty`去监听对象属性值的变化，但是它不能监听对象属性的新增和删除，所以需要使用`$set`，`$delete`这种语法糖去实现，这是一种设计上的不足。

vue3使用了`Proxy`去实现响应式监听对象的增删改查。从api的原生性能上`Proxy`是比`Object.defineProperty`要查的。而vue做的响应式性能优化主要是将嵌套层级深得对象变成响应式这一过程。

vue2的做法是在组件初始化的时候就递归执行`Object.defineProperty`把子对象变成响应式。而vue3是在访问到子对象属性时，才会把他转化为响应式，。这种延时定义子对象响应式对性能有一定提升。

## vue核心diff流程

当同类型的vnode的子节点都是一组节点(数据类型)的时候，会走diff流程

## vue3是快速选择算法

- 同步头部节点
- 同步尾部节点
- 新增新的节点
- 删除多余节点
- 处理未知子序列（贪心 + 二分处理最长递增子序列）

## vue2是双端比较算法

在新旧子节点的头尾节点，也就是四个节点之间进行对比，找到可复用的节点，不断向中间靠拢的过程

## diff算法的目的

diff算法就是为了尽可能地复用节点，减少DOM频繁创建和删除带来的性能开销

# vue双向绑定原理

基于MVVM模型，完成**数据变化后更新视图**和**视图变化后更新数据**这样一个功能，就是传统意义上的双向绑定

## vue2实现双向绑定

核心：`Observer`监听器，`Watcher`订阅者和`Compile`编译器

首先监听器会监听所有的响应式对象属性，编译器会将模板进行编译，找到里面动态绑定的响应式数据并初始化视图；`watcher`回去收集这些依赖；当响应式数据发生变更时，`Observer`就会通知`watcher`；`watcher`接收到监听器的信号就会执行更新函数去更新视图

## vue3实现双向绑定

vue3的变更时数据劫持部分使用了Proxy替代`Object.defineProperty`，收集的依赖使用组件的副作用渲染函数替代watcher

# v-model原理

v-model是用来监听用户事件然后更新数据的语法糖

本质上还是单项数据流，内部通过绑定元素的value值向下传递数据，然后通过绑定input事件，向上接收并处理更新数据

vue3和vue2实现基本一致

# vue响应式原理

 无论是vue2还是vue3，响应式的核心就是观察者模式 + 劫持数据的变化，在访问的时候做依赖收集和在修改数据的时候执行收集的依赖并更新数据，具体如下：

**`vue2`**采用的是`Object.defineProperty`劫持对象的get和set方法，每个组件实例都会在渲染时初始化一个watcher实例，它会将组件渲染过程中所接触的响应式变量记为依赖，并且保存了组件的更新方法update。当以来的setter触发时，会通知watcher触发组件的update方法，从而更新视图。

**`vue3`**使用的时ES6的proxy，proxy不仅能追踪属性的获取和修改，还可以追踪对象的增删，这在vue2中需要set/delete才会实现。然后就是收集的依赖是用组件的副作用渲染函数替代watcher实例。

# computed和watch

computed的大体实现和普通的响应式数据是一致的，只不过添加了延迟计算和缓存功能

`watchEffect`会自动收集回调函数响应式变量的依赖，并在首次自动执行。推荐在大部分时候用`watch`显式的指定依赖以避免不必要的重复触发，也避免在后续代码修改或重构时不小心引入新的依赖。`watchEffect`适用于一些逻辑相对简单，依赖源和逻辑强相关的场景

# $nextTick原理

vue有个机制，更新DOM是异步执行的，当数据变化会产生一个异步更新队列，要等异步队列结束后才会统一进行更新视图，所以改了数据之后立即去拿DOM还没更新就会拿不到最新数据。所以提供了一个`$nextTick`，它的回调函数会在DOM更新后立即执行

`nextTick`本质上是个异步任务，由于事件循环机制，异步任务的回调总会在同步任务执行完成后才得到执行。所以源码实现就是根据环境创建异步函数比如Promise.then(浏览器不支持promise就会用MutationObserver，浏览器不支持MutationObserver就会用setTimeout)，然后调用异步函数执行回调队列。

所以项目中不使用$nextTick的话也可以直接使用Promise.then或者SetTimeout实现相同的效果

# vue异常处理

## 1、全局错误处理：`Vue.config.errorHandler`

`Vue.config.errorHandler = function(err, vm, info) {};`

如果组件在渲染时出现错误，错误将会被传递至全局`Vue.config.errorHandler`  配置函数(如果已设置)

比如前端监控领域的sentry, 就是利用这个钩子函数进行的vue相关异常捕捉处理

`main.js`中添加代码

```js
Vue.config.errorHandler = function(msg, vm, trace) {
  console.log(msg, 'msg');
  console.log(vm, 'vm');
  console.log(trace, 'trace');
}
```

跟路由中向mounted中添加如下代码

```js
 const err = new Error('error');
 throw err;
```

运行项目即可看到控制台的打印

## 2、全局警告处理：`Vue.config.warnHandler`

用法和上一个上不多，但需要注意的是此处理仅在**开发环境**中生效

如果在模板中引用一个没有定义的变量，它就会触发

## 3、单个vue实例错误处理：`renderError`

```js
const app = new Vue({
	el: '#app',
	renderError(h, err){
		return h("pre", { style: { color: 'red' }}, err.stack)
	}
})
```

和组件相关，只适用于开发环境，这个用处不是很大，不如直接看控制台

## 4、子孙组件错误处理：`errorCaptured`

```js
Vue.component("cat", {
	template: `<div><slot></slot></div>`,
	props: { name: {  type: string} },
	errorCaptured(err, vm, info) {
		console.log(`cat EC: ${err.toString()}\ninfo: ${info}`);
        return false;
	}
})
```

注意：只能在组件内部使用，用于捕获子孙组件的错误，一般可以用于组件开发过程中的错误处理

## 5、终极错误捕获：`window.onerror`

`window.onerror = function(message, source, line, column, error) {};`

它是一个全局的异常处理函数，可以抓取所有的js异常

# Vuex流程&原理

Vuex利用vue中的mixin机制，在beforeCreate钩子前混入了vuexinit方法，这个方法实现了将store注入了vue实例当中，并注册了store的引用属性store，所以可以使用`this.store.xxx`去引入vuex中定义的内容

state是利用vue的data，通过`new Vue({data: {$$state: state}})`将state转换成响应式对象，然后使用computed函数实时计算getter

# Vue.use函数里面具体做了什么事

可以通过全局方法`Vue.use()`注册插件，并能组织多次注册相同插件，它需要在`new Vue`之前使用

该方法第一个参数必须是`Object`或`Function`类型的参数。如果是`Object`那么该对象需要定义一个`install`方法，如果是`Function`那么这个函数就被当作`install`方法

`Vue.use()`执行就是执行`install`方法，其他传参会作为`install`方法的参数执行

所以`Vue.use()`本质就是执行需要注入插件的`install`方法

源码实现

```js
    Vue.use = function(plugin: Function | Object) {
        const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
        // 避免重复注册
        if(installedPlugins.indexOf(plugin) > -1) {
            return this
        }     
        // 获取传入的第一个参数
        const args = toArray(arguments, 1);
        args.unshift(this);
        if(typeof plugin.install === 'function') {
            // 如果传入对象中的install属性是个函数则直接执行
            plugin.install.apply(plugin, args)
        } else if (typeof plugin === 'function') {
            // 如果传入的是函数，则直接(作为install方法)执行
            plugin.apply(null, args)
        }
        // 将已经注册的插件推入全局installedPlugins中
        installedPlugins.push(plugin);
        return this
    }
}
```

使用方式

```js
installedPlugins import Vue from 'vue'
import Element from 'element-ui'
Vue.use(Element)
```

# 怎么编写一个vue插件

要暴露一个`install`方法，第一个参数是Vue构造器，第二个参数是一个可选的配置项对象

```js
MyPlugin.install = function(Vue, options = {}) {
    // 1、添加全局方法或属性
    Vue.myGlobalMethod = function() {}
    // 2、添加全局服务
    Vue.directive('my-directive', {
        bind(el, binding, vnode, pldVnode){}
    })
    // 3、注入组件选项
    Vue.mixin({
        created: function() {}
    })
    // 4、添加实例方法
    Vue.prototype.$myMethod = function(methodOptions) {}
}
```

# v-permission属性如何使用

- 在src文件夹下创建directives文件夹

- 在directives文件夹下创建permission.js来实现指令作用

  ```js
  export default {
      // 接入两个参数，使用此指令的元素和内置属性
      inserted(el, bindling){
          // 如果有内置属性
          if(bindling.name){
              let per = ['add', 'delete']
              // 寻找绑定的指令是否存在权限
              let perb = per.some(item => {
                  return item === bindling.value;
              })
              // 如果没有则隐藏该属性
              if(!perb) {
                  el.style.display = 'none'
              }
          }
      }
  }
  ```

- 在directives文件夹下创建index.js来集成所有指令并注册自定义指令

  ```js
  import permission from '@/directives/permission'
  
  // 创建自定义指令对象
  const directives = {
      permission
  }
  
  export default {
      install(Vue) {
          // 遍历所有指令对象并注册
          Object.keys(directives).forEach((key) => {
              // 注册自定义指令
              Vue.directive(key, directives[key])
          })
      }
  }
  ```

- 最后在main.js中添加引用文件代码

  ```js
  import Directive from './directives/index'
  Vue.use(Directive);
  ```

至此就可以在模板中使用v-permission指令了

# vue中部分指令之间的区别

## v-model和:value的区别

两者在给表单类赋值时都可以实现双向绑定，但是v-model里面不可以加入表达式，只可以绑定一个值，但:value可以绑定表达式比如`:value="count += 1"`

## v-model和v-bind的区别

- v-model是双向绑定，数据可以从data流向页面，也可以从页面流向data
- v-bind是单向绑定，用来绑定数据和属性或者表达式，但是只能从data流向页面

v-bind可以给任何属性赋值，而v-model只能给表单类属性赋值，如input，radio，checkbox，selected等，因为v-model就是要捕获用户选择或者填写的value值

## v-if和v-show的区别

- v-if会直接将值为false的dom元素从模板中删除，更新的时候需要重新渲染页面，并且还可以v-else-if这种多判断，但是结尾必须要有v-else
- v-show只是将dom元素添加display: none;属性，更新的时候不会重新渲染页面
