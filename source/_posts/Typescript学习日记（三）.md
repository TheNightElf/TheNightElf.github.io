---
title: Typescript学习日记(三)
date: 2017-01-22 15:51:19
tags:
---
`typescript` 中的类与 `ES6` 中的 `class` 功能及使用基本一致；不过 `typescript` 对于属性的定义却是使用了一些后端语言的关键字。
<!-- more -->
### 类

`typescript` 中的类与 `ES6` 中的 `class` 功能及使用基本一致；不过 `typescript` 对于属性的定义却是使用了一些后端语言的关键字。


#### 类的集成

```ts
class Super {
    // let name: string;
    name: string; 

    constructor(n: string) {
        this.name = n;
    }

    fly () {
        console.log(`${this.name} can fly!`)
    }
}

class Son extends Super {

    // 在 ts 中，子类继承父类，如果之类中声明了 构造器(constructor) 则必须调用 super()
    // 否则 ts 会抛出错误

    constructor() {
        super();
    }

    //...
}

```


#### 公共，私有与受保护的修饰符

**默认值 public**

如果一个类中的成员没有显示的表明修饰符，那么默认他们都是 `public`。
上面的例子和下面的效果是一致的：

```ts
class Super {
    // let name: string;
    public name: string; 

    public constructor(n: string) {
        this.name = n;
    }

    public fly () {
        console.log(`${this.name} can fly!`)
    }
}
```

**私有属性 private**

如果一个类中的成员被标记为 `private`，那么它就不能在该类的外部被访问，因为它是私有的。

```ts
class Super {
    // let name: string;
    private name: string; 

    constructor(n: string) {
        this.name = n;
    }

    public fly () {
        console.log(`${this.name} can fly!`)
    }
}

// error
console.log(new Super('Bird').name)
```

**受保护属性 protected**

`protected` 与 `private` 的行为相似，但是 `protected` 的属性在 `子类内部` 中仍然可以访问。

```ts
class Super {
    // let name: string;
    protected name: string; 

    constructor(n: string) {
        this.name = n;
    }

    public fly () {
        console.log(`${this.name} can fly!`)
    }
}

class Son extends Super {

    constructor() {
        super(name)
    }

    handle() {
        // success
        console.log(this.name)
    }
}

// 能成功输出
console.log(new Son().handle)

// error
console.log(new Son().name) 
```

构造函数被标记为 `protected` 时，该类不能被实例化，但可以被继承，继承它的之类可以被实例化。

```ts
class Super {
    // let name: string;
    protected name: string; 

    protected constructor(n: string) {
        this.name = n;
    }

    public fly () {
        console.log(`${this.name} can fly!`)
    }
}

class Son extends Super {

    constructor() {
        super()
    }

    handle() {
        // ...
    }
}

let S = new Super(); // error, super是被保护的。

let s = new Son(); // success

```

**只读属性 readonly**

只读属性不能被修改，必须在 `声明时或者构造函数` 里被初始化。

```ts
class Super {
    // let name: string;
    name: string; 

    constructor(n: string) {
        this.name = n;
    }

    public fly () {
        console.log(`${this.name} can fly!`)
    }
}

let S = new Super('Jhon');
s.name = 'Carl'; // error
```

**参数属性**

在上面的例子中，我们总会声明一个 `name` 之后再通过构造器赋值。这种做法未免有些麻烦，通过 `参数属性` 我们可以将声明和赋值结合。

```ts
class Super {
    constructor(private name: string) { } // 直接在构造函数的参数中进行 声明、赋值
    fly(: number) {
        //...
    }
}
```


**储存器**

通过储存器，我们可以修改上文中提到的 `private` 属性。

```ts

class Super {
    private name: string;

    constructor(n: string) { this.name = n }

    get name(): string {
        return this.name;
    }

    set name(n: string) {
        this.name = n;
    }
}

let s = new Super('Jhon');
console.log(s.name); // -> 'Jhon'

s.name = 'Carl';

console.log(s.name); // -> 'Carl'

```

**静态属性 static**

上面的例子中属性大多都是实例化之后才能访问到的属性，通过 `static` 我们可以给类设置静态属性。

```ts
class Super {
    static name = 'Jhon';

    constructor() {  }
}

console.log(Super.name); // -> 'Jhon'
```

**抽象类 abstract**

抽象类作为其它子类的父类使用，不能被实例化；不同于接口，抽象类可以包含成员的实现细节。
抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。 抽象方法的语法与接口方法相似，两者都是定义方法签名但不包含方法体。

```ts
abstract class Super {
    abstract name: string;

    abstract fly(): void {
        // ...
    }
}

class Son extends Super {
    constructor() {
        super()
    }

    eat():void {
        // ...
    }
}

let S = new Super(); // -> error，抽象类不能被实例化
let s = new Son(); // -> success

s.fly(); // -> success 
s.eat(); // -> error, 抽象类中没有 eat 方法
```

#### 小结

`typescript` 中的类其实就是 `javascript` 中的构造函数；他们的用法完全相似，只不过 `typescript` 给构造函数添加了一些后端语言才有的 `修饰符`。


