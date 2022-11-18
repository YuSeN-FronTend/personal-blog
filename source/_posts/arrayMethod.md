---
title: js常用数组方法
date: 2022-11-17 14:00
categories: javaScript
---
![](http://106.55.171.176:9000/yusen/Snipaste_2022-11-18_10-20-23.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=sttch%2F20221118%2F%2Fs3%2Faws4_request&X-Amz-Date=20221118T022246Z&X-Amz-Expires=432000&X-Amz-SignedHeaders=host&X-Amz-Signature=01d25e6569215e0f8dd668301062f766172c3a3a85402ea83f44116571a99171)

<!-- more -->

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
