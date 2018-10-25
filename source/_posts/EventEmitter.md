---
title: EventEmitter
date: 2016-10-12 16:08:10
tags: 
---

自己编写一个简单的 `EventEmitter` 实现 `观察者模式`。

<!-- more -->

### 观察者模式
> 观察者模式面向的需求是：A对象（观察者）对B对象（被观察者）的某种变化高度敏感，需要在B变化的一瞬间做出反应。
举个例子，新闻里喜闻乐见的警察抓小偷，警察需要在小偷伸手作案的时候实施抓捕。
在这个例子里，警察是观察者、小偷是被观察者，警察需要时刻盯着小偷的一举一动，才能保证不会错过任何瞬间。
程序里的观察者和这种真正的【观察】略有不同，观察者不需要时刻盯着被观察者(例如A不需要每隔1ms就检查一次B的状态)；
二是采用注册(Register)或者成为订阅(Subscribe)的方式告诉被观察者：
我需要你的某某状态，你要在它变化时通知我。采取这样被动的观察方式，既省去了反复检索状态的资源消耗，也能够得到最高的反馈速度。

### 观察者模式的优缺点
#### 优点
* 观察者模式在被观察者和观察者之间建立一个抽象的耦合。被观察者角色所知道的只是一个具体观察者列表，每一个具体观察者都符合一个抽象观察者的接口。被观察者并不认识任何一个具体观察者，它只知道它们都有一个共同的接口。
* 由于被观察者和观察者没有紧密地耦合在一起，因此它们可以属于不同的抽象化层次。如果被观察者和观察者都被扔到一起，那么这个对象必然跨越抽象化和具体化层次。
* 观察者模式支持广播通讯。被观察者会向所有的登记过的观察者发出通知。

#### 缺点
* 如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。
* 如果在被观察者之间有循环依赖的话，被观察者会触发它们之间进行循环调用，导致系统崩溃。
* 如果对观察者的通知是通过另外的线程进行异步投递的话，系统必须保证投递是以自恰的方式进行的。
* 虽然观察者模式可以随时使观察者知道所观察的对象发生了变化，但是观察者模式没有相应的机制使观察者知道所观察的对象是怎么发生变化的。

### 实现方法

```js
function EventEmitter() {
    this.events = {};
}
    //绑定事件函数
EventEmitter.prototype.on = function(eventName, callback) {
    this.events[eventName] = this.events[eventName] || [];
    this.events[eventName].push(callback); // 1
};
    //触发事件函数
EventEmitter.prototype.emit = function(eventName, _) {
    var events = this.events[eventName],
    args = Array.prototype.slice.call(arguments, 1), // 2
    i, m;
 
    if (!events) {
        return;
    }

    for (i = 0, m = events.length; i < m; i++) {
        events[i].apply(null, args); // 3
    }
};
    // 删除事件函数
EventEmitter.prototype.off = function (eventName) {
    delete this.events[eventName]
}
```

以上有3个注意点：
1.`EventEmitter` 接收事件时是以事件名作为 `key` 执行函数作为 `value:array`。
2.但用户触发 `emit` 时可以传递多个参数，所以将除 `事件名(arg[0])` 之外的参数视为传参，缓存供第 `3` 步时使用。
3.遍历事件的执行函数 `value` 并将 `args` 传递给每一个事件。

更多关于 `call` 和 `apply` 的介绍请参考[这里](https://developer.mozilla.org/en-US/)

### 使用
```js

var Person = function {

}

Person.prototype = new EventEmitter()

var p = new Person()

p.on('hello', function () {
    console.log('hello world')
})

p.emit('hello') // -> hello world

```

传递参数时，可以直接在 `on` 执行函数中获取 `emit` 中传递的参数；
```js
p.on('hello', function (name) {
    console.log('hello' + name)
})

p.emit('hello', 'Mark') // -> hello Mark

```
### 总结
以上只是对观察者模式主体的一个简单实现，还有很多可以扩展的方法和优化的方案，这里我就不再继续写了。
如果你想了解更加完整的实现方式可以参考 [EventEmitter2](https://github.com/asyncly/EventEmitter2)。
