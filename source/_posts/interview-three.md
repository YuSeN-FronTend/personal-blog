---
title: 面经总结(3)
date: 2023-5-11 11:09
categories: 面试
---
# 1、vue3性能提升主要体现在哪些方面

## (1)响应式系统

- vue2

  基于Object.defineProperty()来进行数据接触，然后深度遍历每个属性，给每个属性添加getter和setter方法

- vue3

  使用Proxy(代理)对象重写响应式系统

  - 可以监听动态新增的属性
  - 可以监听删除的属性
  - 可以监听数组的索引和length属性

  实现原理就是用Proxy拦截对象中任意属性的变化，再通过Reflect对源对象属性进行操作

## (2)编译阶段

- vue2

  通过标记静态节点优化diff的过程

- vue3

  - 标记和提升所有的静态节点，diff的时候只需要对比动态节点的内容
  - Fragment：template中不需要唯一根节点，可以直接放文本或者同级标签
  - 静态提升，当使用hoisStatic时，所有静态的节点都会被提升到render方法之外，只会在应用启动的时候被创建一次，之后使用只需要应用提取的静态节点，随着每次的渲染被不停的复用
  - patch flag，在动态标签末尾加上相应的标记，只能带patchFlag的节点才被认为是动态的元素，会被追踪属性的修改，能快速的找到动态节点，而不用逐个逐层遍历，提高了虚拟dom diff算法的性能
  - 缓存事件处理函数cacheHandler，避免每次触发都要重新生成全新的function去更新之前的函数

## (3)源码体积

- 相比Vue2，vue3整体体积变小了，移除了一些不常用的API

- tree shanking
  - 任何一个函数，如ref，reavtived，computed等，仅仅在用到的时候才打包
  - 通过编译阶段的静态分析，找到没有引入的模块并打上标记，将这些模块都给摇掉

# 2、vue有哪些新组建

## (1)Fragment

- 在vue2中必须有一个根标签
- 在vue3中：组件可以没有根标签，内部会将多个标签包含在一个Fragment虚拟元素中

这样做的好处是：**减少标签层级，减小内存占用**

## (2)Teleport

此标签可以将组件的html结构移动到指定位置

**场景**：如果想要点击按钮，打开一个模态框，按钮控制模态框需要在同一个页面中书写逻辑，但模态框需要遮罩整个页面，这里就用到了此标签

```html
<teleport to="移动位置">
    <div v-if="isShow" class="mask">
        <div class="dialog">
            <h3>我是一个弹窗</h3>
            <button @click="isShow = false">关闭弹窗</button>
        </div>
    </div>
</teleport>
```

## (3)Suspense(仍在测试阶段)

等待异步组件时渲染一些额外的内容，让应用拥有更好的用户体验

# 3、vue2和vue3有什么区别

1. 响应式系统的重新配置，用proxy替换了Object.defineProperty
2. typeScript支持
3. 新增组合API，更好的逻辑重用和代码组织
4. v-if和v-for的优先级
5. 静态元素提升
6. 虚拟节点静态标记
7. 生命周期变化
8. 打包提及优化
9. ssr渲染性能提升
10. 支持多个根节点

# 4、vue生命周期

vue3提供了composition API形式的生命周期钩子，和vue2中钩子对应如下：

- `beforeCreate` ===> `setup()`
- `created` ===> `setup()`
- `beforeMount` ===> `onBeforeMount`
- `mounted` ===> `onMounted`
- `beforeUpdate` ===> `onBeforeUpdate` 
- `updated` ===> `onUpdated`
- `beforeDestroy` ===> `onBeforeUnmount`
- `destroyed` ===> `onUnmounted`

# 5、Composition API与Option API有什么不同

## (1)Options API

也就是常说的选项式API(vue2中的形式)，以`.vue`为后缀的文件，通过定义methods，computed，watch，data等属性与方法，共同处理页面逻辑。

但是当组件变得复杂的时候，导致对应属性的列表也会增长，这可能会导致组件难以阅读和理解

## (2)Composition API

就是vue3中的组合式API，组件根据逻辑功能来组织，一个功能所定义的所有API回放在一起(更加的高内聚，低耦合)

即使项目很大，功能很多，我们也可以快速的定位到这个功能所用到的所有API

## (3)对比

针对`Composition API`与`Options API`进行两大方面的比较

- 逻辑组织
- 逻辑复用

### 逻辑组织

#### option API

如果组件是一个大型的组件，内部有很多处理逻辑关注点。选项的分离掩盖了潜在的逻辑问题。此外，在处理单个逻辑关注点时，我们必须不断地跳转相关代码的选项块

#### compostion API

使用此方法正解决上述问题，将某个逻辑关注点相关代码都放在一个函数里，这样当需要修改一个功能时直接在这个区域修改即可

### 逻辑复用

在vue2中我们使用mixin进行逻辑复用，现在可以用compostion API解决，在vue3学习章节有系统学习过，不多做赘述

### 小结

- 在逻辑组织和逻辑复用方面，`composition API`更好
- 因为 `composition API`几乎是函数，会有更好的类型判断
- `composition API`对tree-shaking友好，代码也更容易压缩
- `composition API`中见不到this使用，减少了this指向不明的情况
- 如果是小型组件，可以继续使用`option API`，也是很好的

# 6、什么是SPA单页面应用，首屏加载如何优化

单页面web应用(single page web application, SPA)，就是只有一张Web页面的应用，是加载单个HTML页面并在用户与应用程序交互时动态更新该页面的web应用程序

## SPA首屏优化方式

- 减少入口文件积
- 静态资源本地缓存
- UI框架按需加载
- 图片资源打包压缩
- 组件重复打包
- 开启GZip压缩
- 使用SSR

# 7、对vue项目做过哪些性能优化

## (1)v-if 和 v-show

- 频繁的切换使用v-show，利用其缓存特性
- 首屏渲染时使用v-if，如果为false则不进行渲染

## (2)v-for的key

- 列表变化时，循环时使用唯一不变的key，结合diff算法完成复用策略
- 列表只进行一次渲染时，key可以采用循环的index

## (3)侦听器和计算属性

- 侦听器watch用于数据变化时引起其他行为
- computed计算属性就是新计算带来的属性，如果依赖的数据不发生变化，不会触发重新计算

## (4)合理使用生命周期

- 在destroyed阶段进行绑定事件或者定时器的销毁
- 使用动态组件的时候通过keep-alive包裹进行缓存处理，相关的操作可以在acived阶段激活

## (5)数据响应式处理

- 不需要响应式处理的数据可以通过Object.freeze处理
- 需要响应式处理的属性可以通过this.$set

## (6)路由加载方式

- 路由懒加载
- 页面组件可以采用异步加载方式

## (7)插件引入

- 第三方插件可以采用按需加载的方法，比如`element-ui`

## (8)减少代码量

- 采用mixin(vue3中用响应式API)抽离公共方法实现代码复用
- 抽离公共组件
- 定义公共方法至公共js中
- 抽离公共css

## (9)编译方式

- 如果线上需要template的编译，可以采用完成版vue.esm.js
- 如果线上无需template的编译，可采用运行时版本vue.runtime.esm.js，相比完整版体积要小大约'30%'

## (10)渲染方式

- 服务端渲染，如果需要SEO的网站可以采用服务端渲染的方式
- 前端渲染，一些企业内部使用的后端管理系统可以采用前端渲染的方式

## (11)字体图标的使用

- 有些图片图标尽可能使用字体图标

# 8、vue组件通信的方式

vue中常规情况通信有8种

- 通过props传递
- 通过$emit触发自定义事件
- 使用ref
- EventBus
- $parent或$root
- attrs与listeners
- Provide与Inject
- Vuex

组件间通信的分类可以分成以下

- 父子关系组件传奇选择props与$emit进行传递，也可以使用ref
- 兄弟关系的组件传递可以选择$bus，其次可以选择$parent进行传递
- 祖先与后代组件数据传递可选择attrs与listener或者Provide与Inject
- 复杂关系的组件数据传递可以通过vuex存放共享的变量

# 9、vue常用的修饰符有哪些

## (1)表单修饰符

### ①.lazy

在默认情况下v-model在每次input事件触发后将输入框的值与数据进行同步，可以添加lazy修饰符从而传为在change事件触发后进行同步

### ②.number

如果想自动将用户的传入值转为数值类型，可以给v-model添加number修饰符

### ③.trim

自动过滤到用户输入的首位空格

## (2)事件修饰符

### ①.stop

阻止单击事件继续传播

### ②.prevent

阻止标签的默认行为

### ③.capture

事件先在有此是修饰符的节点上触发，然后在其包裹的内部节点中触发

### ④.self

只当在event.target时当前元素自身时触发处理函数，即事件不是从内部元素触发的

### ⑤.once

表示当前事件只触发一次

### ⑥.passive

尤其能够提升移动端的性能

# 10、vue中的$nextTick有什么作用

待当前的DOM渲染完毕之后再执行nextTick回调函数的逻辑。

- 有时需要根据动态的为页面某些dom元素添加事件，这就是要求在DOM元素完毕时去设置，这种场景就会需要$nextTick
- 再使用第三方插件时，希望在vue生成的某些dom动态发生变化时重新应用该插件，也会用到该方法

总的来说$nextTick的核心就是使用Promise、mutationObserver、setImmediate、setTimeout等原生JavaScript方法来模拟执行相应的微/宏任务

# 11、如何理解双向数据绑定

vue就是双向绑定的框架，双向绑定由三个重要部分构成：

- 数据层(Model)：应用的数据及业务逻辑
- 视图层(View)：应用的展示效果，各类UI组件
- 业务逻辑层(ViewModel)：框架封装的核心，它负责将数据与视图关联起来

上面这个方案就是常说的**MVVM**模型

ViewModel的主要职责是：

- 数据变化后更新视图
- 视图变化后更新数据

它还由两个部分组成：

- 监听器(Observer)：对所有数据的属性进行监听
- 解析器(Compiler)：对每个元素节点的指令进行扫描跟解析，根据指令模板替换数据，以及绑定相应的更新函数

# 12、git中的冲突合并、版本相同解决方法

出现冲突简单场景：

小红对当前库的代码进行修改后，将文件添加到暂存区并上传代码。然后小明并不知道小红修改过代码，还用当前版本的本地仓库修改同一个文件，然后在上传时就会产生冲突

如果出现上述问题，则会用特殊字符来表示冲突的区域，这些字符序列如下：

- `<<<<<<<`
- `=======`
- `>>>>>>>`

`<<<<<<<`和`=======`之间的所有内容都是你的本地修改，这些修改还没有在远程版本库中

`=======`和`>>>>>>>`之间的所有行都是来自远程版本库或另一个分支的修改。

发现冲突后有四种解决方式：

- 保存原修改删除当前修改
- 删除原修改保存当前修改
- 同时保留
- 同时删除

决定好后，可以做以下改变

暂存变化：

```bash
git add <files>
```

提交修改，并添加信息：

```bash
git commit -m "注释"
```

最后将变化推送到远程

```bash
git push
```

# 13、v-show和v-if有什么区别

两者都能控制元素在页面上的显示与隐藏，用法相同

区别有以下三点：

- 控制手段不同

  v-if是直接将dom元素销毁，v-show只是简单的基于css隐藏(给元素添加display: none)

- 编译过程不同

  v-if切换有一个局部编译和卸载的过程，切换过程中合适地销毁和重建内部的事件监听和子组件，v-show只是简单的基于css的隐藏

- 编译条件不同

  v-if是真正的条件渲染，它可以确保在区间的切换过程中销毁和重建。v-show切换显示时不会触发组件的生命周期

性能消耗方面：v-if有更高的切换消耗，v-show由更高的初始渲染消耗

# 14、vue2的初始化过程vsvue3的初始化过程

## vue2

- 合并配置
- 初始化生命周期
- 初始化事件
- 初始化渲染
- 调用`beforeCreate`钩子函数
- init injections and reactivity(这个阶段属性都已注入绑定，而且被`$watch`变成reactivity，但是`$el`还是没有生成，也就是DOM没有生成)
- 初始化state状态(初始化data、props、computed、watcher)
- 调用created钩子函数

在初始化最后，如果检测到有el属性，则调用vm.$mount方法挂载vm，挂载的目标就是把模板渲染成最终的DOM

## vue3

- Vue.createApp()执行的是render的createApp()
- renderer是createRenderer这个方法创建的
- renderer的createApp()是createAppAPI()返回的
- createAppApi接受到render之后，创建一个app实例，定义mount方法
- mount会调用render函数，将vnode转换为真实DOM
