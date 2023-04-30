---
title: js位运算
date: 2023-4-28 9:33
categories: javaScript
---

# js位运算

平时的基本运算都是将数字转换成二进制再进行计算，而位运算是直接进行二进制计算。位运算是最低级的操作，但也是最快的操作，位运算的特性还能实现一些算法。

## 位运算表格

位运算分为两种，**位逻辑运算符**与**位移运算符**

### 位逻辑运算符

| 运算符 | 含义 | 0 0情况 | 0 1 情况 | 1 0 情况 | 1 1 情况 |
| ------ | ---- | ------- | -------- | -------- | -------- |
| &      | 与   | 0       | 0        | 0        | 1        |
| \|     | 或   | 0       | 1        | 1        | 1        |
| ~      | 取反 | 1       | 0        | 1        | 0        |
| ^      | 异或 | 0       | 1        | 1        | 0        |

### 位移运算符

| 运算符 | 含义         |
| ------ | ------------ |
| <<     | 左移位       |
| >>     | 右移位       |
| >>>    | 无符号右移位 |

## 位运算符基础

### &

按位与，如果两个相应的二进制位都为1，则该位结果为1，否则为0

### |

按位或，如果两个相应的二进制位全为0，则该位结果为0，否则为1

### ^

按位异或，如果两个对应的二进制位相同，则该位结果为0，否则为1

### ~

取反，对一个二进制数按位取反，0变1，1变0

### <<

左移，用来将一个数的二进制位全部左移N位，右边补0

### >>

右移，用来将一个数的二进制位全部右移N位，高位补0

## 使用位运算

1. **~求反**

   将运算符后二进制数进行反转

   ```js
   let math = 7; 
   console.log(math.toString(2)); // 111
   console.log(~math); // -8
   ```

2. **<<左移**

   将二进制个位数右移若干位，高位丢弃，低位补零

   ```js
   let math = 6;
   console.log(math.toString(2)); // 110
   let mathLeft = math << 1
   console.log(mathLeft); // 12
   console.log(mathLeft.toString(2)); // 1100
   ```

3. **\>>右移**

   各二进制位全部右移若干位，正数高位补0，负数高位补1，低位丢弃

   ```js
   let math = -12;
   console.log(math.toString(2)); // -1100
   let mathRight = math >> 1;
   console.log(mathRight); // -6
   console.log(mathRight.toString(2)); // -110
   ```

4. **\>\>\>无符号右移**

   各二进制高位补0，低位丢弃

   ```js
   let math = -12;
   console.log(math.toString(2)); // -1100
   let mathRight = math >>> 1;
   console.log(mathRight); // 2147483642
   console.log(mathRight.toString(2)); // 1111111111111111111111111111010
   ```

   因为将-12的二进制位向右移动两位，高位补上一个0，低位丢弃，得此结果

5. **&位于**

   两个二进制数对应位全为1则为1，否则为0

   ```js
   let math1 = 7;
   let math2 = 19;
   console.log(math1.toString(2)); // 00111
   console.log(math2.toString(2)); // 10011
   console.log((math1 & math2).toString(2)); // 00011
   ```

   **特殊用途**

   - 清零(将一个单位与0进行位运算结果为0)
   - 取一个数的指定位(例如取num = 1010 1101的低四位 则将num&0xF得到0000 1101)
   - 判断奇偶性：用if(a&1) == 0) 代替if(a % 2 == 0) 来判断a是不是偶数

6. **|位或**

   两个二进制数对应位全为0则为0，否则为1

   ```js
   let math1 = 7;
   let math2 = 19;
   console.log(math1.toString(2)); // 00111
   console.log(math2.toString(2)); // 10011
   console.log((math1 | math2).toString(2)); // 10111
   ```

7. **^异或**

   当运算符两边相同位置都是相同，结果返回0，不相同返回1

   ```js
   let math1 = 7;
   let math2 = 19;
   console.log(math1.toString(2)); // 00111
   console.log(math2.toString(2)); // 10011
   console.log((math1 ^ math2).toString(2)); // 10100
   ```

   **特殊用途**

   ​	交换两个数，通常要用临时变量，但是使用异或可以不用使用临时变量

   ```js
   let math1 = 7;
   let math2 = 19;
   math1 ^= math2;
   math2 ^= math1;
   console.log(math2); 7
   math1 ^=math2
   console.log(math1); 19
   ```

   