---
title: Typescript学习日记(六)
date: 2017-02-05 18:12:41
tags:
---

本文介绍枚举(`enum`)及类型推论。
<!-- more -->


### 枚举

使用枚举我们可以定义一些有名字的数字常量。 枚举通过 enum关键字来定义。
`typescript` 中的枚举可以理解为是 `javascript` 中的一种特殊的对象——**键为变量名，值为连续的序数**。

```ts
// javascript
const enumOfJS = {
    one: 0,
    two: 1,
    three: 2
}

console.log(enumOfJS.one); // -> 0
console.log(enumOfJS.two); // -> 1
console.log(enumOfJS.three); // -> 2


// typescript
const enum enumOfTS = {
    one,
    two,
    three
}

console.log(enumOfTS.one); // -> 0
console.log(enumOfTS.two); // -> 1
console.log(enumOfTS.three); // -> 2

// 反向枚举
// 在 javascript 并没有这种功能，需要自己遍历
console.log(enumOfTS[0]); // -> one
console.log(enumOfTS[1]); // -> two
console.log(enumOfTS[2]); // -> three

```

**枚举成员**

枚举成员具有一个数字值，它可以是 `常数` 或是 `计算得出的值`;
当满足如下条件时，枚举成员被当作是常数：
* 不具有初始化函数并且之前的枚举成员是常数。 在这种情况下，当前枚举成员的值为上一个枚举成员的值加1。 但第一个枚举元素是个例外。如果它没有初始化方法，那么它的初始值为 `0`。
* 枚举成员使用常数枚举表达式初始化。 常数枚举表达式是TypeScript表达式的子集，它可以在编译阶段求值。 当一个表达式满足下面条件之一时，它就是一个常数枚举表达式：
    * 数字字面量。
    * 引用之前定义的常数枚举成员（可以是在不同的枚举类型中定义的） 如果这个成员是在同一个枚举类型中定义的，可以使用非限定名来引用。
    * 带括号的常数枚举表达式。
    * `+`, `-`, `~` 一元运算符应用于常数枚举表达式。
    * `+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `|`, `^`二元运算符，常数枚举表达式做为其一个操作对象。若常数枚举表达式求值后为 `NaN` 或 `Infinity`，则会在编译阶段报错。

除此之外所有其它的情况，枚举成员都是被当作需要计算得出的值。

```ts
const enum FileAccess {
    // constant members
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    // computed member
    G = "123".length
}
```

**序列化成员**

枚举成员的数值值，默认为从 `0` 开始的一组序数。改变其中的一个，会影响 `之后` 的数值序数，而不会对前面的造成影响。

```ts
// 默认情况
const enum Order {
    A,
    B,
    C,
    D
}

let orders = [Order.A, Order.B, Order.C, Order.D]; // -> [0,1,2,3]

// 更改数值
const enum Order1 {
    A,
    B = 3,
    C,
    D
}

let orders1 = [Order.A, Order.B, Order.C, Order.D]; // -> [0,3,4,5]

// 更改数值为负
const enum Order2 {
    A,
    B = -2,
    C,
    D
}

let orders2 = [Order.A, Order.B, Order.C, Order.D]; // -> [0,-2,-1,0]

// 无序数差值
const enum Order3 {
    A,
    B = 1,
    C,
    D = 2
}

let orders2 = [Order.A, Order.B, Order.C, Order.D]; // -> [0,1,2,2]

```


### 类型推论

`TypeScript` 里，在有些没有明确指出类型的地方，类型推论会帮助提供类型。

```ts
let num = 1;

// num 虽然没有指定为 number 类型，但是 typescript 会自动推断 num 的类型为 number
// let num: number = 1;
```

#### 最佳通用类型

当要从几个表达式中推断类型时，`typescript` 会为我们推断出一个最合适的通用类型。

```ts
let arr = [1,2,3,'4',null,undefined];

// 还记得吗，null 和 undefined 是所有类型的 子类型
// arr 数组中含有 number 和 string 两种类型，则使用联合类型数组
// let arr: (number|string)[] = [1,2,3,'4',null,undefined]
```

#### 上下文类型

`TypeScript` 类型推论也可能按照相反的方向进行，这被叫做“按上下文归类”。按上下文归类会发生在表达式的类型与所处的位置相关时。

这里直接引用官方例子：
```ts
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.button);  //-> Error
}
// TypeScript 类型检查器使用 Window.onmousedown函数 的类型来 推断 右边函数表达式的参数类型

// 通过手动指定的方式，避免类型推断


window.onmousedown = function (mouseEvent: any) {
    console.log(mouseEvent.button); // -> success
}
```





