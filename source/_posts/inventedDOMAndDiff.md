---
title: 虚拟DOM和diff算法学习
date: 2023-4-7 11:19
categories: javaScript
---
# 环境搭建

- 注册身份证

  ```bash
  npm init -y
  ```

- 安装snabbdom依赖

  ```bash
  npm i -S snabbdom
  ```

- 安装webpack依赖

  ```bash
  npm i -D webpack webpack-cli webpack-dev-server
  ```


# 简述虚拟DOM

虚拟DOM是通过一个h函数来生成虚拟的DOM方便diff算法去工作完成最小量更新

- 创建h.js

  ```js
  import vnode from "./vnode";
  
  // 书写简单的h函数，也就是一定要传入三个参数
  // 形态① h('div', {}, '文字)
  // 形态② h('div', {}, [])
  // 形态③ h('div', {}, h())
  export default function h(sel, data, c) {
      // 检查参数的个数
      if(arguments.length !== 3) {
          throw new Error('请输入三个参数')
      }
      // 检查参数的类型
      if(typeof c === 'string' || typeof c === 'number') {
          // 说明现在调用的h函数是形态①
          return vnode(sel, data, undefined, c, undefined);
      } else if(Array.isArray(c)) {
          // 说明现在调用的是形态②
          let children = []
          // 遍历c收集children，因为在index.js引用时已执行完毕并输出结果
          for(let i = 0; i < c.length; i++) {
              if (!(typeof c[i] === 'object' && c[i].hasOwnProperty('sel'))){
                  throw new Error('传入的数组中有的项不是h函数')
              }
              children.push(c[i]);
          }
          // 循环结束返回虚拟节点
          return vnode(sel, data, children, undefined, undefined)
      } else if(typeof c === 'object' && c.hasOwnProperty('sel')){
          // 说明现在调用的是形态③
          // 传入的c是唯一的children
          return vnode(sel, data, [c], undefined, undefined);
      } else {
          throw new Error('传入的第三个参数类型不对')
      }
  }
  ```

- 创建vnode.js

  用于生成h函数最终返回的对象

  ```js
  // 函数功能非常简单，就是把传入的5个参数组合成对象返回
  export default function vnode(sel, data, children, text, elm) {
      return { sel, data, children, text, elm }
  }
  ```

# diff算法

- diff可以检测到key相等的虚拟节点，无论是添加删除修改顺序都会识别旧虚拟节点和新虚拟节点中没有改变的部分并且不再重复渲染
- diff算法只在同一个虚拟节点上才做精细化比较(也就是sel要相同)
- diff算法还只会进行同层比较，如果跨层比较diff算法是不会生效的，这并不是缺陷，而是这样没有任何开发价值

