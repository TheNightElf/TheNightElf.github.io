---
title: Axios设置默认请求参数
date: 2016-10-28 10:15:40
tags:
---

在网站建设成行对接 `api` 接口的时候经常会遇到一个场景——用户的登录验证。
大多数的做法都是服务端返回一个 `token` 给客户端，客户端存储在 `cookie` 或者 `Storage` 中。
等下次发请求的时候再将 `token` 携带，传递给服务端。但是这么做会非常的繁琐，每次都要在请求中设置 `token`。
<!-- more -->

### 解决方案

#### 方案一
使用 `axios` 官方提供的 `Interceptors` 拦截器进行配置。
假设 `token` 是储存在 `localeStorage` 中；
```js
// 请求拦截器
axios.interceptors.request.use(function (config) {
    const token = localStorage.getItem('token')
     // 判断请求的类型
     // 如果是post请求就把默认参数拼到data里面
     // 如果是get请求就拼到params里面
    if(config.method === 'post') {
        let data = qs.parse(config.data)

        config.data = qs.stringify({
            token: token,
            ...data
        })
    } else if(config.method === 'get') {
        config.params = {
            token: token,
            ...config.params
        }
    }
    return config;
  }, function (error) {
    return Promise.reject(error);
  })
```

拦截器会拦截所有的 `请求` 或者 `响应`，我们可以在拦截到请求的时候将用户的请求参数中添加 `token`。
这样就不需要每次都在请求中添加一次 `token` 的操作了。

**扩展**
响应拦截器可以拦截每一次服务端的响应信息，用户的 `登录超时` 完全可以在这个拦截器中处理。

#### 方案二
通过更改 `request config` 的默认配置实现。
在上文的介绍中，我们可以看到 `request config` 中有一项 `transformRequest` 的配置。该方法会在每个请求发送给服务端之前执行，我们来看下如何使用。
```js
axios.defaults.transformRequest = [function (data, header) {
    const token = localStorage.getItem('token')
    //header.post = {
      //  "Content-Type": "application/json;charset=UTF-8"
    //}
    var obj = Object.assign({}, data, { token: token })
    return JSON.stringify(obj)
}]
```

使用这个方法的时候有几个需要注意的地方:
1. 该方法的返回值必须是 `string` 或 `Buffer实例` 或 `ArrayBuffer` 或 `FormData` 或 `Stream`。
2. 转换完之后请求体的 `payload` 会变成 `FormData` 的形式，可以在 `header` 中重写 `Content-Type`。

### 总结
`Axios` 提供了非常灵活的配置方法供用户使用，我们不需要再自己封装一个类似的 `xhr` 方法。所有的用法官方都有详细的说明，如果你想了解更多，可以点击[这里](https://github.com/axios/axios)。