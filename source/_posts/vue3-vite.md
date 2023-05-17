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

# 实现动态的导航栏路由匹配

如果一个管理系统平台一定少不了导航栏。以前写项目的时候都是直接在导航栏中添加静态的名字，图标以及路由信息。这次有了一个好的想法，直接获取router里面的信息，然后遍历生成一个导航栏，这样每次再添加新的导航栏只需要添加路由，页面上就会显示出对应的导航栏。

- 布局

  利用element布局容易调整宽高即可

- 导航栏

  使用的是elementPlus的导航栏

- 实现过程

  首先通过router的内置API，getRoutes来获取所有路由，并筛选属于一级菜单的部分

  ```ts
  import { useRouter } from 'vue-router';
  const router = useRouter();
  let list = router.getRoutes().filter((item) => item.meta.type === 'first')
  ```

  寻找到即可去实现遍历菜单功能，但需要考虑两种情况：

  - 一级菜单有子路由
  - 一级菜单没有子路由

  思路理顺之后的代码如下

  ```html
  <div v-for="item in list" :key="item.path">
  	<el-sub-menu v-if="item.children.length">
  		<template #title>
          	<el-icon>
              	<Icon :icon="item.meta.icon"></Icon>
              </el-icon>
              span>{{ item.meta.name }}</span>
  		</template>
  	<el-menu-item-group>
      	<el-menu-item v-for="childItem in item.children" :key="childItem.path" :index="childItem.path">
          	<el-icon>
              	<Icon :icon="childItem.meta?.icon"></Icon>
  			</el-icon>{{ childItem.meta?.name }}</el-menu-item>
  		</el-menu-item-group>
  	</el-sub-menu>
      <el-menu-item v-else :index="item.path">
      	<template #title>
          	<el-icon>
              	<Icon :icon="item.meta.icon"></Icon>
  			</el-icon>
              {{ item.meta.name }}</template>
  	</el-menu-item>
  </div>
  ```

  这样一来只要按照一定的规则书写路由，那每次添加完路由，页面上就会显示了

- 踩坑

  elementPlus的图标相较于element有所变动，是以标签形式引用，这样对于我们的动态引入是很有影响的。寻找了很多资料最后发现注册全局组件的方法比较科学，实现如下

  ```ts
  import * as Icons from '@element-plus/icons'
  // 创建Icon组件
  const Icon = (props: { icon: string }) => {
      const { icon } = props
      console.log(icon);
      return createVNode(Icons[icon as keyof typeof Icons])
  }
  // 注册Icon组件
  app.component('Icon', Icon)
  ```

  这样注册完在页面中直接调用即可

  ```html
  <Icon :icon="item.meta.icon"></Icon>
  ```

  但这样图标太大了，我用了很多方法包括强制穿透样式，都没有效果。最后才想起来在外面套一个el-icon标签，结果真的实现了，至此功能已实现完成

- 优化

  今天突然想到如果一个路由有好多层children，我们的代码就会出现bug，所以我写了一个递归组件，让代码自己检测，这样就解决了这个问题，思路如下：

  - 注册一个全局组件nav-aside

    ```ts
    import navAside from './components/layout/navAside/index.vue'
    app.component('navAside', navAside)
    ```

    navAside内部代码为

    ```vue
    <template>
            <div v-for="item in props.navData" :key="item.path">
                <el-sub-menu v-if="item.children?.length">
                    <template #title>
                        <el-icon>
                            <Icon :icon="item.meta.icon"></Icon>
                        </el-icon>
                        <span>{{ item.meta.name }}</span>
                    </template>
                    <nav-aside :navData="item.children"></nav-aside>
                </el-sub-menu>
                <el-menu-item v-else :index="item.path">
                    <template #title>
                        <el-icon>
                            <Icon :icon="item.meta.icon"></Icon>
                        </el-icon>
                        {{ item.meta.name }}</template>
                </el-menu-item>
            </div>
    </template>
    
    <script setup lang="ts">
    let props = defineProps(['navData']);
    
    </script>
    ```

    也就是把原来menu里面的代码剖离出来，可以发现再次页面也调用了本页面，这个还是比较巧妙的，但是一定要留一个出口来避免死循环问题

  - 修改aside内部代码

    ```vue
    <template>
        <el-row>
            <el-col :span="24">
                <div style="height: 7vh"></div>
                <el-menu :uniqueOpened="true" default-active="/dashboard" class="el-menu-vertical-demo" @open="handleOpen"
                    @close="handleClose" background-color="#191a23" text-color="#fff" active-text-color="#4d70ff" router>
                   <nav-aside :navData="props.navData"></nav-aside>
                </el-menu>
            </el-col>
        </el-row>
    </template>
    
    <script setup lang="ts">
    let props = defineProps(['navData'])
    
    const handleOpen = (key: any, keyPath: any) => {
        console.log(key, keyPath);
    };
    const handleClose = (key: any, keyPath: any) => {
        console.log(key, keyPath);
    };
    </script>
    ```

    在原来位置引入组件即可，至此优化成功

# 导航栏切换路由产生动画效果

使用vue自带的transition的标签即可解决

```html
<transition>	
	<router-view></router-view>
</transition>
```

但再vue3中这样使用报错，因为router-view标签不可以在transition和keep-alive标签中使用，所以我们使用以下格式

```html
<router-view v-slot="{ Component }">
	<transition>
    	<component :is="Component" />
    </transition>
</router-view>
```

# 导航栏的显示与隐藏

由于太长时间没有写管理系统相关代码，并且还用的是vue3和TS的新知识，遇到了一些坑，具体如下：

- 隐藏导航栏时aside宽度不变化问题

  寻找了很长时间，最后发现它的宽度是定死的，将宽度更改成auto后，再添加overflow: hidden属性，即可完成侧边栏宽度随着导航栏宽度缩小而缩小

- 递归组件后隐藏导航栏文字不消失问题

  这真的是个大坑，搜索了很多相关资料，都说是因为外层有一个div，会造成预期之外的错误，需要下载vue-fragment插件。下载之后进行类似操作也没有想过，还试过根据状态去隐藏文字，虽然实现了但是感觉治标不治本，最后突然想起，vue3不需要根组件的原因就是因为默认给每个组件模板添加了`<fragment>`标签，所以直接在模板中添加`<template>`标签进行循环，问题解决

  ```html
  <template>
      <template v-for="item in props.navData" :key="item.path">
          <el-sub-menu v-if="item.children?.length" :index="item.path">
              <template #title>
                  <el-icon>
                      <Icon :icon="item.meta.icon"></Icon>
                  </el-icon>
                  <span>{{ item.meta.name }}</span>
              </template>
              <nav-aside :navData="item.children"></nav-aside>
          </el-sub-menu>
          <el-menu-item v-else :index="item.path">
              <el-icon>
                  <Icon :icon="item.meta.icon"></Icon>
              </el-icon>
              <template #title>
                  <span>{{ item.meta.name }}</span>
              </template>
          </el-menu-item>
      </template>
  </template>
  ```

- 点击侧边栏刷新高亮消失问题

  只需要再sessionStorage存入每次点击的路由路径，每次刷新获取即可。一定要给展开的菜单也设置路径index，不然可能会导致刷新高亮在但是一级菜单不会默认展开
