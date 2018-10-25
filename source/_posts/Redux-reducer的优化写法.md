---
title: Redux-reducer的优化写法
date: 2017-05-12 17:25:43
tags:
---

本文介绍如何摆脱 `switch-case`，以一种更方便的形式接收 `action` 的通知。
<!-- more -->

### 前言
`redux` 是 `react` 数据管理的一个解决方案，其原理借鉴了 `flux` 架构。`redux` 中一共由三个核心部分构成--`store`、`reducer`、`action`，其中 `reducer` 则是负责接收 `action` 的消息，然后对数据部分进行处理。官方对于 `reducer` 的描述为一个简单的 `switch-case` 结构。下面会介绍 `reducer` 以一种 `key-value` 的形式接收 `action`。

### 旧版 `switch-case` 写法

下面是伪代码：

```js
import {xxx} from 'constants';

// action
export const actionHandler = () => {
    return {
        type: xxx,
        //...
    }
}
```

```js
import {xxx, yyy} from 'constants';

const initialState = {

}

// reducer
export const Model = (state = initialState, action) => {
    switch(action.type) {
        case xxx: 
            return //...
        case yyy:
            return //...

        default:
            return state;
    }
}
```

上面的这种写法并不能说是错，因为官方给出的例子就是这么介绍的。但是就好比 `javascript loop` 一样，大家都在用 `forEach`、`for-of`了，而我们还是在使用 `for` 循环。而且 `switch-case` 就如同循环一般，如果找不到匹配项就会一直向下查找，性能方面不是特别好(我真不想说这句话)。

### 重写 `reducer`

简单的说，`reducer` 是为了接收一个 `action.type` 并对数据进行相应的处理，然后返回。

```js
function reducer (state = initialState, action){
    // 处理 action，返回 {}
}

```

我们可以将 `action.type` 更 `reducer` 合并：

```js
function reducer (state = initialState, action){
    return () => (
        {
            [action.type]: (state, action) => {

            }
        }
    )
}
```

现在 `reducer` 还是接收 `action.type` 然后触发相应的数据处理函数。只不过从 `switch-case` 变成了 `key-value` 取值的形式。
---

最终，我们对上面的代码进行优化：

```js
// createReducer
export  default function (initialState, reducerMap) {

    return (state = initialState, action) => {

        const reducer = reducerMap[action.type];
        return reducer ? reducer(state, action) : state;
    
    };

}
```

上面的最终版还是有一点问题，这里我再优化了，留给有心的人去解决。
