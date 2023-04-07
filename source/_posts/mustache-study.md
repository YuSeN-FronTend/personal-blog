---
title: mustache源码学习
date: 2023-4-7 9:17
categories: javaScript
---

# mustache模板引擎

mustache是模板引擎的鼻祖，vue的模板引擎思想也来源于mustache

mustache生成过程是将一段html代码编程tokens格式，然后填入数据再转成html代码形式

- 先搭建webpack环境

  - `npm init -y`

    用来生成package.json

  - `npm install webpack webpack-cli webpack-dev-server`

    下载webpack依赖，但是要基于自己的node版本

  - 配置webpack.config.js代码

    ```js
    module.exports = {
        // 开发环境
        mode: 'development',
        // 入口文件
        entry: './src/mustache.js',
        // 出口文件
        output:{
            filename: 'bundle.js'
        },
        // 使打包后的代码在控制台打印时行数与我们写的代码行数相对应
        devtool: 'inline-source-map'
    }
    ```

- 创建index.html文件

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
      
  </head>
  <body>
      <div id="box"></div>
      <!-- 引入打包好的js代码 -->
      <script src="./dist/bundle.js"></script>
      <script>
          let templateStr = `
              <div>
                  <ol>
                      {{#students}}
                      <li>
                          学生{{name}}的爱好是
                          <ol>
                              {{#hobbies}}
                              <li>{{.}}</li>
                              {{/hobbies}}
                          </ol>    
                      </li>
                      {{/students}}    
                  </ol>    
              </div>
          `;
          let data = {
              students: [
                  {'name': '鞠丰泽', 'hobbies': ['吃屎', '鼻青脸肿']},
                  {'name': '张建', 'hobbies': ['打游戏', '健身', '装b']},
                  {'name': 'AKA-YM袁猛', 'hobbies': ['军人'] },
              ]
          }
          let box = document.getElementById('box');
          // 引入打包好的构造函数中的方法
          let domStr = Mustache.render(templateStr, data);
          box.innerHTML = domStr;
      </script>
  </body>
  </html>
  ```

- 创建mustache.js

  此文件为入口文件

  ```js
  import parseTemplateTokens from './parseTemplateTokens'
  import renderTemplate from './renderTemplate';
  // 全局创建Mustache
  window.Mustache = {
      render(templateStr, data) {
          // 将html代码转化成tokens形式并获取
          let tokens = parseTemplateTokens(templateStr);
          // 将tokens填入数据后再组合成html代码
          let domStr = renderTemplate(tokens, data);
          return domStr;
      }
  }
  ```

- 创建scanner.js

  将传入的模板字符串通过"{{"和"}}"来进行分割

  ```js
  export default class Scanner{
      constructor(templateStr){
          this.templateStr = templateStr
          // 移动的指针
          this.pos = 0;
          // 尾巴字符串
          this.tail = templateStr;
      }
      // 遇到指定内容跳过
      scan(tag){
          if(this.tail.indexOf(tag) === 0) {
              this.pos += tag.length;
              this.tail = this.templateStr.substring(this.pos)
          }
      }
      // 指针扫描遇到指定内容结束
      scanUnit(stopTag){
          const pos_backup = this.pos;
          while (!this.eos() && this.tail.indexOf(stopTag) !== 0) {
              this.pos++;
              this.tail = this.templateStr.substring(this.pos)
          }
          return this.templateStr.substring(pos_backup, this.pos)
      }
      // 判断指针是否到头
      eos() {
          return this.pos >= this.templateStr.length;
      }
  } 
  ```

- 创建parseTemplateTokens.js

  将scanner.js中生成的数组继续封装成'text', 'name', '#', '/'做区分后的数组

  ```js
  import Scanner from "./scanner";
  import nextTokens from "./nextTokens";
  
  export default function parseTemplateTokens(templateStr) {
      let tokens = [];
      // 获取scanner.js中生成的数据
      let scanner = new Scanner(templateStr);
      let words;
      // 判断指针是否遍历完成
      while(!scanner.eos()){
          // 调用scanner中的scanUnit方法
          words = scanner.scanUnit('{{');
          // words不是空数组
          if(words !== '') {
              tokens.push(['text', words]);
          }
          // 调用scanner中的scan方法
          scanner.scan("{{")
          // 再次调用scanner中的scanUnit方法来做闭合
          words = scanner.scanUnit('}}');
          if (words !== '') {
              if(words[0] === '#'){
                  tokens.push(['#', words.substring(1)])
              }else if(words[0] === '/') {
                  tokens.push(['/',words.substring(1)])
              } else {
                  tokens.push(['name', words])
              }
          }
          scanner.scan("}}")
      }
      // 调用nextTokens函数
      return nextTokens(tokens);
  }
  ```

- 创建nextTokens.js

  将简单处理好的数组变成我们想要的符合条件的tokens数组(这里用到了栈的思想和收集器思想)

  ```js
  export default function nextTokens(tokens) {
      // 结果数组
      let nestedTokens = [];
      // 栈
      let sections = [];
      // 收集器
      let collector = nestedTokens;
      for(let i = 0; i < tokens.length; i++) {
          let token = tokens[i];
          switch (token[0]){
              case '#':
                  // 向收集器中添加
                  collector.push(token);
                  // 向栈中添加
                  sections.push(token);
                  // 给收集器中开辟新空间并定义为空数组
                  collector = token[2] = [];
                  break;
              case '/':
                  // 弹栈
                  sections.pop();
                  // 改变收集器的区域
                  collector = sections.length > 0 ? sections[sections.length - 1][2] : nestedTokens;
                  break;
              default:
                  // 向当前收集器添加元素
                  collector.push(token)
          }
      }
      return nestedTokens;
  }
  ```

- 创建renderTemplate.js

  将tokens再转换成html片段

  ```js
  import  lookup  from "./lookup.js";
  import parseArray from "./parseArray.js";
  
  export default function renderTemplate(tokens, data) {
      // 创建结果字符串
      let resultStr = '';
      for(let i = 0; i < tokens.length; i++) {
          // 将每项赋值
          let token = tokens[i];
          // 如果数组第0项为text 直接拼接到resultStr
          if(token[0] === 'text') {
              resultStr += token[1]
          } else if(token[0] === 'name'){
              // 如果数组第0项为name 需要使用lookup处理过的值来拼接到resultStr，lookup主要用来解决{{a.b.c}}无法识别问题
              resultStr += lookup(data, token[1]);
          } else if(token[0] === '#') {
              // 如果数组第0项为# 需要引入parseArray来完成递归遍历
              resultStr += parseArray(token, data)
          }
          
      }
      return resultStr;
  }
  ```

- 创建lookup.js

  来解决{{a.b.c}}无法识别问题

  ```js
  // 功能是可以在dataObj对象中，寻找用连续点符号的keyName属性
  
  export default function lookup(dataObj, keyName){
      // 判断keyName中是否存在"."但不是"."
      if(keyName.indexOf('.') !== -1 && keyName !== '.') {
          // 临时变量
          let temp = dataObj;
          // 用点拆分
          let names = keyName.split('.')
          // 遍历获取值
          names.forEach((item) => {
              temp = temp[item]
          })
          return temp;
      }
      // 如果不存在也不是，直接返回
      return dataObj[keyName]
  }
  ```

- 创建parseArray.js

  用来处理token第0项为#的情况

  ```js
  // 处理数组 结合renderTemplate实现递归
  
  import  lookup  from "./lookup.js";
  import renderTemplate from "./renderTemplate.js";
  
  export default function parseArray(token, data) {
      // 通过lookup来处理数据
      let v = lookup(data, token[1]);
      console.log(v);
      let resultStr = ''
      // 遍历数据来确定渲染的DOM次数
      for(let i = 0; i < v.length; i++) {
          // 因为在mustache中'.'属性是一个特定的值
          // v[i]['.'] = v[i];
          resultStr += renderTemplate(token[2], {
              '.': v[i],
              ...v[i]
          })
      }
      return resultStr;
  }
  ```

至此mustache的基本部分源码就以完成。

