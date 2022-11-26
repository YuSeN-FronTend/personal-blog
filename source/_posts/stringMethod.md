---
title: js常用字符串方法
date: 2022-11-24 10:20
categories: javaScript
---

![](http://106.55.171.176:9000/yusen/Snipaste_2022-11-22_13-08-28.png)

<!-- more-->

## String

### 字符串的初始化

用此方法定义的字符串是对象类型

```javascript
let stringObject = new String('ys');
console.log(typeof stringObject);
```

输出 **object**

### length

获取字符串的长度

```javascript
let stringValue = 'hello world';
console.log(stringValue.length);
```

输出 **11**

### charAt()

js字符串由16位**码元(code unit)**组成对多数字符来说，每16个码元对应一个字符。此方法查找指定索引位置的16位码元。

```javascript
let message = 'abcde';
console.log(message.charAt(2));
```

输出 **c**

### charCodeAt()

此方法可以查看指定码元的字符编码。

```javascript
let message = 'abcde';
console.log(message.charCodeAt(2));
```

输出 **99**  (c对应的字符编码是99)

### fromCharCode()

此方法根据给定的UTF-16码元创建字符串中的字符，并返回接收到所有数值对应的字符拼接起来的字符串。

```javascript
console.log(String.fromCharCode(0x61,0x62,0x63,0x64,0x65));
console.log(String.fromCharCode(97,98,99,100,101));
```

输出均为 **abcde**

### concat()

此方法**不会改变**原字符串

```javascript
let stringValue = 'hello ';
let result = stringValue.concat('world');
console.log(result);
```

输出 **hello world**

### 提取字符串的方法

#### slice() && substring() && substr()

这三个方法**都不会改变**原字符串，并且接受一个到两个参数

```javascript
let stringValue = 'hello world';
console.log(stringValue.slice(3));
console.log(stringValue.substring(3));
console.log(stringValue.substr(3));
console.log(stringValue.slice(3,7));
console.log(stringValue.substring(3,7));
console.log(stringValue.substr(3,7));
```

​	输出

**lo world
lo world
lo world
lo w
lo w
lo worl**

由此可见当参数均为正数时，slice和substring两个参数是截取字符串的起始索引和终止索引，而substr第一个参数是起始索引，第二个参数是向后截取多少位。

```javascript
let stringValue = 'hello world';
console.log(stringValue.slice(-3));
console.log(stringValue.substring(-3));
console.log(stringValue.substr(-3));
console.log(stringValue.slice(3,-4));
console.log(stringValue.substring(3,-4));
console.log(stringValue.substr(3,-4));
```

输出

**rld**
**hello world**
**rld**
**lo w**
**hel**
**""**

由此可见当传入负参数时，slice会将所有负值参数都变成字符串长度加上负参数值，substring会把所有负参数转化为0，substr会把第一个负参数当成字符串长度加上该负值，将第二个负参数转化为0

**注：substr()可能将要废弃**

### 返回字符串位置的方法

#### indexof() && lastIndexOf()

```javascript
let stringValue = 'hello world';
console.log(stringValue.indexOf('o'));
console.log(stringValue.lastIndexOf('o'));
```

输出 **4 7**

顾名思义，indexOf从前向后检索，lastIndexOf从后向前检索。

### 查询字符串中是否包含方法

#### startsWith() && endsWith() && includes()

```javascript
let message = "foobarbaz";
console.log(message.startsWith('foo')); // true
console.log(message.startsWith('bar')); // false
console.log(message.endsWith('baz')); // true
console.log(message.endsWith('bar')); // false
console.log(message.includes('bar')); // true
console.log(message.includes('qux')); // false
```

结果顾名思义，startsWith查找起始，endsWith查找末尾，includes查找全部。

### 清除字符串空格的方法

#### trim() && trimLeft() && trimRight()

均**不会改变**原字符串

```javascript
let message = '     hello world     ';
console.log(message.trim());
console.log(message.trimLeft());
console.log(message.trimRight());
```

输出 

**hello world
hello world     
     hello world**

trim清除左右空格，trimLeft清除左边，trimRight清除右边。

**注：trimLeft和trimRight未来可能会被trimStart和trimEnd所取代。**

### repeat()

复制字符串的方法

```javascript
let message = "yusen "
console.log(message.repeat(3));
```

输出 **yusen yusen yusen **

### padStart() && padEnd()

```javascript
let stringValue = 'foo';
console.log(stringValue.padStart(6));
console.log(stringValue.padStart(9, '.'));
console.log(stringValue.padEnd(6));
console.log(stringValue.padEnd(9, '.'));
```

输出 

***      foo
......foo
foo
foo......**

此两个方法会复制字符串，如果小于指定长度，则在相应一边填充字符直到满足条件为止。

### 转换大小写的方法

#### toLowerCase() && toLocaleLowerCase() && toUpperCase() && toLocaleUpperCase()

```javascript
let stringValue = 'hello world';
console.log(stringValue.toLocaleLowerCase());
console.log(stringValue.toLocaleUpperCase());
console.log(stringValue.toLowerCase());
console.log(stringValue.toUpperCase());
```

输出

 **hello world
HELLO WORLD
hello world
HELLO WORLD**

前两者属于地区特定实现，在少部分地区如(土耳其语)，会有所差异。

###  字符串模式匹配方法

#### match()

接受的参数可以为正则表达式或者RegExp对象

```javascript
let text = "cat, bat, sat, fat";
let pattern = /.at/;
let matches = text.match(pattern);
console.log(matches.index);
console.log(matches);
```

输出 

**0
[ 'cat', index: 0, input: 'cat, bat, sat, fat', groups: undefined ]**

#### search()

接受的参数可以为正则表达式或者RegExp对象

```javascript
let text = "cat, bat, sat, fat";
let pos = text.search(/at/);
console.log(pos);
```

输出 **1**  即所查找字符在字符串中一次出现的位置索引

#### replace()

此方法接收两个参数，第一个参数可以为字符串或者RegExp对象，第二个参数可以为字符串或者函数，如果是字符串就会替换第一个参数对应的字符串，如果需要全局替换，第一个参数必须带上全局标记。

```javascript
let text = "cat, bat, sat, fat";
let result = text.replace('at', 'ond');
console.log(result);
result = text.replace(/at/g, 'ond');
console.log(result);
```

输出

**cond, bat, sat, fat
cond, bond, sond, fond**

###  localeCompare()

此方法会比较两个字符串，并按以下规则返回三个值中的一个。

#### 返回-1

根据字母表的顺序，字符串应该排在字符串参数的前头，则返回-1。

#### 返回0

字符串要与字符串参数相等，则返回0。

#### 返回1

根据字母表的顺序，字符串应该排在字符串参数的后头，则返回1。

```javascript
let stringValue = "yellow";
console.log(stringValue.localeCompare('black')); // 输出 1
console.log(stringValue.localeCompare('yellow')); // 输出 0
console.log(stringValue.localeCompare('zoo')); // 输出 -1
```

