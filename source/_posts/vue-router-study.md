---
title: vue路由专项学习
date: 2023-6-23 16:02
categories: vue
---

# 两种路由模式的区分

## hash模式

- 路由中#号后面就是hash的内容
- 可以通过location.hash拿到对应的内容
- 可以通过onhashchange监听hash的改变

## history

- history即正常路径
- 可以通过location.pathname拿到相对应的内容
- 可以用onpopstate监听history的变化

## 基本代码的实现

准备工作，用我们写的页面把原来引用的router插件取代即可，方便调试，然后开始代码实现。

基本的router代码实现需要分为以下6步

- 改变url

  ```js
  let Vue;
  
  class VueRouter {
      constructor() {
  
      }
  }
  
  VueRouter.install = function(_vue) {
      Vue = _vue;
  
      Vue.component('router-link', {
          props: {
              to: {
                  type: String,
                  require: true
              }    
          },
          render(h) {
              return h('a',{
                  attrs: {
                      href: '#' + this.to
                  }
              }, 'router-link');
          }
      })
  
      Vue.component('router-view', {
          render(h) {
              return h('div', 'router-link');
          }
      })
  }
  
  export default VueRouter;
  ```

  因为调用router会用到Vue.use(router)，在vue源码当中，use是优先检测是否有install方法，如果有就优先调用里面的功能，这里处理的是路由中两个最重要的标签。并且给router-link添加一个属性，完成路由的切换。

- 触发监听事件和改变vue-router里面的current变量

  ```js
  class VueRouter {
      constructor(options) {
          this.mode = options.mode || '/';
          this.routes = options.routes;
          this.current = '/';
          this.init();
      }
      init() {
          if(this.mode === 'hash') {
              window.addEventListener('load',() => {
                  this.current = location.hash.slice(1);
              })
              window.addEventListener('hashchange', () => {
                  this.current = location.hash.slice(1);
              })
          }
      }
  }
  ```

  这是两边，由于功能紧凑，放在一点了。触发的监听事件上面已经提到过了，由于这里是简易版，只对hash模式做解析，第一次监听是首次渲染页面，第二次是监听路由hash变化的时候触发。

- vue监视current的监视者

  ```
  
  ```

  

