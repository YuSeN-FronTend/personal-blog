---
title: js常用数组方法
date: 2022-11-17 14:00
categories: javaScript
---
## 增

### push()

此方法会**改变**原数组。

``` javascript
let arr = [1,2,3];
let newArr = arr.push(4);
console.log(arr, newArr);
```

输出 **[ 1, 2, 3, 4 ] 4**

此方法是向数组后添加一个元素。

### unshift()

此方法会**改变**原数组。

``` javascript
let arr = [1,2,3];
let newArr = arr.unshift(4);
console.log(arr, newArr);
```

输出 **[ 4, 1, 2, 3 ] 4**

此方法是向数组前面添加一个元素。



## 删

### pop()

此方法会**改变**原数组

``` javascript
let arr = [1,2,3];
let newArr = arr.pop();
console.log(arr, newArr);
```

输出 **[ 1, 2 ] 3**

此方法会删除数组中的最后一个元素，并且返回值为被删除的元素。

### shift()

此方法会**改变**原数组

``` javascript
let arr = [1,2,3];
let newArr = arr.shift()
console.log(arr, newArr);
```

输出 **[ 2, 3 ] 1**

此方法会删除数组中的第一个元素，并且返回值为被删除的元素。

## 改(增、删)

### splice()

splice会**改变**原数组

##### 增

```javascript
let arr = [0,1,2,3,4];
let newArr = arr.splice(arr.length,0,5);
console.log(newArr, arr);
```

输出 **[] [ 0, 1, 2, 3, 4, 5 ]**

从数组末尾开始向后截取0个元素，并在此插入新元素5

##### 删

```javascript
let arr = [0,1,2,3,4];
let newArr = arr.splice(0,1);
console.log(newArr, arr);
```

输出 **[ 0 ] [ 1, 2, 3, 4 ]**

从索引0开始向后截取一个元素

##### 改

```javascript
let arr = [0,1,2,3,4];
let newArr = arr.splice(0,1,0.5);
console.log(newArr, arr);
```

输出 **[ 0 ] [ 0.5, 1, 2, 3, 4 ]**

从索引0开始向后截取一个元素，并在此插入元素0.5。

## 查

### slice()

此方法**不会改变**原数组

```javascript
let arr = [0,1,2,3,4];
let newArr = arr.slice(0,3);
console.log(arr,newArr);
```

输出 **[ 0, 1, 2, 3, 4 ] [ 0, 1, 2 ]**

从索引0开始查询到数组的第三项。

## 将数组转换为字符串

### toString()

此方法**不会改变**原数组

```javascript
let arr = [0,1,2,undefined,4];
let newArr = arr.toString();
console.log(arr, newArr);
```

输出 **[ 0, 1, 2, undefined, 4 ] 0,1,2,,4**

### join()

join方法**不会改变**原数组

```javascript
let arr = [0,1,2,undefined,4];
let newArr = arr.join('|');
console.log(arr, newArr);
```

输出 **[ 0, 1, 2, undefined, 4 ] 0|1|2||4**

join方法接收一个参数是连接数组元素的分隔符

## 数组拼接

### concat()

concat方法**不会改变**原数组

```javascript
let arr = [0,1,2,3,4];
let arr2 = [5,6,7];
let newArr = arr.concat(arr2);
console.log(arr,arr2,newArr);
```

输出 **[ 0, 1, 2, 3, 4 ] [ 5, 6, 7 ] [ 0, 1, 2, 3, 4, 5, 6, 7 ]**

## 检测数组中是否包含某一项

### indexOf()

indexOf**不会改变**原数组

```javascript
let arr = [0,1,2,3,4,5,'morning',7,'morning'];
console.log(arr.indexOf('morning'));
console.log(arr.indexOf('morning',7));
```

输出 **6 8**

indexOf接收的第一个参数是要查找的元素值，默认返回第一次出现的索引，第二个参数是从此数索引开始查找。

### lastIndexOf()

lastIndexOf**不会改变**原数组

```javascript
let arr = [0,1,2,3,4,5,'morning',7,'morning'];
console.log(arr.lastIndexOf('morning'));
```

输出 **8**

返回元素最后后一次出现的索引。

### includes()

includes方法**不会改变**原数组

```javascript
let arr = [0,1,2,3,4,'includes',5,6];
console.log(arr.includes('includes'));
console.log(arr.includes('includes',6));
```

输出 **true false**

第一个参数是要被检索的元素，第二个参数是从索引第几项开始检索，如果没填默认为0，但需要注意的是此方法和indexOf的区别是此方法返回的是布尔值，而indexOf返回的是对应项的索引。

## 数组排序

### reverse()

此方法会**改变**原数组

```javascript
let arr = [0,1,2,3,4];
let newArr = arr.reverse();
console.log(arr, newArr);
```

输出 **[ 4, 3, 2, 1, 0 ] [ 4, 3, 2, 1, 0 ]**

此方法使数组倒序

### sort()

此方法会**改变**原数组

##### 升序

```javascript
let arr = [12,45,6,21,51,81];
let newArr = arr.sort((a,b) => (a-b));
console.log(newArr, arr); 
```

输出 **[ 6, 12, 21, 45, 51, 81 ] [ 6, 12, 21, 45, 51, 81 ]**

##### 降序

```javascript
let arr = [12,45,6,21,51,81];
let newArr = arr.sort((a,b) => (b-a));
console.log(newArr, arr); 
```

输出 **[ 81, 51, 45, 21, 12, 6 ] [ 81, 51, 45, 21, 12, 6 ]**

## 遍历数组的方法

### forEach()

此方法可以对数组进行循环遍历，内部是一个回调函数，里面有三个参数分别为，数组中对应元素的值，对应元素的索引，和数组本身

```js
let arr = ['jfz', 'zj', 'zpz'];

arr.forEach((item, index, self) => {
    console.log(item, index, self); 
    // jfz 0 [ 'jfz', 'zj', 'zpz' ]
    // zj 1 ['jfz', 'zj', 'zpz']
    // zpz 2 ['jfz', 'zj', 'zpz']
})
```

## 数组内部添加函数

### map()

该方法不会改变原数组，会返回一个新的数组，map方法按照数组的原始数据依此处理数组

```js
let arr = [1,2,3,4,5];
let arr2 = arr.map((item) => {
    return item**2;
})
console.log(arr2); // [ 1, 4, 9, 16, 25 ]
```

### filter()

该方法**不改变**原数组

顾名思义就是过滤器，返回满足过滤条件的数组

```js
let arr = [1,2,3,4,5,6,7,8,9,10];

let arr2 = arr.filter((item, index) => {
    return item % 2 === 0 || index >= 8;
})
console.log(arr2); // [ 2, 4, 6, 8, 9, 10 ]
```

### every()

该方法**不改变**原数组

判断数组中每一项是否满足条件，只有全部满足才返回true

```js
let arr = [2,4,6,7];
let arr2 = arr.every((item) => {
    return item >=2;
})
let arr3 = arr.every((item) => {
    return item % 2 === 0;
})

console.log(arr2); // true
console.log(arr3); // false
```

### some()

判断数组中是否有满足条件的值，有一项满足就会返回true

```js
let arr = [2,4,6];

let arr2 = arr.some((item) => {
    return item % 2 !== 0;
})

let arr3 = arr.some((item) => {
    return item > 5;
})
console.log(arr2); // false
console.log(arr3); // true
```

### reduce()和reduleRight()

这两个方法都会迭代数组的所有项(累加器)，然后构建返回一个最终的值。

reduce是从前向后，reduceRight是从后向前

接收四个参数：前一个值，当前值，当前项的索引，原数组对象

```js
let arr = [1,2,3,4,5];
let sum = arr.reduce((acc, cur, index, array) => {
    return acc + cur;
},10); // 数组一开始加的默认值，不设置默认为0
let sumRight = arr.reduceRight((acc, cur) => {
    return acc + cur;
},10);
console.log(sum); // 25
console.log(sumRight); // 25
```

### find()和findIndex()

这两个方法均接收两个参数：一个回调函数，一个可选值用于指定回调函数内部的this

回调函数会接受三个参数：当前项的元素值，当前项的索引，数组对象本身

区别是find会返回匹配的值，findIndex会返回匹配位置的索引

```js
let arr = ['zj', 'jfz', 'zpz'];

let arr2 = arr.find((value,index,array) => {
    return value === 'zj'
})
let arr3 = arr.findIndex((value, index, array) => {
    return value === 'zj'
})
console.log(arr2); // zj
console.log(arr3); // 0
```

## ES6中新增方法

### Array.of()

此方法是创建数组的方法

```js
let arr = Array.of(1,2);
console.log(arr); // [ 1, 2 ]
```

### Array.from()

可以将非数组对象转换为数组

```js
let map = new Map([[1, 2], [3, 4]])
console.log(map instanceof Array); // false

let arr = Array.from(map);
console.log(arr instanceof Array); // true
```

在手写Promise.all()方法时，会用到此方法把传过来的非数组对象转换为数组

### fill()

该方法会**改变**原数组

可以使用特定值填充数组的一个或多个元素

可以传入的三个参数分别为：填充数值，起始位置参数，结束位置参数(不包括结束位置的那个元素)

当传入一个参数是，会填充所有数组元素：

```js
let arr = [6,6,6,6,6];
arr.fill(1);
console.log(arr); // [ 1, 1, 1, 1, 1 ]
```

当传入两个参数时，会填充包括第二个参数索引即后面的元素：

```js
let arr = [6,6,6,6,6];
arr.fill(1,2);
console.log(arr); // [ 6, 6, 1, 1, 1 ]
```

当传入三个参数时，包括起始位置索引但不包括结束位置索引(和slice方法类似)

```js
let arr = [6,6,6,6,6];
arr.fill(1,2,4);
console.log(arr); // [ 6, 6, 1, 1, 6 ]
```

### copyWithin()

该方法会**改变**原数组

该方法用于从数组的指定位置拷贝到另一个数组韦志中

该方法接收三个参数：从索引几开始粘贴，从索引几开始复制，遇到索引几时停止

```js
let arr = [1,1,2,2,3,3];
arr.copyWithin(3,0);
console.log(arr); // [ 1, 1, 2, 1, 1, 2 ]
```

```js
let arr = [1,1,2,2,3,3];
arr.copyWithin(3,0,2);
console.log(arr); // [ 1, 1, 2, 1, 1, 3 ]
```

### flat() 和 flatMap()

flat方法会按照一个可指定的深度递归遍历数组，并返回一个新数组，**不改变**原数组

参数为嵌套数组的结构深度，默认为1

```js
let arr1 = [1,2,3,[4,5]];
let arrFlat1 = arr1.flat();
console.log(arrFlat1); // [ 1, 2, 3, 4, 5 ]

let arr2 = [1,2,3,[[[4,5]]]];
let arrFlat2 = arr2.flat(2);
console.log(arrFlat2); // [ 1, 2, 3, [ 4, 5 ] ]

// 使用Infinity可展开任意深度的数组
let arr3 = [1,[2,[3,[4,[5,[6,[7]]]]]]];
let arrFlat3 = arr3.flat(Infinity);
console.log(arrFlat3); // [ 1, 2, 3, 4, 5, 6, 7 ]

// 扁平化处理数组，会跳过空位
let arr4 = [1,2, , 4];
let arrFlat4 = arr4.flat();
console.log(arrFlat4); // [ 1, 2, 4 ]
```

flatMap方法对原数组的每个成员执行一个函数，相当于执行Array.prototype.map()，然后对返回值组成的数组执行flat()方法

该方法返回一个新数组，**不改变**原数组

```js
let arr = [2,3,4];
let arrFlatMap = arr.flatMap((item) => {
    return [item, item*2]
})

console.log(arrFlatMap); // [ 2, 4, 3, 6, 4, 8 ]
```

### entries(),keys()和values()

它们用于遍历数组，都返回一个遍历器对象，可以用for-of循环进行遍历

区别是keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历

```js
let arr = [1,2,3];
for(let key of arr.keys()) {
    console.log(key); // 0 1 2 
}

for(let value of arr.values()) {
    console.log(value); // 1 2 3
}

for(let item of arr.entries()) {
    console.log(item); // [ 0, 1 ] [ 1, 2 ] [ 2, 3 ]
}
```

