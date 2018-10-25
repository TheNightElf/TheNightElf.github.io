---
title: Typescript学习日记(一)
date: 2017-01-08 16:14:30
tags:
---

`javascript` 是运行在浏览器中的一门脚本语言，为了尽可能的减少内存的开支所以在设计之初它就是一门 `弱类型` 的语言。这在开发小而简单的应用时是可以为我们带来性能的提升，但是随着项目的扩大 `状态` 变的难以追踪将会引发很多的 `bug` 出现。
例子：
```ts
var test;

typeof test;
// -> 'undefined'

test = 'str';

typeof test;
// -> 'string'

test = 123;

typeof test;
// -> 'number'
```

<!-- more -->

### Typescript
`typescript` 是 `javascript` 的强类型版本。它会在编译期去掉类型和特有语法生成 `javascript` 代码。`typescript` 是 `javascript` 的超集，它支持所有 `javascript语法`，与此同时它又为 `javascript` 添加了 `静态类型检查` 使得 `javacript` 也拥有了 `强类型语言` 的特性。


### Typescript 基础类型

#### 布尔值(boolean)
```ts
let bool: boolean = false;

bool = 123;
// 报错，123是number类型而非boolean类型

bool = true;
// ok，true是一个boolean类型
```

#### 数值(number)
```ts
let num: number = 123;

num = 'str';
// error

num = 456;
// success
```

#### 字符串(string)
```ts
let str: string = 'str';
// ...
```

#### 数组(array)
**定义方式：**
第一种，可以在 `元素类型` 后面接上 `[]`。
```ts
let arr: number[] = [1,2,3];
```

第二种，使用 `数组泛型` 定义。
```ts
let arr: Array<number> = [4,5,6];
```

#### 元组(tuple)
元组类型允许表示一个已知元素数量和类型的数组(其实就是一个数组内包含多个类型的值)。
```ts
let tup: [number, string] = [123,'str'];
```

#### 枚举(enum)
`enum` 类型是对JavaScript标准数据类型的一个补充。 像C#等其它语言一样，使用枚举类型可以为一组数值赋予友好的名字。
**用人话说就是：把一组值放到一个可迭代的对象中，然后给这个对象起了个名字。**

```ts
enum animal {dog,cat,pig}

// 对象取值
let a: animal = animal.dog

// 数组取值
let b = animal[0] // -> dog
let b = animal[1] // -> cat

// 可以手动更改索引
// enum animal {dog = 1, cat, pig}
// let b = animal[1] // -> dog
// let b = animal[2] // -> cat
```

#### 任何类型(any)
若是不清楚变量的具体类型可以指定为 `any`。

```ts
let a: any = false;

a = 'str'; // success
a = 123; // success
```

#### 无类型(void)
与 `any` 相反。一个函数没有返回值的时候通常指定其返回值类型为 `void`。
一个 `void` 类型的变量只能赋值为 `undefined` | `null`。
```ts
function noReturn(): void {
    console.log('这个函数没有返回值')
}

let n: void = null;
// let n: void = undefined;
```

#### null和undefined

`undefined`和`null`两者各自有自己的类型还是叫做`undefined`和`null`。

**默认情况下null和undefined是所有类型的子类型。 就是说你可以把 null和undefined赋值给number类型的变量。**

```ts
let u: undefined = undefined;
let n: null = null;

let num: number = 123;
num = undefined; // success
```

#### 永远不存在的值(never)
例如， never类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型；
**never类型是任何类型的子类型，也可以赋值给任何类型；然而，没有类型是never的子类型或可以赋值给never类型（除了never本身之外）。 即使 any也不可以赋值给never。**

```ts
let n: never; // never 不能被赋值
let num: number = 123;
let a: any;

num = n; // success，never类型是任何类型的子类型

n = 123; // error
n = a; // error
```

```ts
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
    throw new Error(message);
}
```

#### 类型断言

有时候你会遇到这样的情况，你会比TypeScript更了解某个值的详细信息。 通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。

```ts
let str: any = "this is a string";
let strLength1: number;
let strLength2: number;

// 使用 as 断言语法
strLength1 = (str as string).length;

// 使用 <> 断言语法
strLength2 = (<string>str).length;

// 上面这两种断言方式得到的效果完全一致，下面增加的内容是因为模版出了点bug
// 对于语法的解析已经不是js的语法了，不填充以下内容，整块js都无法展示
strLength1 === strLength2; // -> true
strLength1 === strLength2; // -> true
strLength1 === strLength2; // -> true
strLength1 === strLength2; // -> true
strLength1 === strLength2; // -> true
strLength1 === strLength2; // -> true
strLength1 === strLength2; // -> true
```

以上是 `typescript` 所有的基础类型。
