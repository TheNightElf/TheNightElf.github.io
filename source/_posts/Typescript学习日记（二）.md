---
title: Typescript学习日记(二)
date: 2017-01-18 17:59:42
tags:
---

本篇介绍 `typescript` 中接口的使用方式。
<!-- more -->

### 接口(interface)

`TypeScript`的核心原则之一是对值所具有的结构进行类型检查。接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。
基础类型是为简单的变量定义类型，**通常的，接口就是为你所定义的对象(参数、函数等)的添加类型的约束。**
按照 typescript 的默认约定，接口通常是以 `驼峰命名方式，大写字母开头` 的方式定义。

#### 使用方式

```ts
// 定义一个接口约束
interface T {
    num: number;
    str: string
}

// 函数的接受的参数中，num必须为number类型，str必须为string类型
function demo(arg: T) {
    console.log(typeof arg.num)
}

// 也可以直接把约束条件写在函数参数中，为了可读性，推荐在外面定义
// eg:
// function demo(arg: { num: number; str: string }) {
//     console.log(typeof arg.num)
// }

let obj1 = {num: 100, str: 'typescript'}
demo(obj1) // success -> number

let obj2 = {num: '123', str: 'typescript'}
demo(obj2) // error

let obj3 = {num: 123}
demo(obj3) // error 缺少一个 str 参数
```

**注意：定义接口的时候不需要使用 `=`，`{}` 内部必须使用 `分号(;)` 分隔。**


#### 接口的可选属性

接口里的属性不全都是必需的。 有些是只在某些条件下存在，或者根本不存在。

```ts
interface T {
    num: number;
    str?: string;
}

function demo(arg: T) {
    console.log(typeof arg.num)
}

let obj4 = {num: 100, str: 'typescript'}
demo(obj4) // success

let obj5 = {num: 100}
demo(obj5) // success

let obj6 = {str: 'typescript'}
demo(obj6) // error

```

#### 接口的只读属性

这里的 `只读属性` 类似于原生javascript的 `const` 关键字，在赋值之后就不能更改了。

```ts
interface Rd {
    readonly x: number;
    readonly y: string
}

let r: Rd = { x: 100, y: 'str' }

r.x = 123; // -> error
```

#### 只读的数组

TypeScript具有 `ReadonlyArray<T>` 类型，它与 `Array<T>` 相似，只是把所有可变方法去掉了，因此可以确保数组创建后再也不能被修改。

```ts
let arr: ReadonlyArray<number> = [1,2,3,4];

arr[0] = 2; // -> error
arr.push(5); // -> error
arr.length = 10; // -> error

let arr1: Array<number> = arr;
// error, arr 是ReadonlyArray，不能赋值给别的数组
```

#### readonly还是const

最简单判断该用readonly还是const的方法是看要把它做为变量使用还是做为一个属性。 
做为变量使用的话用 const，若做为属性则使用readonly。

#### 额外的属性检查

```ts

interface Pc {
    x?: number;
    y?: string
}

function demo(arg: Pc) {
    // ..
}

demo({x: 100, z: 110})
// error，因为 z 并没有在接口 Pc 中定义

// 使用类型断言改写
demo({x: 100, z: 110} as Pc)

// 告诉 ts 入参 '{x: 100, z: 110}' 都是来自于 Pc 接口的，你别 tmd 再帮我检查了
// ts： 好吧，既然你这么确定，我就听你的。

```

然后上面的这种做法并不是最佳的做法，最佳的做法是添加一个 `占位用的签名`。

```ts

// 我不知道我后面还会传什么属性进来，所以我先用个 propsName 占个位置
interface Pc {
    x?: number;
    y?: string;
    [propName: string]: any;
}


demo({x: 100, z: 110})
// success
```

#### 函数类型

为了使用接口表示函数类型，我们需要给接口定义一个调用签名。 它就像是一个 `只有参数列表和返回值类型` 的函数定义。
**参数列表里的每个参数都需要名字和类型。**

```ts

interface Fn {
    (a: number, b: string): boolean
}

// 定义一个 Fn 函数接口，该函数接受 a: number，b: string 作为参数，返回一个 boolean 值
```

#### 可索引的类型

可索引类型具有一个 `索引签名`，它描述了对象 `索引的类型`，还有相应的索引 `返回值类型`。
共有两种索引签名：字符串和数字。可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型。

```ts
// 1. 指定 索引类型 为 number
interface IndexNum {
    [index: number]: string;
}

let arr: IndexNum = ['hello', 'world'];
console.log(arr[0])

// 2. 指定 索引类型 为 string
interface IndexStr {
    [index: string]: string;
}

let obj: IndexStr = {'x': 'hello', 'y': 'world'}
console.log(obj['x'])

// 3. 同时指定 索引类型 为 number 和 string

interface TotalIndex {
    [index: number]: number;
    at: number; // success
    where: string; // error 使用 where 获取值时返回的是 string 类型
}

// 4. 只读的索引类型

interface RdIndex {
    readonly [index: number]: string;
}

let arr: RdIndex = ['a','b']

arr[1] = 'c'; // error, arr 为只读的索引类型

```

#### '类'类型

和其它的类型约束相似，类类型使用 `implements` 实现接口。

```ts
// 定义方式完全一致
interface C {
    date: Date;
    setTime(d: Date);
}

// 
class Clock implements C {
     // 声明一个 date 属性，且约束该属性类型为 Date
     // 这种写法相当于 var date: Date
     // 只是声明并约束值类型，但并不赋值
    date: Date;
    setTime(d: Date) {
        this.date = d;
    }
    constructor(h: number, m: number) {
        // ..
    }
}
```
关于更多的 `implements` 的用法我目前也不是特别的清楚，所以下面就不再详细叙述了。
等我了解清楚之后再补上这一部分内容，你也可以通过查看[官方文档](https://www.tslang.cn/docs/handbook/interfaces.html)了解。

#### 继承接口

和 `es2015` 中的 `extends` 一样，接口也是可以像类一样用来继承的。

```ts
interface A {
    a: number;
}

interface C {
    c: number;
}

interface B extends A, C {
    b: string;
}

let obj = <B>{};
obj.a = 10;
obj.b = '10';
obj.c = 10;

```

#### 混合类型

`interface` 还可以在一个接口中约束 `javascript` 中多个类型；

```ts
interface Counter {
    a: number; // 约束 a 为 number 类型
    getStr(): void;// 约束 getStr 无返回值；
    (n: number): string;// 约束匿名函数接受一个参数 n 为 number 类型，返回值 string 类型
}

// 函数 getCounter 返回值被 Counter 约束
function getCounter(): Counter {
    let counter = <Counter>function (start: number) { }; // 类型断言
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;

//
// 这个模版有问题，写 ts 的代码就会出问题
// 只能这样了
//
//
//
```

#### 接口继承类

接口继承类时会继承父类所有的属性和方法，包括 `private` 和 `protected` 成员。
这意味着当你创建了一个接口继承了一个拥有 `私有或受保护的成员` 的类时，这个接口类型只能被 `这个类或其子类` 所实现（implement）。

```ts
class Control {
    private state: any;
}

// SelectableControl 含有 Control 所有的属性方法
interface SelectableControl extends Control {
    select(): void;
}

// Button 继承自 Control 含有 Control 所有的属性方法
class Button extends Control implements SelectableControl {
    select() { }
}

// TextBox 继承自 Control 含有 Control 所有的属性方法
class TextBox extends Control {

}

// Image 没有继承自 Control，没有 state 属性，无法实现 SelectableControl 接口
// 错误：“Image”类型缺少“state”属性。
class Image implements SelectableControl {
    select() { }
}
```