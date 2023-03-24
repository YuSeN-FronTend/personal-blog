---
title: 服务端渲染(SSR)
date: 2023-3-23 11:10
categories: vue
---

# 服务端渲染(SSR)

## 什么是SSR

vue是用来构建客户端应用的框架。在默认情况下，vue组件的职责是在浏览器生成和操作DOM。然而，vue也支持将组件直接在服务端直接渲染成HTML字符串，作为服务端响应返回给浏览器，最后在浏览器端将静态的HTML"激活"为能够交互的客户端应用。

一个有服务端渲染的vue项目也可以被认为是“同构的”或“通用的”，因为应用大部分代码同时运行在客户端和服务端。

## 使用SSR比客户端单页应用(SPA)相比的优势

- **更快的首屏加载**

  这一点在慢网速和运行缓慢的设备上尤为重要。服务端渲染的HTML页面是不用等JS代码都下载完成之后才显示，所以用户可以更快的看到页面。除此之外，数据获取过程在首次访问时在客户端完成，相比于从客户端获取可能有更快的数据库连接。对于“首屏加载速度和转化率直接相关”的应用来说，这是至关重要的。

- **统一的心智模型**

  我们可以使用相同的语言以及相同的声明式、面向组件的心智模型来开发整个应用，而不需要在后端模板系统和前端框架之间来回切换。

- **更好的SEO**

  搜索引擎爬虫可以直接看到完全渲染的页面

## **使用SSR需要注意的问题**

- 开发中的限制。浏览器特定代码只能在某些生命钩子中使用，一些外部库可能需要特殊处理才能在服务端渲染的应用中运行。
- 更多的与构建配置和部署相关的要求。服务端渲染的应用需要一个能让node.js服务器运行的环境，不像完全静态的SPA那样可以部署在任意的静态文件服务器上。
- 更高的服务端负载。在node环境中渲染一个完整应用比仅仅托管静态文件更加占用CPU资源

再使用SSR之前，应该合理考虑以上这些原因确定是否真的需要。

## SSR vs SSG

**静态站点生成**(Static-Site Generation，缩写为SSG)，也被称为预渲染，是另一种流行的构建快速网站的技术。如果服务端渲染一个页面所需的数据对每个用户来说都是相同的，那么我们可以只渲染一次，提前在构建过程中完成，而不是每次请求进来都重新渲染页面。预渲染的页面生成后作为静态HTML文件被服务器托管。

SSG保留了和SSR相同的性能表现：他带来了优秀的首屏加载性能。同时它比SSR应用的花销更小，也更容易部署，因为输出的是静态HTML和资源文件。SSG仅可以用于消费静态数据的页面，即数据在构建期间就是已知的，并且在多次部署期间不会改变。每当数据变化时，都需要重新部署。

如果调研SSR只是为了优化为数不多的营销页面SEO，那么SSG也许能更好的完成需求，SSG也很适合构建基础内容的网站，比如文档站点或者博客。

## 基础用法

### 渲染一个应用

1. 先创建一个新的文件夹，cd进入
2. 执行`npm init -y`
3. 在 `package.json`中添加`"type": "module"`使Node.js以ES modules mode运行
4. 执行`npm install vue`
5. 创建一个`example.js`

以下使example.js中的代码片段：

```js
// 此文件运行在node.js服务器上
import { createSSRApp } from "vue";
// vue的服务端渲染API位于'vue/server-renderer'路径下
import { renderToString } from 'vue/server-renderer'

const app = createSSRApp({
    data: () => ({ count: 1 }),
    template: `<button @click="count++">{{ count }}</button>`
})

renderToString(app).then((html) => {
    console.log(html);
})
```

运行之后输出

```
<button>1</button>
```

 `renderToString()`接收一个Vue应用实例作为参数，返回一个Promise，当Promise resolve时得到应用渲染的HTML，然后我们可以把vue SSR代码移动到一个服务器请求处理函数里，他将应用的HTML片段包装为完整的页面HTML，接下来我们会使用`express`

- 执行`npm install express`
- 创建下面的`server.js`文件

以下是server.js的代码片段

```js

// 引入express
import express, { response } from 'express';
// 此文件运行在node.js服务器上
import { createSSRApp } from "vue";
// vue的服务端渲染API位于'vue/server-renderer'路径下
import { renderToString } from 'vue/server-renderer'

const server = express();
server.get('/', (requset, response) => {
    const app = createSSRApp({
        data: () => ({ count: 1 }),
        template: `<button @click="count++">{{ count }}</button>`
    })
    renderToString(app).then((html) => {
        response.send(`
            <!DOCTYPE html>
            <html>
                <head>
                    <title>Vue SSR Example</title>
                </head>
                <body>
                    <div id="app">${html}</div>
                </body>
            </html>
        `)
    })
})

server.listen(3000, () => {
    console.log('ready');
})
```

最后执行`server.js`  然后访问`http://localhost:3000`就可以看到页面上的按钮了

## 客户端激活

按钮渲染出来但是并没有什么功能，这段代码完全是静态的。所以为了客户端应用可交互，vue需要一个激活步骤。在激活过程中，vue会创建一个与服务端完全相同的应用实例，然后将每个组件与它应该控制的DOM节点相匹配，并添加DOM事件监听器。

### 代码结构

我们将应用的创建逻辑拆分到一个单独的文件`app.js`中:

```js
// app.js (在服务器和客户端之间共享)
import { createSSRApp } from "vue";

export function createApp() {
    return createSSRApp({
        data: () => ({
            count: 1
        }),
        template: `<button @click="count++">{{ count }}</button>`
    })
}
```

该文件及其依赖项在服务器和客户端之间共享——它们被称作**通用代码**，下面是其他文件的例子

- `client.js`

  ```js
  import { createApp } from "./app.js";
  
  createApp().mount('#app')
  ```

- `server.js`

  ```js
  
  // 引入express
  import express from 'express';
  // vue的服务端渲染API位于'vue/server-renderer'路径下
  import { renderToString } from 'vue/server-renderer'
  // 引入createApp
  import { createApp } from './app.js';
  const server = express();
  server.get('/', (requset, response) => {
      // 上面讲到的代码已经在app.js中封装完毕，直接引用即可
      const app = createApp();
      renderToString(app).then((html) => {
          response.send(`
              <!DOCTYPE html>
              <html>
                  <head>
                      <title>Vue SSR Example</title>
                    	// 添加import Map以支持在浏览器中使用 import * from 'vue'
                      <script type="importmap">
                          {
                            "imports": {
                                 "vue": "https://unpkg.com/vue@3/dist/vue.esm-browser.js"
                              }
                          }
                      </script>
                      // 外部引入client.js文件
                      <script type="module" src="./client.js"></script>
                  </head>
                  <body>
                      <div id="app">${html}</div>
                  </body>
              </html>
          `)
      })
  })
  // 用于托管客户端文件
  server.use(express.static('.'))
  server.listen(3000, () => {
      console.log('ready');
  })
  ```

  继续执行 `node server.js`指令，访问`http://localhost:3000`，发现按钮可以点击自增了

## 更通用的解决方案

上面的简单例子到一个生产就虚的SSR应用还有很多工作，但是vue推荐使用更通用更集成化的解决方案，如下面例子：

- `Nuxt`

  这是一个构建与vue生态系统之上的全栈框架，它为vue SSR提供了丝滑的开发体验。更厉害的是，它还可以被当作一个静态站点生成器来用

- `Quasar`

  这是一个基于vue的完美解决方案，它可以用同一套代码库构建不同目标的应用，如SPA、SSR、PWA、移动端应用、桌面端应用以及浏览器插件。除此之外哦，它还提供了一套`Material Design`风格的组件库

- Vite SSR

  Vite提供了内置的Vue服务端渲染支持，但它的设计是偏底层的。使用这种方式只有在有丰富的SSR和构建工具经验，并希望对应用的架构做深入定制时才推荐使用。