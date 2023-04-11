---
title: AST抽象语法树源码
date: 2023-4-10 13:48
categories: vue
---

# 指针思想

给定一个字符串，返回最大连续字符长度

```js
let str = 'aaaaaaaabbbbbbbbbbccccccccccdddd';
        let i = 0;
        let j = 1;
        let len = 0;
        console.log(str[32]);
        while(i < str.length) {
            if(str[i] !== str[j]){
                if( j - i >= len) {
                    len = j - i;
                }
                i = j;
            }
            j++;
        }
        console.log(len);
```

# 递归思想

- 斐波那契数列(f(n) = f(n - 1) + f(n - 2))

  ```js
  // 创立缓存对象，解决重复计算问题
  let cache = {};
      function fib(n) {
          console.log("进入了fib" + n);
          if(cache.hasOwnProperty(n)) {
              return cache[n];
          } 
          let v = n === 0 || n === 1 ? 1 : fib(n - 1) + fib(n - 2);
          cache[n] = v;
          return v
      }
  ```

- 遍历一个数组，如果是数字就变为{value: 数字}格式，见到数组就放到children数组中

  ```js
  let arr = [1,2,[3,[4,5],6],7,8,9];
          function handle(item) {
              if(typeof item === 'number') {
                  return {
                      value: item
                  }
              } else if(Array.isArray(item)) {
                  return {
                      children: item.map(_item => handle(_item))
                  }
              }
          }
  ```

# 栈

以下代码是很经典的一个栈题目，创立两个栈和指针移动，通过正则判断指针指到的字符串类型。大体实现思想如下

- 如果当前指到的类型为数字，并且数字后面是默认为'['，需要把数字放到栈`stackMath`中，并且在`stackStr`中加入空字符串，然后指针移动
- 如果当前指到的类型为[]内部的内容，则把`stackStr`中的最后一项变成当前内容，然后指针移动
- 如果当前指到的内容为']'，则需要弹出`stackMath`栈顶的数字，并且弹出`stackStr`栈顶的内容并重复和前面弹出的数字相同的次数，然后放到当前`stackStr`的栈顶上，然后指针移动
- 当全部移动过后，两个栈中都还剩最后一条数据，只需把`stackStr`中的数据重复`stackMath`中数据对应的次数即可

```js
let str = '3[2[a]2[b]]';
        function stack(str) {
            let stackMath = [];
            let stackStr = [];
            let index = 0;
            let rest = str;
            while(index < str.length - 1) {
                rest = str.substring(index);
                // 判断是否为数组和[开头
                if(/^\d+\[/.test(rest)) {
                    let times = Number(rest.match(/^(\d+)\[/)[1]);
                    stackMath.push(times);
                    stackStr.push("");
                    index += times.toString().length + 1
                } else if(/^\w+\]/.test(rest)) {
                    // 判断是否是字母
                    let word = rest.match(/^(\w+)\]/)[1];
                    stackStr[stackStr.length - 1] = word;
                    index += word.length;
                } else if(rest[0] === ']') {
                    let times = stackMath.pop();
                    let word = stackStr.pop();
                    stackStr[stackStr.length - 1] += word.repeat(times);
                    index++;
                }
            }
            return stackStr[0].repeat(stackMath[0])
        }
        console.log(stack(str)); // aabbaabbaabb
```

# AST抽象语法树

和上述堆方法实现类似

- 建立parse.js

  ```js
  // parse函数， 主函数
  
  export default function parse(templateString) {
      // 指针
      let index = 0;
      // 剩余部分
      let rest = '';
      // 开始标记
      let startRegExp = /^\<([a-z]+[1-6]?)\>/;
      // 结束标记
      let endRegExp = /^\<\/([a-z]+[1-6]?)\>/;
      // 抓取结束标记前的文字
      let wordRegExp = /^([^\<]+)\<\/([a-z]+[1-6]?)\>/;
      // 建立栈
      let stack1 = [];
      let stack2 = [{'children':[]}];
      while(index < templateString.length - 1) {
          rest = templateString.substring(index);
          // 识别遍历到这个字符，是不是一个开始标签
          if (startRegExp.test(rest)){
              let tag = rest.match(startRegExp)[1];
              stack1.push(tag);
              stack2.push({ 'tag': tag, 'children': []});
              // 得到attrs的长度
              const attrsStringLength = attrsString != null ? attrsString.length : 0;
              index += tag.length + 2 + attrsStringLength;
          } else if(endRegExp.test(rest)){
              let tag = rest.match(/^\<\/([a-z]+[1-6]?)\>/)[1];
              let pop_tag = stack1.pop();
              if (tag === pop_tag) {
                  let pop_arr = stack2.pop();
                  if(stack2.length > 0) {
                      stack2[stack2.length - 1].children.push(pop_arr);
                  }
              } else {
                  throw new Error('此标签不是封闭标签')
              }
              index += tag.length + 3;
          } else if(wordRegExp.test(rest)){
              let word = rest.match(wordRegExp)[1];
              // 看word是不是全空
              if(!/^\s+$/.test(word)) {
                  // 不是全是空
                  stack2[stack2.length - 1].children.push({'text': word, 'type': 3})
              }
  
              index += word.length
          }else {
              index++
          }
      }
      return stack2[0].children[0];
  }
  ```

  

