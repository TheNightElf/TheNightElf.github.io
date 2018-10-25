---
title: React.16.0.0
date: 2017-10-20 14:58:27
tags:
---
`React` 的版本从 `15.6.2` 升级到 `16.0.0` 不仅更改了 `License`，其内部也给我们提供了一些非常有实用性的特性，像 `createPortal、React.Fragment` 等。
<!-- more -->


### 为什么要有 `createPortal` ？

`React` 的数据流动是单向传递的，所以我们写的很多的 `React` 的组件中，DOM结构也是 一层套着一层。绝大多数的web网站都会有一个会被频繁使用的全局弹层，如果按照 `React` 中的 `DOM` 结构来写的话，那么只能被放在当前的组件内部使用。`createPortal` 的出现帮我们解决了这个问题，你可以在任何地方使用它并且可以自己指定你的弹层出现的位置。

### `createPortal` 介绍

`createPortal` 的使用非常简单，该方法只接受两个参数：
* 一个是你要添加的DOM节点
* 一个是目标节点。

```js
createPortal(
    dom,  // 你要添加的DOM结构
    targetDom  // 放置该DOM结构的位置
)
```

### 自己实现一个全局提示

模仿 `antd` 中 `message` 组件的使用方法。

```js
import React, {Component} from 'react';
import {render, createPortal} from 'react-dom';
import {Icon} from 'antd';

import './style.scss';

class Message extends Component {
    constructor() {

        super(...arguments);

        this.node = document.createElement('div');
    
    }

    componentDidMount() {

        const {during} = this.props;

        document.body.appendChild(this.node);

        // 弹层显示时间达到后移除
        setTimeout(() => {

            document.body.removeChild(this.node);
        
        }, during);
    
    }

    render() {

        const {type} = this.props;

        return createPortal(
            <div className={`global-message global-message-${type}`}>
                {
                    type === 'success' ? <Icon type='check-circle'/> :
                        type === 'error' ? <Icon type='exclamation-circle-o'/> : null
                }
                {this.props.children}
            </div>,
            this.node
        );
    
    }

    componentWillUnmount() {

        document.body.removeChild(this.node);
    
    }
}

export default {
    success: function (txt, during = 3000) {

        let far = document.createElement('div');

        render(
            <Message during={during} type='success'>{txt}</Message>,
            far
        );
    
    },
    error: function (txt, during = 3000) {

        let far = document.createElement('div');

        render(
            <Message during={during} type='error'>{txt}</Message>,
            far
        );
    
    }
};
```

这样我们就可以在任何组件内部调用 `Message` 组件了，它会为我们在全局的DOM结构上展示一个我们想要的 `DOM` 结构。

### 顺便一提 – `React.Fragment`

通俗的，你可以把它理解为 `document.fragment` 的 `React` 版。它的出现意味着我们可以直接渲染两个并列的同级组件，而不是每次都要在他们外层套一个无用的 `div` 标签。

**错误写法：**

```js
// 这种写法在React中是会报错的
render () {
    return (
        <li>1</li>
        <li>2</li>
    )
}
```

**正确写法：**

```js
render () {
    return (
        <React.Fragment>
            <li>1</li>
            <li>2</li>
        </React.Fragment>
    )
}
```

### 剩下的特性

`render` 方法不再只能渲染 `DOM` 结构了，现在你可以用它来渲染 `Array` & `String`。

```js
render () {
    // return [1, 2, 3]
    return 'hello world'
}
```

`componentDidCatch` 就不说了，它的用法很简单，想了解的可以去官方文档上找找。就写这么多吧，干活去了。