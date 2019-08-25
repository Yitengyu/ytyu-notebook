## 为什么发请求的时候总是伴随着OPTIONS请求



## 前言

“为什么发请求的时候总会有OPTIONS请求？为什么有时候有，有时候没有？它做了什么事情？”

这个问题的本质涉及到同源策略与跨域。以下做简单介绍。

## 相关名词与文章

1. 同源策略([same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy))

2. 跨域资源共享（[CORS Cross-Origin Resource Sharing](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)）

3. [CORS的W3C标准](https://www.w3.org/TR/cors/)

参考文章：

[浏览器同源政策及其规避方法](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)

[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)

## 读后感

重要的几个点：

1. 同源策略是**浏览器**为了安全而广泛采用的。
2. 有时候我们需要突破同源策略的限制。
3. 跨域和CORS不是一回事。
4. CORS是允许客户端跨域请求资源的一个机制，需要浏览器和服务器支持，前端几乎无感（与同源请求相比）。
5. 发出非简单请求之前，浏览器会通过**预检请求**询问当前域名以及希望使用的HTTP动词和头信息字段是否会被服务器接受。

##  结合项目解答问题

1. 我们的前端域名与后端域名是不一样的，所以显然我们的所有API都是跨域的。
2. 因为我们的请求会有`token`（admin）， 'platform'（移动端）等字段，因此请求均为非简单请求

```js
// AppPivotEHRAdminWeb/src/shared/utils/http.ts
headers: {
    common: {
        token: ssoAuth.token
        // platform: 'ehrAdmin'
    },
    post: {
        'Content-Type': 'application/x-www-form-urlencoded'
    }
}
```

*更多的问题：content-type为什么是`application/x-www-form-urlencoded`?*

3. 因此浏览器会自动先发一个预检（preflight）请求，也就是OPTIONS请求。
4. 可以看到我们发的OPTIONS请求，后端返回的响应报文头中(admin项目)，字段`Access-Control-Max-Age`的值是3600（单位是秒），所以相同（*怎样定义‘相同’？*）的跨域请求在**6分钟**内再次发起时，不需要先发起预检请求。

## 优化的探讨

1. `Access-Control-Max-Age`的值能不能更大？以减少请求数来提高性能。比如移动端返回是18000
2. 移动端的请求如果去掉platform字段，那对应的的GET、POST、PUT就是简单请求了，理论上可以提高网络请求的性能。现在有网关的情况下是不是不再需要了？