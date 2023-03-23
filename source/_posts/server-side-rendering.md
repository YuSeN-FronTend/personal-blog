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