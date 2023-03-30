---
title: 性能优化面试总结
date: 2023-3-30 13:17
categories: 面试
---

# 性能优化的手段

- 从缓存的角度
  - 将一些不常变的大数据通过localStorage/sessionStorage/indexedDB进行读取
  - 活用http缓存（强缓存和协商缓存），将内存存储在内存或者硬盘中，减少对服务器端的请求
- 网络方面比较常用的是静态资源使用CDN
- 打包方面
  - 路由按需加载
  - 优化打包后资源的大小
  - 开启gzip压缩资源
  - 按需加载第三方库
- 代码层面
  - 减少不必要的请求，删除不必要的代码
  - 避免耗时过长的js处理阻塞主线程（耗时且无关DOM可以丢到web worker去处理或者拆分成小的任务）
  - 图片可以使用懒加载的方式，长列表使用虚拟滚动
- 首屏速度提升
  - 代码压缩，减少打包静态资源体积
  - 路由懒加载，首屏就只会请求第一个路由的相关资源
  - 使用cdn加速第三方库
  - ssr服务端渲染，由服务器直接返回拼接好的html页面
- vue常见的性能优化方式
  - 图片懒加载：vue-lazyLoad
  - 虚拟滚动
  - 函数式组件
  - v-show/keep-alive复用dom
  - deffer延时渲染组件
  - 时间切片
  - v-for的时候绑定key值
  - 路由懒加载
  - 组件库按需引入
  - 防抖节流
  - 组件的颗粒度不宜设计太细，层级越深，性能消耗越大
  - 设计vue响应式数据时不宜设计太深，会做全量递归的计算

# 前端监控SDK技术要点

1. 可以通过`window.performance`获取各项性能指标数据
2. 完成的前端监控平台包括：数据采集和上报、数据整理和存储、数据展示
3. 网页性能指标：
   - FP(first-paint)从页面加载到第一个像素绘制到屏幕上的时间
   - FCP(first-contentful-paint)，从页面加载开始到页面内容的任何部分在屏幕上完成渲染的时间
   - LCP(largest-contentful-paint)，从页面加载到最大文本或图像元素在屏幕上完成渲染的时间
4. 以上指标可以通过PerformanceObserver获取
5. 首屏渲染时间计算：通过MutationObserver监听document对象的属性变化

# 如何减少回流重绘，并且充分利用GPU渲染

首先应该避免直接使用DOM API操作DOM，像vue react虚拟DOM让对DOM的多次操作合并成了一次

- 样式集中改变，好的方式时使用动态class

- 读写操作分离，避免读后写，写后又读

  ```js
  // bad 强制刷新 触发四次重排+重绘
  div.style.left = div.offsetLeft + 1 + 'px';
  div.style.top = div.offsetTop + 1 + 'px';
  div.style.right = div.offsetRight + 1 + 'px';
  div.style.bottom = div.offsetBottom + 1 + 'px';
  // good 缓存布局信息 相当于读写分离 触发一次重排+重绘
  var curLeft = div.offsetLeft;
  var curTop = div.offsetTop;
  var curRight = div.offsetRight;
  var curBottom = div.offsetBottom;
  
  div.style.left = curLeft + 1 + 'px';
  div.style.top = curTop + 1 + 'px';
  div.style.right = curRight + 1 + 'px';
  div.style.bottom = curBottom + 1 + 'px';
  ```

  如上述代码，读写分离之后只触发了一次重排，这都得益于浏览器的渲染队列机制：

  > 当我们修改元素的几何属性，导致浏览器触发重排或重绘时。它会把该操作放进渲染队列，等到队列中的操作了一定的数量或者到了一定的时间间隔时，浏览器就会批量执行这些操作

- 使用`display: none;`后元素不会存在渲染树中，这时对它进行各种操作，然后更改display显示即可

- 通过documentFragment创建DOM片段，在它上面批量操作DOM，操作完后再添加到文档中，这样只有一次重排

- 复制节点在副本上操作然后替换它

- 使用BFC脱离文档流，重排开销小

CSS中的`transform`、`opacity`、`filter`、`will-change`能触发硬件加速

# 大图片优化的方案

1. 优化请求数

   - **雪碧图**

     将所有图标合并成一个独立的图片文件，再通过`background-url`和`background-position`来显示图标

   - **懒加载**

     尽量只加载用户访问窗口能访问到的图片或即将浏览的图片，最简单的实现是使用监听页面滚动判断图片是否进入视野；使用intersection Observer API；使用已知工具库；使用css的`background-url`来懒加载

   - base64，小图标或骨架图可以使用内联base64因为base64相比普通图片体积大。注意首屏不需要懒加载，设置合理的占位图避免抖动

2. 减小图片的大小

   - 使用合适的格式比如WebP、svg、video替代GIF、渐进式JPEG
   - 削减图片质量
   - 使用合适的大小和分辨率
   - 删除冗余的图片信息
   - svg压缩

3. 缓存

# 代码优化

1. 非响应式变量可以定义在`created`钩子中使用`this.xxx`赋值
2. 访问局部变量比全局变量快，因为不需要切换作用域
3. 尽可能使用`const`声明变量，注意数组和对象
4. 使用v8引擎时，运行期间，v8会将创建的对象与隐藏类关联起来，以追踪它们的属性特征。我们要避免动态属性赋值以及动态删除属性(使用delete关键字)。属性删除时可以设置为null，这样可以保持隐藏类不变和继续共享
5. 避免内存泄漏的方式
   - 尽可能少创建全局变量
   - 手动清除定时器
   - 少用闭包
   - 清除DOM引用
   - 弱引用
6. 避免强制同步，在修改DOM之前查询相关值
7. 避免布局抖动(一次JS执行过程中多次强制布局和抖动操作)，尽量不要在修改DOM结构时再去查询一些相关值
8. 合理利用css合成动画，如果能用css处理就交给css。因为合成动画会由合成线程执行，不会占用主线程
9. 避免频繁的垃圾回收，优化存储结构，避免小颗粒对象的产生