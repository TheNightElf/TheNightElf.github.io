---
title: Typescript学习日记（八）
date: 2017-02-23 17:41:17
tags:
---

本文主要介绍 `typescript` 几种高级类型及相关内容。
<!-- more -->

### 高级类型

#### 交叉类型(Intersection Types)

交叉类型是将多个类型合并为一个类型。这让我们可以把现有的多种类型叠加到一起成为一种类型，它包含了所需的所有类型的特性。
我们大多是在混入（mixins）或其它不适合典型面向对象模型的地方看到交叉类型的使用。
简单的说：将多个类型进行并集操作，目标类型必须满足合并之后的所有类型。

```ts
// 添加类型 T、U，指定返回值类型必须符合 T 且符合 U
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U>{};
    for (let id in first) {
        (<any>result)[id] = (<any>first)[id];
    }
    for (let id in second) {
        if (!result.hasOwnProperty(id)) {
            (<any>result)[id] = (<any>second)[id];
        }
    }
    return result;
}

class Person {
    constructor(public name: string) { }
}
interface Loggable {
    log(): void;
}
class ConsoleLogger implements Loggable {
    log() {
        // ...
    }
}

// 混合 Person 类和 ConsoleLogger 类
// 生成的 jim 包含两个父类的所有属性和方法
var jim = extend(new Person("Jim"), new ConsoleLogger());
var n = jim.name;
jim.log();
```

#### 联合类型(Union Types)

如果把 `交叉类型` 用 `&` 符号标示，那么 `联合类型` 就是 `|`;
如果一个值是联合类型，**我们只能访问此联合类型的所有类型里`共有的成员`**。

```ts
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // -> success，Bird U Fish 求交集，共有成员为 layEggs
pet.swim();    // -> error
```

#### 类型保护与区分类型(Type Guards and Differentiating Types)

#### 用户自定义的类型保护

类型保护就是一些表达式，它们会在运行时检查以确保在某个作用域里的类型。 要定义一个类型保护，我们只要简单地定义一个函数，它的返回值是一个 `类型谓词`。

```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
}

// (<Fish>pet)  在这里表示强制类型转换
```

上述，`pet is Fish` 就是类型谓词。 谓词为 `parameterName is Type` 这种形式，`parameterName` 必须是来自于当前函数签名里的一个参数名。


#### typeof 类型保护

```ts
function isNumber(x: any): x is number {
    return typeof x === "number";
}

function isString(x: any): x is string {
    return typeof x === "string";
}

function padLeft(value: string, padding: string | number) {
    if (isNumber(padding)) {
        return Array(padding + 1).join(" ") + value;
    }
    if (isString(padding)) {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

我们不必将 `typeof x === "number"` 抽象成一个函数，因为TypeScript可以将它识别为一个类型保护。 也就是说我们可以直接在代码里检查类型了。

重写上面的代码：

```ts
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") { // 直接使用 typeof 判断
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

这些 `typeof类型保护` 只有两种形式能被识别： `typeof v === "typename"` 和 `typeof v !== "typename"`， `"typename"` 必须是 `"number"`， `"string"`， `"boolean"` 或 `"symbol"`。

#### instanceof 类型保护

与 `javascript` 中的类似，`instanceof` 类型保护是通过构造函数来细化类型的一种方式。
这里直接使用官方例子：

```ts
interface Padder {
    getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) { }
    getPaddingString() {
        return Array(this.numSpaces + 1).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) { }
    getPaddingString() {
        return this.value;
    }
}

function getRandomPadder() {
    return Math.random() < 0.5 ?
        new SpaceRepeatingPadder(4) :
        new StringPadder("  ");
}

// 类型为SpaceRepeatingPadder | StringPadder
let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder; // 类型细化为'SpaceRepeatingPadder'
}
if (padder instanceof StringPadder) {
    padder; // 类型细化为'StringPadder'
}
```

### 可以为 null 的类型

`TypeScript` 具有两种特殊的类型， `null` 和 `undefined`，它们分别具有值**null**和**undefined**。
默认情况下，类型检查器认为 `null` 与 `undefined` 可以赋值给任何类型。 `null` 与 `undefined` 是所有其它类型的一个有效值。 这也意味着，你阻止不了将它们赋值给其它类型，就算是你想要阻止这种情况也不行。


#### 可选参数与可选属性

使用了 `--strictNullChecks`，`可选参数`会被自动地加上 `| undefined`。

```ts
function f(x: number, y?: number) { // -> (x: number, y?: number | undefined)
    return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined); // 注意，是可选参数，不是参数
f(1, null); // error, null !== undefined
```

同理，`可选属性` 中也是同样的结果：

```ts
class C {
    a: number;
    b?: number;
}
let c = new C();
c.a = 12;
c.b = 13;
c.b = undefined; // ok
c.b = null; // error, null !== undefined
```

#### 类型保护和类型断言

由于可以为null的类型是通过联合类型实现，那么你需要使用类型保护来去除 `null`。

```ts
function f(sn: string | null): string {
    return sn || "default";
}
```

如果编译器不能够去除 `null` 或 `undefined`，你可以使用 `类型断言` 手动去除。 语法是添加 `!` 后缀:
例如：
`identifier!` 表示从 `identifier` 的类型里去除了 `null` 和 `undefined`。

### 类型别名

类型别名会给一个类型起个新名字。 类型别名有时和接口很像，但是可以作用于原始值，联合类型，元组以及其它任何你需要手写的类型。
**起别名不会新建一个类型 - 它创建了一个新名字来引用那个类型**。

```ts
type Name = string; // string 重命名为 Name
type NameResolver = () => string; // 同上..
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === 'string') {
        return n;
    }
    else {
        return n();
    }
}
```

#### 类型别名(type) vs 接口(interface)

1. 接口创建了一个新的名字，可以在其它任何地方使，类型别名并不创建新名字。
2. 类型别名不能被 `extends` 和 `implements`(自己也不能 `extends` 和 `implements` 其它类型)。
3. 如果你无法通过接口来描述一个类型并且需要使用联合类型或元组类型，这时通常会使用类型别名。

