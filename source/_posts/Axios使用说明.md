---
title: Axios使用说明
date: 2016-10-20 09:41:40
tags:
---
Axios是近年来备受推崇的一个网络请求库，它以基于Promise的方式封装了浏览器的XMLHttpRequest和服务器端node http请求，使得我们可以用es6推荐的异步方式处理网络请求。
<!-- more -->

### 为什么用Axios
> 官方是这样介绍的：
* 从浏览器创建XMLHttpRequest
* 从node.js创建http请求
* 支持Promise API
* 转换请求与响应数据
* 取消请求
* 自动转换JSON数据
* 支持客户端XSRF攻击防护

扩展了诸多功能之后 `axios.min.js` 文件最终打包出来大小仅有 `13kb`。

### 配置说明
官方给出了一份详尽的配置说明，供使用者自定义自己的使用方式。详情可以点击[这里](https://github.com/axios/axios)。
#### 全局默认配置

`axios` 给出了3个全局的配置信息，如下：
```js
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
```

**baseURL**
代表请求的地址，类似于 `host`。比如：
```js
var baseUrl = 'https://api.github.com'
axios.defaults.baseURL = baseUrl;
axios.get('/user')
    .then(res => {
        console.log(res)
    })
// ------------ 与上面的写法作用相同 --------------//
axios.get('https://api.github.com/user')
    .then(res => {
        console.log(res)
    })
```
通过配置 `baseUrl` 可以简化请求地址。

**Authorization**
授权信息，一般用于用户验证。如果没有设置，则请求是不会携带 `Authorization` 信息。

**Content-Type**
请求默认的 `Content-Type` 设置为 `application/x-www-form-urlencoded`，可以在全局更改，也可以在 `拦截器` 和 `请求/响应预处理` 中设置(后文中会细说)。

#### 请求默认配置

```js
{
  url: '/user',

  method: 'get', // default

  baseURL: 'https://some-domain.com/api/',

  transformRequest: [function (data, headers) {

    return data;
  }],

  transformResponse: [function (data) {

    return data;
  }],

  headers: {'X-Requested-With': 'XMLHttpRequest'},

  params: {
    ID: 12345
  },

  paramsSerializer: function(params) {
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },

  data: {
    firstName: 'Fred'
  },

  timeout: 1000,

  withCredentials: false, // default

  adapter: function (config) {
    /* ... */
  },

  auth: {
    username: 'janedoe',
    password: 's00pers3cret'
  },

  responseType: 'json', // default

  xsrfCookieName: 'XSRF-TOKEN', // default

  xsrfHeaderName: 'X-XSRF-TOKEN', // default

  onUploadProgress: function (progressEvent) {

  },

  onDownloadProgress: function (progressEvent) {

  },

  maxContentLength: 2000,

  validateStatus: function (status) {
    return status >= 200 && status < 300; // default
  },

  maxRedirects: 5, // default

  httpAgent: new http.Agent({ keepAlive: true }),
  httpsAgent: new https.Agent({ keepAlive: true }),

  proxy: {
    host: '127.0.0.1',
    port: 9000,
    auth: {
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },

  cancelToken: new CancelToken(function (cancel) {

  })
}

```
最常用的也就是前面的 `10` 个配置，而且意思都特别容易理解。
具体的用法可以参考[官方说明](https://github.com/axios/axios)和[examples](https://github.com/axios/axios/tree/master/examples)。

#### 返回体
```js
{
  data: {}, // 服务端对于请求返回的具体信息

  status: 200,// 服务端 http 的状态码

  statusText: 'OK', // 服务端 http 文本的响应状态

  headers: {}, // 响应头部

  config: {}, // axios 请求时的配置信息

  request: {} // 请求的详细信息(XMLHttpRequest)
}
```

由于内容实在太多，本文暂时就讲这么多。其它关于 `axios` 的方法的用法和说明都会在后文中讲解。