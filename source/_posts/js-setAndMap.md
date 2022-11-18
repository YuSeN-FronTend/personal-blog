---
title: Set&&Map
date: 2022-11-18 10:30
categories: javaScript
---
![](http://106.55.171.176:9000/yusen/Snipaste_2022-11-18_12-03-28.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=sttch%2F20221118%2F%2Fs3%2Faws4_request&X-Amz-Date=20221118T040357Z&X-Amz-Expires=432000&X-Amz-SignedHeaders=host&X-Amz-Signature=58a0fc0395c0f55ab6d64976a57ed0844c9df6bd4d332207138ad58aa737a2e8)

<!-- more -->

## Map

### 初始化Map

``` javascript
const m1 = new Map([['a', 111], ['b', 222]]);
console.log(m1);
```

输出 **Map(2) { 'a' => 111, 'b' => 222 }**

### 获取、查找、添加、删除元素

``` javascript
const m1 = new Map([['a', 111], ['b', 222]]);
// 获取元素 返回value
console.log(m1.get('a')); // 输出 111
// 查找元素 返回布尔值
console.log(m1.has('b')); // 输出 true
// 添加元素 返回全数组
console.log(m1.set('c', 333)); // 输出 Map(3) { 'a' => 111, 'b' => 222, 'c' => 333 }
// 删除元素 返回布尔值
console.log(m1.delete('c')); // 因为map数组中包含键值'c' 所以输出 true
console.log(m1); // 输出 Map(2) { 'a' => 111, 'b' => 222 }
```

### size

可以获取map中所包含的所有键值对的个数

``` javascript
const m1 = new Map([['a', 111], ['b', 222]]);
console.log(m1.size);
```

输出 **2**

### clear()

``` javascript
const m1 = new Map([['a', 111], ['b', 222]]);
m1.clear();
console.log(m1);
```

输出 **Map(0) {}**

### 遍历方法

#### keys() 

返回键名的遍历器

```javascript
const m1 = new Map([['a', 111], ['b', 222]]);
console.log(m1.keys());
```

输出 **[Map Iterator] { 'a', 'b' }**

#### values()

返回键值的遍历器

```javascript
const m1 = new Map([['a', 111], ['b', 222]]);
console.log(m1.values());
```

输出 **[Map Iterator] { 111, 222 }**

#### entries()

返回键值对的遍历器

```javascript
const m1 = new Map([['a', 111], ['b', 222]]);
console.log(m1.entries());
```

输出 **[Map Entries] { [ 'a', 111 ], [ 'b', 222 ] }**

#### forEach()

使用回调函数可以遍历每个成员

```javascript
const m1 = new Map([['a', 111], ['b', 222]]);
m1.forEach((value,key,m1)=>{
    console.log(value, key, m1);
})
```

输出 **111 a Map(2) { 'a' => 111, 'b' => 222 }
         222 b Map(2) { 'a' => 111, 'b' => 222 }**

### map和对象的相互转换

```javascript
const obj = {}
const m1 = new Map([['a', 111], ['b', 222]]);
for(let [key,value] of m1) {
    obj[key] = value
}
console.log(obj);
```

输出 **{ a: 111, b: 222 }**

## Set

最常用的场景即为数组去重

### 初始化

```javascript
let s1 = new Set([1,2,3,4,2,2,3])
console.log(s1);
```

输出 **Set(4) { 1, 2, 3, 4 }**

### 实例对象的属性

#### size

```javascript
let s1 = new Set([1,2,3,4,2,2,3])
console.log(s1.size);
```

输出 **4**

### 实例对象的方法

#### add()

向set添加某个值，返回值为set本身

```javascript
let s1 = new Set([1,2,3,4,2,2,3])
console.log(s1.add(5));
```

输出 **Set(5) { 1, 2, 3, 4, 5 }**

#### delete()

删除一个值并返回一个布尔值

```javascript
let s1 = new Set([1,2,3,4,2,2,3])
console.log(s1.delete(4)); // 输出 true
console.log(s1); // 输出 Set(3) { 1, 2, 3 }
```

#### has()

返回一个布尔值，表示set中是否存在此元素

```javascript
let s1 = new Set([1,2,3,4,2,2,3])
console.log(s1.has(2));
```

输出 **true**

##### clear()

清空所有成员，没有返回值

```javascript
let s1 = new Set([1,2,3,4,2,2,3])
s1.clear();
console.log(s1);
```

输出 **Set(0) {}**

### 遍历方法

#### keys() && values()

因为Set结构没有键名，所以两者的返回值是相同的

```javascript
let s1 = new Set([1,2,3,4,2,2,3])
console.log(s1.keys());
console.log(s1.values());
```

输出均为 **[Set Iterator] { 1, 2, 3, 4 }**

#### entries()

返回所有键值对的遍历器

```javascript
let s1 = new Set([1,2,3,4,2,2,3])
console.log(s1.entries());
```

输出 **[Set Entries] { [ 1, 1 ], [ 2, 2 ], [ 3, 3 ], [ 4, 4 ] }**

#### forEach()

使用回调函数遍历每个成员

```javascript
let s1 = new Set([1,2,3,4,2,2,3])
s1.forEach((value,key,s1)=>{
    console.log(value, key, s1);
})
```

输出

**1 1 Set(4) { 1, 2, 3, 4 }
2 2 Set(4) { 1, 2, 3, 4 }
3 3 Set(4) { 1, 2, 3, 4 }
4 4 Set(4) { 1, 2, 3, 4 }**

