---
title: Typescript学习日记(十)
date: 2017-03-17 14:20:02
tags:
---

`symbol` 是 `ECMAScript 2015` 提出的一种行的原生类型，就像 `number` 和 `string` 一样。实现了 `Symbol.iterator` 方法就可以使用 `for-of` 遍历。
<!-- more -->

### Symbols

#### 创建

```js
// 注意，Symbol() 并不是一个构造函数，所以不需要 new 关键字
let sy = Symbol('key');
```


#### Symbols是不可改变且唯一的
```js
let sy1 = Symbol('a');
let sy2 = Symbol('a');

sy1 === sy2;  // -> false
```

#### 像字符串一样，symbols也可以被用做对象属性的键

```js
let sy = Symbol('key');

let obj = {
    [sy]: "value"
};

console.log(obj[sym]); // "value"
```

#### Symbols也可以与计算出的 `属性名声明相结合` 来 `声明对象的属性和类成员`

```js
const getClassNameSymbol = Symbol();

class C {
    [getClassNameSymbol](){
       return "C";
    }
}

let c = new C();
let className = c[getClassNameSymbol](); // "C"
```

#### 内置属性

**`Symbol.hasInstance`**
方法，会被 `instanceof` 运算符调用。构造器对象用来识别一个对象是否是其实例。

**`Symbol.isConcatSpreadable`**
布尔值，表示当在一个对象上调用 `Array.prototype.concat` 时，这个对象的数组元素是否可展开。

**`Symbol.iterator`**
方法，被for-of语句调用。返回对象的默认迭代器。

**`Symbol.match`**
方法，被 `String.prototype.match` 调用。正则表达式用来匹配字符串。

**`Symbol.replace`**
方法，被 `String.prototype.replace` 调用。正则表达式用来替换字符串中匹配的子串。

**`Symbol.search`**
方法，被 `String.prototype.search` 调用。正则表达式返回被匹配部分在字符串中的索引。

**`Symbol.species`**
函数值，为一个构造函数。用来创建派生对象。

**`Symbol.split`**
方法，被 `String.prototype.split` 调用。正则表达式来用分割字符串。

**`Symbol.toPrimitive`**
方法，被 `ToPrimitive` 抽象操作调用。把对象转换为相应的原始值。

**`Symbol.toStringTag`**
方法，被内置方法 `Object.prototype.toString` 调用。返回创建对象时默认的字符串描述。

**`Symbol.unscopables`**
对象，它自己拥有的属性会被 `with` 作用域排除在外。

更多关于 `Symbols` 的介绍可以查看[这里](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)。

### 迭代器(iterator)和生成器

当一个对象实现了 `Symbol.iterator` 属性时，我们认为它是可迭代的。目前一些内置的类型如 `Array`，`Map`，`Set`，`String`，`Int32Array`，`Uint32Array` 等都已经实现了各自的 `Symbol.iterator`。 对象上的 `Symbol.iterator` 函数负责返回供迭代的值。


#### `for-of` 语句

`for-of` 会遍历可迭代的对象，调用对象上的 `Symbol.iterator` 方法。

```js
let arr = [1, "string", false];

for (let val of arr) {
    console.log(val); // 1, "string", false
}
```

#### `for-of` vs `for-in`

`for-of` 和 `for-in` 均可迭代一个列表；但是用于迭代的值却不同，`for-in` 迭代的是对象的 `键` 的列表，而`for-of` 则迭代对象的键对应的值。

```js
let arr = [1, "string", false];

for(let key in arr) {
    console.log(key); // 0, 1, 2
}

for (let val of arr) {
    console.log(val); // 1, "string", false
}
```

另一个区别是 `for-in` 可以操作任何对象；它提供了查看对象属性的一种方法。 但是 `for-of` 关注于迭代对象的值。内置对象 `Map` 和 `Set` 已经实现了 `Symbol.iterator` 方法，让我们可以访问它们保存的值。

```js
let pets = new Set(["Cat", "Dog", "Hamster"]);
pets["species"] = "mammals";

for (let pet in pets) {
    console.log(pet); // "species"
}

for (let pet of pets) {
    console.log(pet); // "Cat", "Dog", "Hamster"
}
```


### 三斜线指令

三斜线指令是包含单个XML标签的单行注释。 注释的内容会做为编译器指令使用。

三斜线指令仅可放在包含它的文件的最顶端。 一个三斜线指令的前面只能出现单行或多行注释，这包括其它的三斜线指令。 如果它们出现在 `一个语句或声明` 之后，那么它们会被当做普通的单行注释，并且不具有特殊的涵义。

#### `/// <reference path="..." />`

`/// <reference path="..." />` 指令是三斜线指令中最常见的一种。 它用于声明文件间的 `依赖`。三斜线引用告诉编译器在编译过程中要引入的额外的文件。

#### 使用 `--noResolve`
如果指定了 `--noResolve` 编译选项，三斜线引用会被忽略；它们不会增加新文件，也不会改变给定文件的顺序。

#### `/// <reference types="..." />`
与 `/// <reference path="..." />` 指令相似，这个指令是用来声明依赖的； 一个 `/// <reference path="..." />` 指令声明了对 `@types` 包的一个依赖。

在声明文件里包含 `/// <reference types="node" />`，表明这个文件使用了 `@types/node/index.d.ts` 里面声明的名字； 并且，这个包要在编译阶段与声明文件一起被包含进来。

解析 `@types` 包的名字的过程与解析 `import` 语句里模块名的过程类似。 所以可以简单的把三斜线类型引用指令想像成针对包的 `import` 声明。

仅当在你需要写一个 `d.ts` 文件时才使用这个指令。

对于那些在编译阶段生成的声明文件，编译器会自动地添加 `/// <reference types="..." />`； 当且仅当结果文件中使用了引用的@types包里的声明时才会在生成的声明文件里添加 `/// <reference types="..." />` 语句。


### 总结

大致看了下，后面的内容都是一些需要记忆的内容，不会牵扯到太多的代码；这里就不再继续往下写了，有想阅读后面内容的朋友可以进入[Typescript中文文档](https://www.tslang.cn/index.html)继续学习。




