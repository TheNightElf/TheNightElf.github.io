---
title: Typescript学习日记(七)
date: 2017-02-12 14:23:14
tags:
---

本文主要介绍 `typescript` 各个类型的兼容性。
<!-- more -->

### 类型兼容性

TypeScript里的类型兼容性是基于结构子类型的。 结构类型是一种只使用其成员来描述类型的方式。 它正好与名义（nominal）类型形成对比。
**如果不了解，请忽略上面这段话**。

#### 比较两个变量

这里直接使用官方的例子：

```ts
interface Named {
    name: string;
}

let x: Named;
let y = { name: 'Alice', location: 'Seattle' }; // y 的类型推断为 {name: string, location: string}
x = y; // -> success，x 中拥有一个 name: string 类型， 符合 y 中 name 的数据类型；location 则继续沿用 y 的。
y = x; // -> error，x 中的 name 属性类型符合 y 中 name 属性，但 x 缺少 location 属性，无法赋值给 y
```

**总结**：
赋值对象(y)中包含被赋值对象(x)的属性类型，且被赋值对象(x)中的属性在赋值对象(y)中存在，则可以正常成功进行赋值。

#### 比较两个函数

比较两个函数和比较变量的比较方式大致一致；比较函数时，比较的是 `函数的参数` 和 `函数的返回值` ，且比较规则于比较变量规则相同。

```ts
// 比较参数
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // -> success 同上
x = y; // -> error 同上
```

```ts
// 比较返回值
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // -> success 同上
y = x; // -> error 同上
```

可以看出，两个函数比较的时候比较的仅仅是 `函数参数` 和 `函数返回值`，比较的规则依然沿用 `比较变量` 的结论。


#### 函数参数双向协变

这里直接引用官方的说法。
当比较函数参数类型时，只有当源函数参数能够赋值给目标函数或者反过来时才能赋值成功。 这是不稳定的，因为调用者可能传入了一个具有更精确类型信息的函数，但是调用这个传入的函数的时候却使用了**不是那么精确的类型信息**。 
实际上，这极少会发生错误，并且能够实现很多 `JavaScript` 里的常见模式。

```ts
enum EventType { Mouse, Keyboard }

interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

function listenEvent(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// 常见的使用方式
// e: MouseEvent => {x: number; y: number; timestamp: number;}
listenEvent(EventType.Mouse, (e: MouseEvent) => console.log(e.x + ',' + e.y));

// 并不是一个很好的代替品
// e: Event => { timestamp: number; }
listenEvent(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + ',' + (<MouseEvent>e).y));
listenEvent(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + ',' + e.y)));

// 这也不是很好
// e: number
listenEvent(EventType.Mouse, (e: number) => console.log(e));
```


#### 可选参数及剩余参数

这里直接引用官方的说法。
比较函数兼容性的时候，可选参数与必须参数是可互换的。 **源类型上有额外的可选参数不是错误，目标类型的可选参数在源类型里没有对应的参数也不是错误**。
当一个函数有剩余参数时，它被当做 `无限个可选参数`。
这对于类型系统来说是不稳定的，但从运行时的角度来看，可选参数一般来说是不强制的，因为对于大多数函数来说相当于传递了一些  `undefinded`。

```ts
function invokeLater(args: any[], callback: (...args: any[]) => void) {
    /* ... Invoke callback with 'args' ... */
}

// Unsound - invokeLater "might" provide any number of arguments
invokeLater([1, 2], (x, y) => console.log(x + ', ' + y));

// Confusing (x and y are actually required) and undiscoverable
invokeLater([1, 2], (x?, y?) => console.log(x + ', ' + y));
```

#### 函数重载

官方文档上的说明很好理解：
对于有重载的函数，源函数的每个重载都要在目标函数上找到对应的函数签名。 这确保了目标函数可以在所有源函数可调用的地方调用。


### 枚举

枚举类型与数字类型兼容，并且数字类型与枚举类型兼容。不同枚举类型之间是不兼容的。

```ts
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
status = Color.Red; // -> error, 不同枚举类型
```

### 类

类与对象字面量和接口差不多，但有一点不同：**类有静态部分和实例部分的类型**。 比较两个类类型的对象时，只有 `实例的成员会被比较`，`静态成员` 和 `构造函数` 不在比较的范围内。

```ts
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

let a: Animal;
let s: Size;

a = s;  // -> success，如果比较构造函数，这里就会抛出错误
s = a;  // -> success
```

#### 私有成员
私有成员会影响兼容性判断。 当类的实例用来检查兼容时，如果目标类型包含一个私有成员，那么**源类型必须包含来自同一个类的这个私有成员**。 这允许子类赋值给父类，但是不能赋值给其它有同样类型的类。


#### 泛型

因为 `TypeScript` 是结构性的类型系统，类型参数只影响使用其做为类型一部分的结果类型。


```ts
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // success, x 与 y 互相兼容，并不排斥
```

```ts
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // error, x 为 number 时，date 为 number；y 为 string 时，date 为 string。x、y不兼容。
```

对于没指定泛型类型的泛型参数时，会把所有泛型参数当成 `any` 比较。
如下：

```ts
let identity = function<T>(x: T): T {
    // ...
}

let reverse = function<U>(y: U): U {
    // ...
}

identity = reverse;  // Okay because (x: any)=>any matches (y: any)=>any
```

### 总结
在 `TypeScript` 里，有两种类型的兼容性：子类型与赋值。 
它们的不同点在于，赋值扩展了子类型兼容，允许给 `any` 赋值或从 `any` 取值和允许数字赋值给枚举类型或枚举类型赋值给数字。