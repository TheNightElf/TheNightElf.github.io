---
title: Typescript学习日记(五)
date: 2017-01-30 16:40:56
tags:
---

使用泛型来创建可重用的组件，一个组件可以支持多种类型的数据，这样就可以 `以自己的数据类型` 来使用组件。
<!-- more -->

### 泛型

这里直接使用 `typescript` 官方的例子
```ts
// 传入 arg: number 返回 arg: number
// 但是这只能处理 arg: number 这么一种情况
function identity(arg: number): number {
    return arg;
}
```

**重载**

```ts
// 使用重载来约束函数又会显得比较啰嗦
// 不写完又无法全面的进行类型检查
function identity(arg: string): string;
function identity(arg: number): number;
// 其它类型...

function identity(arg): any {
    return arg
}
```

**泛型写法**

```ts
// 这句话的意思
// 声明 identity 函数并添加一个 类型变量 T
// 告诉它我传入了一个 参数类型 为 T 的参数
// 你不需要管我这个 T 传递的是什么类型
// 只要帮我检查返回值的类型也是 T 就行了
function identity<T>(arg: T): T {
    return arg
}

// 调用方式
// 1. 传入 T 的类型
let output = identity<string>('str');


// 2. 直接调用，借助类型推断来判断 T 的类型
let output = identity('str');
```

这就是典型的泛型函数，不同于 `any` 它不会丢失类型信息。
但是在使用泛型变量的时候，我们必须要注意的是泛型变量 `T` 指的是所有类型。必须确定它能访问的函数和属性，否则会报错。
对于添加变量类型被命名为 `T` 是一种成俗的约定；当然我们也可以定义成别的，只要前后能够统一就行。

#### 泛型类型

**泛型接口**

```ts
interface TI {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg
}

let output: TI = identity;
```

我们还可以将泛型变量 `T` 放在 `interface` 上，从而省略内部的泛型变量 `T`;
官方对这一段内容写了一大篇，我没感觉出有多大用处。

```ts
interface TI<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg
}

let output: TI<number> = identity;
```

**泛型约束的继承使用**

```ts
interface L {
    length: number;
}

// 泛型变量 T 继承自接口 L
// 泛型变量 T 必定含有一个 length 属性
function identity<T extends L>(arg: T): T {
    console.log(arg.length)
    return arg.length
}

identity(3); // -> error，现在泛型函数 identity 的参数被 L 约束

identity({value: 3, length: 1}) // -> success

```

**在泛型约束中使用类型参数**
这里直接引用官方的例子：

```ts
function getProperty(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // -> success
getProperty(x, "m"); // -> error，x 里面没有 'm' 这个 key
```

**在泛型里使用类类型**
这里直接引用官方的例子：

```ts
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string;
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

// createInstance 
// 参数: c: new () => A 指 参数c 为一个 可以实例化的构造函数
// 且实例化之后的函数类型被 变量类型 A
function createInstance<A extends Animal>(c: new () => A): A {
    return new c();
}

createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;

```


#### 泛型类

```ts
class Demo<T> {
    prop: T;
    add: (x: T, y: T) => T;
}

let d = new Demo<number>();
d.prop = 10;
d.add = (x, y) => x + y;

// 还是老样子
//
//

```

