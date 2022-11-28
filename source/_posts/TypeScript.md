---
title: TypeScript基础
date: 2022-11-27 12:16
categories: javaScript
---

## TypeScript的含义

TypeScript是以javaScript为基础构建的语言，是javaScript的一个超集。TS是微软开发的，写出的代码相较于JS更容易维护，并且TS完全包含JS的所有特性和功能，又在基础上加了许多的新功能。

![](http://106.55.171.176:9000/yusen/Snipaste_2022-11-28_12-43-42.png)

<!-- more -->

## TypeScript的环境搭建

1.**安装node坏境** 

[node官网下载地址 ⬇](https://nodejs.org/zh-cn/download/ )

2.**使用npm全局安装typeScript**

```bash
npm i -g typescript
```

3.**创建一个ts文件**

4.**使用tsc对ts文件进行编译**

- 进入命令行
- 进入ts文件所在目录
- 执行命令：tsc xxx.ts

## 基本类型

- 类型声明

  - 类型声明是TS非常重要的一个特点

  - 通过类型声明可以指定TS中变量(参数、形参)的类型

  - 指定类型后，给变量赋值时，TS编译器会自动检测值是否符合类型声明，符合则赋值，不符合则报错。

  - 简而言之，类型声明给变量设置了类型，使得变量只能存储某种类型的值

  - 语法：

    ```typescript
    let 变量: 类型;
    
    let 变量: 类型 = 值;
    
    function fn(参数: 类型, 参数: 类型): 类型 {
        ...
    }
    ```

    

- 自动类型判断
  - TS拥有自动的类型判断机制
  - 当对变量的声明和赋值是同时进行的，TS编译器会自动判断变量的类型
  - 所以如果你的变量的声明和赋值是同时进行的，可以省略掉类型声明。

- 类型：

  |  类型   |      例子       |              描述              |
  | :-----: | :-------------: | :----------------------------: |
  | number  |    1,-33,2.5    |            任意数字            |
  | string  |  'hello world'  |           任意字符串           |
  | boolean |   true、false   |       布尔值true或false        |
  | 字面量  |     其本身      |  限制变量的值就是该字面量的值  |
  |   any   |        *        |            任意类型            |
  | unknown |        *        |         类型安全的any          |
  |  void   | 空值(undefined) |      没有值(或undefined)       |
  |  never  |     没有值      |          不能是任何值          |
  | object  | {name:'孙悟空'} |          任意的JS对象          |
  |  array  |    [1, 2, 3]    |           任意JS数组           |
  |  tuple  |     [4, 5]      | 元素，TS新增类型，固定长度数组 |
  |  enum   |   enum(A, B)    |       枚举，TS中新增类型       |

- number

  ```typescript
  let decimal: number = 6;
  let hex: number = 0xf00d;
  let binary: number = 0b1010;
  let octal: number = 0o744;
  let big: bigint = 100n;
  ```

  

- boolean

  ```typescript
  let isDone: boolean = false;
  ```

  

- string

  ```typescript
  let color: string = 'blue';
  color = 'red';
  
  let fullName: string = 'BoB';
  let age: number = 37;
  let sentence: string = `Hello, my name is ${fullName}, I'll be ${age};
  
  ```

  

- 字面量

  - 也可以使用字面量去指定变量的类型，通过字面量可以确定变量的取值范围

    ```typescript
    let color: 'red' | 'blue' | 'black';
    let num1: 1 | 2 | 3 | 4 | 5;
    ```

    

- any

  ```typescript
  let d: any = 4;
  d = 'hello';
  d = true;
  ```

  

- unknown

  ```typescript
  let notSure: unknown = 4;
  notSure = 'hello';
  ```

  

- void

  ```typescript
  let unusable: void = undefined;
  ```

  

- never

  ```typescript
  function error(message: string): never {
      throw new Error(message);
  }
  ```

  

- object(没啥用)

  ```typescript
  let obj: object = {};
  ```

  

- array

  ```typescript
  let list: number[] = [1,2,3];
  let list: Array<number> = [1,2,3];
  ```

  

- tuple

  ```typescript
  let x: [string, number];
  x = ['hello', 10];
  ```

  

- enum

  ```typescript
  enum Color {
  	Red,
  	Green,
  	Blue,
  }
  let c: Color = Color.Green;
  enum Color {
      Red = 1,
      Green = 2,
      Blue = 4,
  }
  let c: Color = Color.Green;
  ```

  

- 类型断言

  - 有些情况下，变量的类型对于我们来说是很明确，但是TS编译器却并不清楚，此时，可以通过类型断言来告诉编译器的类型，断言的两种形式：

    - 第一种

      ```typescript
      let someValue: unknown = 'this is a string';
      let strLength: number = (someValue as string).length;
      ```

      

    - 第二种

      ```typescript
      let someValue: unknown = 'this is a string';
      let strLength: number = (<string>someValue).length;
      ```

      

