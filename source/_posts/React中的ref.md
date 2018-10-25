---
title: React中的ref
date: 2017-06-20 11:25:06
tags:
---

众所周知，`React` 是一个非常优秀的 `UI-Library`，它将我们**从事件驱动**带入了**数据驱动**的大潮中。在 `React` 中，一般通过修改 `state` 和 `props` 来更新组件。但是**数据驱动**并不意味着我们就完全不需要操作 `DOM` 了， `ref` 就是 `React` 为我们提供操作 `DOM` 的一个方法。
<!-- more -->

#### 什么时候该使用 `ref` ?

> There are a few good use cases for refs:
* Managing focus, text selection, or media playback.
* Triggering imperative animations.
* Integrating with third-party DOM libraries.

官方给出的对于 `ref` 应该被使用的三种情况。

1. 需要操作 `input` 框的 `focus` 状态、文本的选中、媒体资源回放等。
2. 触发必要的动画。
3. 集成了第三方操作 `DOM` 的库。

虽然给出了使用 `ref` 的场景，但是 `React` 还是非常不推荐使用 `ref` 的。


#### ref是什么？

`ref` 取自英文单词 `reference`，意为**引用**。在 `React` 中， `ref` 表示对 **组件/DOM** 的真实引用而非在 **虚拟DOM** 中的引用。所以，在通过 `ref` 访问到的 `DOM结构` 和 `组件` 都是已经存在于页面上的。需要注意的是，这里所说的 `组件` 指的是 **有状态的组件——可以被实例化** 而非无状态的组件。引用组件后，可以通过该组件的实例访问其内部的 `state` & `props` & `method`。

#### ref的使用

官方给出了两种 `ref` 的使用方式：
* 回调函数(推荐)：
```js

class Demo extends Component {

    getInput = (ref) => {
        // 回调函数中的参数 `ref` 就是当前的真实 `DOM`。 
        console.log(ref)

        this.textInput = ref;
    }

    focusInput = () => {

        this.textInput.focus()
    }

    render () {
        return (
            <div>
                <button onClick={this.focusInput}>click</button>

                {/*<input ref={this.getInput()}/>*/}
                <input ref={ref => this.textInput = ref}/>
            </div>
        )
    }
}
```

    **使用回调函数的触发时机**

    * 组件被渲染后，回调参数 `ref` 为input的dom对象
    * 组件被卸载后，回调参数 `ref` 为null，确保内存不被泄露
    * `ref` 被改变

* 使用字符串

```js

class Demo extends Component {

    focusInput = () => {

        this.refs.textInput.focus()
    }

    render () {
        return (
            <div>
                <button onClick={this.focusInput}>click</button>

                <input ref='textInput'/>
            </div>
        )
    }
}


```

使用字符串时，通过 `this.refs` 访问dom或者组件实例。

#### 结尾

最后，还是要引用官方说的一句话：
> ** Don’t Overuse Refs **

既然我们选择了使用 `React` 我们就要习惯于使用数据来驱动组件更新，如果还是频繁的使用DOM操作，那和使用 `jQuery` 有什么区别呢？





