---
title: Typescript学习日记(九)
date: 2017-03-02 11:01:21
tags:
---


本文主要介绍 `typescript` 几种高级类型及相关内容。
<!-- more -->


### 字符串字面量类型

字符串字面量类型允许你指定字符串必须的固定值。 在实际应用中，字符串字面量类型可以与联合类型，类型保护和类型别名很好的配合。

这里直接使用官方的例子：
```ts
type Easing = "ease-in" | "ease-out" | "ease-in-out"; // 使用 type 重命名 联合类型
class UIElement {
    animate(dx: number, dy: number, easing: Easing) {
        if (easing === "ease-in") {
            // ...
        }
        else if (easing === "ease-out") {
        }
        else if (easing === "ease-in-out") {
        }
        else {
            // error，没传 easing
        }
    }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // error，没有 uneasy 类型
```

字符串字面量类型还可以用于区分函数重载：
```ts
function createElement(tagName: "img"): HTMLImageElement; // img 标签
function createElement(tagName: "input"): HTMLInputElement; // input 标签
// ... 
function createElement(tagName: string): Element {
    // ... 
}
```

### 数字字面量类型

TypeScript还具有数字字面量类型。不过数字字面量类型很少使用，因为它不能像字符串字面量表达一个具有意义的值。

```ts
function rollDie(): 1 | 2 | 3 | 4 | 5 | 6 {
    // ...
}
```

### 枚举成员类型

在我们谈及“单例类型”的时候，多数是指枚举成员类型和数字/字符串字面量类型，尽管大多数用户会互换使用“单例类型”和“字面量类型”。


### 可辨识联合（Discriminated Unions）

你可以合并 `单例类型`，`联合类型`，`类型保护` 和 `类型别名` 来创建一个叫做 `可辨识联合的高级模式`，它也称做 `标签联合` 或 `代数数据类型`。 可辨识联合在函数式编程很有用处。 一些语言会自动地为你辨识联合；而TypeScript则基于已有的JavaScript模式。 它具有3个要素：

1. 具有普通的单例类型属性— `可辨识的特征`。
2. 一个类型别名包含了那些类型的联合— 联合。
3. 此属性上的类型保护。

```ts
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}
```

每个接口都有 `kind` 属性但有不同的字符串字面量类型。 `kind` 属性称做 `可辨识的特征` 或 `标签`。
将上面三个接口联合：

```ts
type Shap = Square | Rectangle | Circle;
```

使用可辨识联合：

```ts
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

发现了吗？这种写法估计大家都不会陌生，只是为这种形式起了一个合理化的名字罢了。

#### 完整性检查

当没有涵盖所有可辨识联合的变化时，我们想让编译器可以通知我们。 比如，如果我们添加了 `Triangle` 到 `Shape`，我们同时还需要更新 `area`:

```ts
type Shap = Square | Rectangle | Circle | Triangle;

function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
    // 这会出错，因为没有一个 Triangle 类型
}
```

解决方法一：
使用 `--strictNullChecks` 并且指定一个返回值类型;

```ts
function area(s: Shape): number { // error: returns number | undefined
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```
因为 `switch` 没有包涵所有情况，所以 `TypeScript` 认为这个函数有时候会返回 `undefined`。 如果你明确地指定了返回值类型为 `number`，那么你会看到一个错误，因为实际上返回值的类型为 `number | undefined`。

解决方法二：
使用 `never` 类型，编译器用它来进行完整性检查。

```ts
function assertNever(x: never): never {
    throw new Error("Unexpected object: " + x);
}
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
        default: return assertNever(s); // error，忘记的那个 case 类型将被 assertNever 标记为 never 类型，并抛出错误
    }
}
```

`assertNever` 检查 `s` 是否为 `never` 类型—-即为除去所有可能情况后剩下的类型。 如果你忘记了某个 `case`，那么 `s` 将具有一个真实的类型并且你会得到一个错误。

### 多态的 this 类型

不多解释。

### 索引类型(Index types)

使用索引类型，编译器就能够检查使用了动态属性名的代码。 例如，一个常见的 `javaScript` 模式是从对象中选取属性的子集。

下面是如何在 `TypeScript` 里使用此函数，通过 `索引类型查询` 和 `索引访问` 操作符：

```ts
// keyof 索引类型查询操作符，类似与 javascript 中的 for in 遍历中的取 key 操作
// T[K] 索引访问操作符，T[K][] 这里表示 取值后的类型，就是返回的类型
// pluck(person, ['name']) -> T[k][] -> 'Jarid'[] -> string[]
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}

interface Person {
    name: string;
    age: number;
}
let person: Person = {
    name: 'Jarid',
    age: 35
};
let strings: string[] = pluck(person, ['name']);// -> return ['Jarid']
```

编译器会检查 `name` 是否真的是 `Person` 的一个属性。 本例还引入了几个新的类型操作符。 首先是 `keyof T`， 索引类型查询操作符。 对于任何类型 `T`， `keyof T`的结果为 `T` 上已知的公共属性名的联合。例如：

```ts
let props: keyof Person;  
// -> name | age
// keyof Person是完全可以与 'name' | 'age'互相替换的
```

第二个操作符是 `T[K]`， 索引访问操作符。在这里，类型语法反映了表达式语法。 这意味着 `person['name']` 具有类型 `Person['name']` —- 在我们的例子里则为 `string` 类型。 然而，就像索引类型查询一样，你可以在普通的上下文里使用 `T[K]`，这正是它的强大所在。 你只要确保类型变量 `K extends keyof T` 就可以了。


#### 索引类型和字符串索引签名

```ts
interface Map<T> {
    [key: string]: T;
}
// Map 是一个只包含一对键值的对象 key -> string，value -> T

let keys: keyof Map<number>; // string
let value: Map<number>['foo']; // number
```

### 映射类型

通过 `映射类型` 将一个已知的类型每个属性都变为可选的/只读的。在映射类型里，新类型以相同的形式去转换旧类型里每个属性。

```ts
// 转换成只读属性
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}

// 转换成可选属性
type Partial<T> = {
    [P in keyof T]?: T[P];
}
```

使用方式：

```ts
type PersonPartial = Partial<Person>;
type ReadonlyPerson = Readonly<Person>;

// 假设Person interface Person = {age: number; gender: string; name: string}

type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}

// T -> Person
// P in keyof T -> keyof T 指的是 Person 中所有的 key
// P in keyof T -> P 为 age & gender & name
// T[P] 取值操作

```

再看一个`包装属性`的例子：

```ts
type Proxy<T> = {
    get(): T;
    set(value: T): void;
}
type Proxify<T> = {
    [P in keyof T]: Proxy<T[P]>;
}
function proxify<T>(o: T): Proxify<T> {
   // ...
}
let proxyProps = proxify(props);
```
是不是有种函数互相嵌套的感觉？

**`Pick`**
取出指定的属性。

定义：
```ts
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
}
```

我们用上面的Person来使用：
```ts

type PickPerson = Pick<Person, 'age' | 'name'>; // -> {age: number; name: string}


// T: Person, K: 'age' | 'name',  T: 'age' | 'gender' | 'age', P: 'age' | 'name',
// [P in K] 遍历 K
// T[P] 取值
```

**`Record`**
将传入的属性统一定义类型。

```ts
type Record<K extends string, T> = {
    [P in K]: T;
}

type RecordPerson = Record<'gender' | 'name', string>; // -> {gender: string; name: string}

// K: 'gender' | 'name', T: string
```


#### 由映射类型进行推断

上面是包装属性，下面看下如何拆包。

```ts
function unproxify<T>(t: Proxify<T>): T {
    let result = {} as T;
    for (const k in t) {
        result[k] = t[k].get();
    }
    return result;
}

```

