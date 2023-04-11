---
title: 虚拟DOM和diff算法学习
date: 2023-4-7 11:19
categories: vue
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

虚拟DOM转化为DOM有很多种情况：

- 如果oldVnode和newVnode不是同一个节点(同一个sel)，删除旧的，插入新的

- 如果是同一节点还分为以下情况(在createVnode.js实现)

  - 如果是同一个对象，什么都不做

  - 如果newVnode和oldVnode中都有text属性且相等，则什么都不做。不相等就用新节点的text来替换老节点的text

  - 如果老节点没有children而新节点有，则删除老节点的text并添加新节点的children

  - 如果老节点有children新节点也有，则用到了**diff算法**(主要在updateChildren.js中实现)

    diff算法大概就分为四种情况，设立四个指针分别命名为新前、新后、旧前、旧后，按照以下规则比较移动指针或插入节点

    - 新前和旧前

      如若相等，新前和新后同时下移

    - 新后和旧后

      如果相等，新前和新后同时上移

    - 新后和旧前

      如若相等，新后指向的节点插入到旧后之后，旧前所指向的对应节点变为undefined，新后上移，旧前下移

    - 新前和旧后

      如若相等，新前指向的节点插入到旧前之前，旧后所指向的对应节点变为undefined，新前下移，旧后上移

- 创建patch.js

  这是一个集成的文件，里面实现将DOM节点转化为虚拟节点，并且判断oldVnode和newVnode是否是同一个节点

  ```js
  import vnode from "./vnode";
  import patchVnode from "./patchVnode";
  import createElement from './createElement'
  
  export default function patch(oldVnode, newVnode) {
      // 判断传入的参数是DOM节点还是虚拟节点
      if(oldVnode.sel === '' || oldVnode.sel === undefined) {
          // 传入的参数为DOM节点，要封装成虚拟节点
          oldVnode = vnode(oldVnode.tagName.toLowerCase(), {}, [], undefined, oldVnode);
      }
      // 判断oldVnode和newVnode是否是同一个节点
      if(oldVnode.sel === newVnode.sel && oldVnode.key === newVnode.key) {
          console.log('是同一个节点');
          patchVnode(oldVnode, newVnode);
      }else{
          console.log('不是同一个节点，暴力删除旧的，插入新的');
          let newVnodeElm = createElement(newVnode);
          // 判断调用insertBefore之前的值是否有值
          if(oldVnode.elm.parentNode && newVnodeElm) {
              oldVnode.elm.parentNode.insertBefore(newVnodeElm, oldVnode.elm)
          }
          // 暴力删除旧节点
          oldVnode.elm.parentNode.removeChild(oldVnode.elm);
      }
  }
  ```

- 创建createElement.js文件

  此文件可以将虚拟节点转化为DOM，并且可以时间多层转化，内部有递归实现

  ```js
  // 真正创建节点，将vnode创建为DOM, 插入到pivot之前
  export default function createElement(vnode) {
      // 创建一个DOM节点
      let domNode = document.createElement(vnode.sel);
      // 有子节点还是有文本
      if(vnode.text !== '' && (vnode.children === undefined || vnode.children.length === 0)) {
          // 节点内部为文字
          domNode.innerText = vnode.text;
      } else if (Array.isArray(vnode.children) && vnode.children.length > 0){
          for(let i = 0; i < vnode.children.length; i++) {
              // 获取每一个子元素
              let ch = vnode.children[i];
              // 获取递归后的elm
              let chDOM = createElement(ch);
              // 将递归后的节点上树
              domNode.appendChild(chDOM); 
          }
      }
      // 补充elm属性 为递归做准备
      vnode.elm = domNode;
      // 返回elm，是一个纯DOM对象
      return vnode.elm;
  }
  ```

- 创建patchVnode.js

  用于处理比较在同一层新旧节点区别问题

  ```js
  import createElement from "./createElement";
  import updateChildren from "./updateChildren";
  
  export default function patchVnode(oldVnode, newVnode) {
      // 如果新旧节点是同一个对象
      if (oldVnode === newVnode) return;
      // 如果newVnode中有text属性
      if (newVnode.text !== undefined && (newVnode.children === undefined || newVnode.children.length === 0)) {
          console.log('新节点有text属性');
          if (oldVnode.text === newVnode.text) return;
          // 如果新虚拟节点的text和老的虚拟节点的text不同，则替换
          oldVnode.elm.innerText = newVnode.text
      } else {
          // newVnode没有text属性
          console.log('新节点没有text属性');
          // 判断老的没有有children
          if (oldVnode.children !== undefined && oldVnode.children.length > 0) {
              // 老的有children，新的也有，此时为最复杂的情况
              updateChildren(oldVnode.elm, oldVnode.children, newVnode.children)
          } else {
              // 老的没有children  新的有
              // 清空老的text
              oldVnode.elm.innerText = ''
              // 遍历新的子节点
              for (let i = 0; i < newVnode.children.length; i++) {
                  let dom = createElement(newVnode.children[i]);
                  oldVnode.elm.appendChild(dom)
              }
          }
      }
  }
  ```

- 创建updateChildren.js

  用来实现diff算法中最难的并且最厉害的功能，用于比较新旧节点都有children的情况

  ```js
  import createElement from "./createElement";
  import patchVnode from "./patchVnode";
  
  // 判断是否是同一虚拟节点
  function checkSameVnode(oldVnode, newVnode) {
      return oldVnode.sel === newVnode.sel && oldVnode.key === newVnode.key;
  }
  
  export default function updateChildren(parentElm, oldCh, newCh) {
  
      // 旧前
      let oldStartIdx = 0;
      // 新前
      let newStartIdx = 0;
      // 旧后
      let oldEndIdx = oldCh.length - 1;
      // 新后
      let newEndIdx = newCh.length - 1;
      // 旧前节点
      let oldStartVnode = oldCh[0];
      // 新前节点
      let newStartVnode = newCh[0];
      // 旧后节点
      let oldEndVnode = oldCh[oldEndIdx];
      // 新后节点
      let newEndVnode = newCh[newEndIdx];
  
      let keyMap = null;
  
      // 开始循环
      while(oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
          // 首先不是判断命中，实现判断要掠过的节点是否是undefined标记的东西
          if(oldStartVnode == null || oldCh[oldStartIdx] == undefined) {
              oldStartVnode = oldCh[++oldStartIdx]; 
          } else if (oldEndVnode == null || oldCh[oldEndIdx] == undefined) {
              oldEndVnode = oldCh[--oldEndIdx];
          } else if (newStartVnode == null || newCh[newStartIdx] == undefined) {
              newStartVnode = newCh[++newStartIdx];
          } else if (newEndVnode == null || newCh[newEndIdx] == undefined) {
              newEndVnode = newCh[--newEndIdx];
          }else if(checkSameVnode(oldStartVnode, newStartVnode)) {
              // 新前与旧前
              patchVnode(oldStartVnode, newStartVnode);
              oldStartVnode = oldCh[++oldStartIdx];
              newStartVnode = newCh[++newStartIdx];
          }else if(checkSameVnode(oldEndVnode, newEndVnode)) {
              // 新后与旧后
              patchVnode(oldEndVnode, newEndVnode);
              oldEndVnode = oldCh[--oldEndIdx];
              newEndVnode = newCh[--newEndIdx];
          }else if(checkSameVnode(oldStartVnode, newEndVnode)) {
              // 新后与旧前
              patchVnode(oldStartVnode, newEndVnode);
              // 当新后与旧前命中后，要把当前旧前指向的节点插入到旧后节点的后面
              // 如何一定节点？？ 只要插入一个已经在DOM树上的节点，它就会被移动
              parentElm.insertBefore(oldStartVnode.elm, oldEndVnode.elm.nextSibling);
              oldStartVnode = oldCh[++oldStartIdx];
              newEndVnode = newCh[--newEndIdx];
          }else if(checkSameVnode(oldEndVnode, newStartVnode)) {
              // 新前与旧后 
              patchVnode(oldEndVnode, newStartVnode);
              // 当新前与旧后命中后，要把当前旧后指向的节点插入到旧前节点的前面
              // 如何一定节点？？ 只要插入一个已经在DOM树上的节点，它就会被移动  
              parentElm.insertBefore(oldEndVnode.elm, oldStartVnode.elm)
              oldEndVnode = oldCh[--oldEndIdx];
              newStartVnode = newCh[++newStartIdx]
          }else{
              console.log('进入');
              // 四种命中都没有找到
              if(!keyMap){
                  keyMap = {};
                  for(let i = oldStartIdx; i <= oldEndIdx; i++) {
                      let key = oldCh[i].key;
                      if(key != undefined) {
                          keyMap[key] = i;
                      }
                  }
                  console.log(keyMap);
              }
              // 寻找当前这项在keyMap中的映射的位置序号
              let idxInOld = keyMap[newStartVnode.key];
              console.log(idxInOld);
              if (!idxInOld) {
                  // 如果idxInOld是undefined表示它是全新的项
                  // 被加入地向不是真正的DOM节点
                  console.log(newCh);
                  parentElm.insertBefore(createElement(newStartVnode), oldStartVnode.elm)
              } else {
                  // 如果不是undefined，不是全新的项，而是要移动
                  const elmToMove = oldCh[idxInOld];
                  // 对比新旧节点差异
                  patchVnode(elmToMove, newStartVnode);
                  // 把这项设成undefined，表示我已经处理完这项了
                  oldCh[idxInOld] = undefined;
                  // 移动，调用insertBefore也可以实现移动
                  parentElm.insertBefore(elmToMove.elm, oldStartVnode.elm)
              }
              // 指针下移，只移动新的头
              newStartVnode = newCh[++newStartIdx];
          }
      }
  
      // 查看是否有剩余节点，循环结束后start比old小
      if(newStartIdx  <= newEndIdx) {
          // 遍历新的newCh，添加到老的没有处理的之前
          for(let i = newStartIdx; i <= newEndIdx; i++) {
              console.log(oldCh);
              parentElm.insertBefore(createElement(newCh[i]), oldCh[oldStartIdx - 1].elm.nextSibling);
          }
      }else if(oldStartIdx <= oldEndIdx) {
          // 批量删除oldStart和oldEnd指针之间的项
          for(let i = oldStartIdx; i <=oldEndIdx; i++) {
              if(oldCh[i]) {
                  parentElm.removeChild(oldCh[i].elm)
              }
          }
      }
  }
  ```

  

心得：

- diff可以检测到key相等的虚拟节点，无论是添加删除修改顺序都会识别旧虚拟节点和新虚拟节点中没有改变的部分并且不再重复渲染
- diff算法只在同一个虚拟节点上才做精细化比较(也就是sel要相同)
- diff算法还只会进行同层比较，如果跨层比较diff算法是不会生效的，这并不是缺陷，而是这样没有任何开发价值

