---
title: vue3学习
date: 2023-5-13 18:55
categories: vue
---

# 项目创建

- npm create vite@latest

  但是可能会比较慢

- yarn create vite

  全局安装yarn命令即可使用

- pnpm create vite

  全局安装pnpm命令后即可使用(对node.js版本有限制，起码要16.14版本)

# 路由配置

由于使用vite创建的vue3项目没有像vue-cli一样可以默认配置引入路由的，需要手动下载，引入并配置路由

- 下载

  ```bash
  npm i vue-router -S
  ```

- 配置

  在src文件夹下创建router文件夹，在文件夹内创建index.ts文件，做如下配置

  ```typescript
  import { createRouter, createWebHistory } from 'vue-router';
  
  const routes:any = [
      {
          path: '/',
          name: 'Home',
          component: () => import('../view/Home.vue')
      },
      {
          path: '/demo',
          name: 'Demo',
          component: () => import('../view/demo.vue')
      }
  ]
  
  const router = createRouter({
      history: createWebHistory(),
      routes
  })
  
  export default router;
  ```

- 引入

  在main.ts做如下引入

  ```ts
  import { createApp } from 'vue'
  import App from './App.vue'
  import router from "./router/index";
  
  const app = createApp(App);
  
  app.use(router)
  app.mount('#app')
  ```

  这样路由就配置好了，在App.vue的模板中加入`router-view`标签就可以显示配置路由的根路由对应的页面了

- 扩展

  如果想实现一个简单的路由跳转，也很简单，在vue2时实现路由跳转是使用this.$router.push，但vue3中没有this，那该如何获取到$router属性呢？见如下代码

  ```vue
  <template>
      <div>
          我是home
          <button @click="routeChange">点击我跳转到demo路由</button>
      </div>
  </template>
  
  <script setup lang="ts">
  import { getCurrentInstance } from 'vue'
      const { appContext }: any = getCurrentInstance();
      function routeChange() {
          appContext.config.globalProperties.$router.push('/demo')
      }
  </script>
  ```

  这样在点击按钮时就会跳转到/demo路由所对应的页面(前提是要在路由页面配置)

# 添加状态管理模式库

vue2中的状态管理模式库就是我们熟悉的vuex，但在之前的学习中我们也提到过，vue3给了我们更好的解决方案----pinia。

- 安装

  ```bash
  npm install pinia
  ```

- 配置

  在src目录下创建store文件夹，添加index.ts文件，向里面添加如下配置

  ```tsx
  import { defineStore } from 'pinia' // 引入pinia
  
  export const useCar = defineStore('test', {
      state: () => {
          return({
              msg: '这是pinia的数据',
              name: '小狮子',
              age: 18
          })
      }
  })
  ```

- 引入

  在main.js全局引入

  ```ts
  import { createApp } from 'vue'
  import App from './App.vue'
  import router from "./router/index";
  import { createPinia } from 'pinia'
  const app = createApp(App);
  app.use(createPinia())
  app.use(router)
  app.mount('#app')
  ```

- 使用

  在页面中引入store/index.ts即可很容易的获取到state中定义的数据

  ```ts
  import { useCar } from '../store';
  let store = useCar();
  console.log(store);
  ```

  这样就获取到原始的数据值了，如像咱们上面设置的初始值，则可以直接store.name去进行修改，然后所有页面引入该值的地方都会发生改变

- $reset

  将状态重置到初始值。如果我们做了大量的修改已经忘掉初始值是什么并且还需要使用的时候，可以直接调用此方法，即可还原到原始值

  ```ts
  function reset() {
      store.$reset()
  }
  ```

- $patch

  可以将pinia的数据进行统一修改，特点是它状态只刷新一次

  ```ts
  function patch() {
      store.$patch({
          name:'张建',
          age: 21
      })
  }
  ```

- getter

  和vuex中的计算属性类似，这里不多赘述

- actions

  众所周知在vuex中它只用来处理异步操作，同步操作在mutations里面进行，但在pinia中删除了mutations，因为只需要actions就都可以搞定了，这样一来代码就清晰很多了，也更好维护了
