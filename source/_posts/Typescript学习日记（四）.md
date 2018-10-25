---
title: Typescript学习日记(四)
date: 2017-01-27 11:58:26
tags:
---

本文将会介绍 `typescript` 中函数的相关内容。
<!-- more -->

### 函数

#### 函数定义

此处略过。

#### 函数类型

在函数定义的时候我们可以为函数指定 `参数类型` 与 `返回类型`。

 ```ts
 // 指定 add 函数接受两个 number 类型的参数x,y，返回一个 number 类型的值
 function add(x: number, y: number) :number {
     return x + y;
 }
 ```

上面这种写法是 `typescript` 中最常见的函数类型写法，不过这是一种 `简写写法`。

**完整写法：**

```ts
let add: 
    //       这是 参数类型 部分         =              这是 返回值类型 部分
    (x: number, y: number) => number = function (x: number, y: number): number { return x + y }
    // (baseValue: number, increase: number) => number = function (x: number, y: number): number { return x + y }
```
有没有发现原本简单明了的函数表达式变的更加难以理解了?没关系，定义函数的时候还是可以使用 `简写写法`，`typescript` 会自动帮我们转化。

**类型推导**
非常简单，不再叙述，[详见这里](https://www.tslang.cn/docs/handbook/functions.html)。


#### 可选参数与默认参数

看过前面 `interface` 的可选形式，对这里函数的可选形式应该不会陌生，它们的表现形式完全一样。至于默认参数，写过 `es6` 函数语法的应该已经非常熟悉了。

**可选参数**
```ts
function yourName(firstName: string, lastName?: string): string {
    if(lastName) {
        return `${firstName} ${firstName}`;
    } else {
        return firstName;
    }
}

yourName(); // -> error，最少传入一个参数
yourName('Carl'); // -> success, 'Carl'
yourName('Mr.', 'Carl'); // success, 'Mr. Carl'
yourName('Mr.', 'Carl', 'Jhon'); // error, 参数太多
```

**默认参数**
```ts
// 需要注意的是 参数位置 不能更改
function yourName(firstName: string, lastName = 'Jhon'): string {
        return `${firstName} ${firstName}`;
}

yourName(); // -> error，最少传入一个参数
yourName('Mr.'); // -> success， 'Mr. Jhon'
yourName('Mr.', undefined); // -> success， 'Mr. Jhon'
yourName('Mr.', 'Carl'); // -> success， 'Mr. Carl'
```

#### 剩余参数

剩余参数使用的是 `es6` 的 `rest剩余参数` 语法(没有使用过的人可以点击[这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Rest_parameters)查看)。`typescript` 中的 `剩余参数` 的使用方式和 `es6` 中完全一致。

```ts
function add(x: number, ...restArgs: number[]) {
    let total = x;
    restArgs.forEach(item => {
        total += item
    })

    return total;
}

add(1,2,3,4,5); // -> 15
```

#### this

关于 `this` 这里不说太多，网上关于它的文章太多了。
关于使用 `箭头函数` 对 `this` 的影响可以查看我前面写的一片文章[箭头函数](https://thenightelf.github.io/2018/01/03/%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0/);


`typescript` 中对于函数 `this` 的处理放在了参数中进行约束。

```ts
interface Card {
    suit: string;
    card: number;
}
interface Deck {
    suits: string[];
    cards: number[];
    createCardPicker(this: Deck): () => Card;
}

let deck: Deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    // 在参数中约定 this 为 Deck 类型
    createCardPicker: function(this: Deck) {
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

#### 函数重载

不同于 `javascript` 的函数重载，`typescript` 中重复的对于一个函数进行定义，只要 `参数与返回值类型` 不同，就不会相互覆盖。
```ts
// javascript
function say() {
    console.log('hi');
}

function say() {
    console.log('hello');
}

say(); // -> hello

// typescript
// 官方的例子比较直接，这里直接使用官方的例子
let suits = ["hearts", "spades", "clubs", "diamonds"];

// 对于函数 pickCard 的参数与返回值进行约束
// 约束参数类型为 {suit: string; card: number; }[] 类型时，返回值类型为 number
function pickCard(x: {suit: string; card: number; }[]): number;

// 约束参数类型为 number 类型时，返回值类型为 {suit: string; card: number; }
function pickCard(x: number): {suit: string; card: number; };

// 主体处理函数
function pickCard(x): any {
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);


// -> error，没有一个重载的参数约束为 string
let pickedCard3 = pickCard('15');
alert("card: " + pickedCard3.card + " of " + pickedCard3.suit);

```

**这里需要注意的是**：如果通过重载对函数进行约束之后，那么该函数只能接受重载的定义类型。

