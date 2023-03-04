---
title: 面试手写题总结
date: 2023-3-4 9:28
categories: 面试
---

# 1、手写深拷贝

- JSON.parse(JSON.stringify(obj))

  但是有缺点，比如原对象上有function函数，经过此方法拷贝后的对象上面并没有这个方法。

- 循环递归方法   

  ```js
  const obj1 = {
              name: '张三',
              age: 18,
              address: {
                  city: '北京'
              },
              hobby: ['台球', '篮球'],
              fn: function(){
                  console.log(123);
              }
          }
          // const obj2 = obj1
          const obj2 = deepClone(obj1);
          obj2.age = 20;
          obj2.address.city = '天津'
          console.log(obj1);
          console.log(obj2);
  
          function deepClone(obj) {
              if(typeof obj !== 'object' || obj == null) {
                  return obj;
              }
  
              let res = obj instanceof Array ? [] : {};
  
              for(let key in obj) {
                  if(obj.hasOwnProperty(key)){
                      res[key] = deepClone(obj[key]);
                  }
              }
  
              return res;
          }
  
  ```

- lodash  很好用的库 里面就有很完善深拷贝方法

## 2、快排算法

先找一个基点，将小于这个基点的数字放到一个数组，将大于这个几点的数字也放入一个数字，然后递归合并即可。

- 基本有序问题

  就是数组中的值是有顺序的，这时就要给基点设为一个随机数即可

- 基本一致问题

  就是数组中的值都是相等的，这时就要把与基点相同的元素都放在一个数组里，用这个数组和最开始的数组进行长度比较，如若长度相等，说明数组中都为相同值，直接返回即可

下面是代码实现：

```js
var sortArray = function(nums) {
    if(nums.length <= 1) return nums;
    // 取随机数, 优化连续数字, 如[1,2,3,4,5.....]
    // ~~向下取整, 位运算, 速度最快, Math.trunc()和Math.floor()也一样效果
    let pivotIndex = ~~(Math.random()*nums.length);
    // 删除数组任意一项并当作 基准(如果不选随机数, 第11个例子会卡住)
    let pivot = nums.splice(pivotIndex, 1)[0];
    let left = [];
    let right = [];
    let mid = [];
    for(let i = 0; i < nums.length; i++) {
        if(nums[i] < pivot){
            left.push(nums[i])
        }else if (nums[i] > pivot){
            right.push(nums[i])
        } else {
            // 如果和基准相同, 则放进这个数组
            mid.push(nums[i])
        }
    }
    // 此判断用来解决第17个例子(5万个2)
    if(mid.length === nums.length){
        // 用concat同理  push方法要快一些
        mid.push(pivot)
        return mid;
    }
    return sortArray(left).concat([pivot], mid, sortArray(right));
};
```

