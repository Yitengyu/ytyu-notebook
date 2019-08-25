# 读Axios源码xhr.js, 顺带了解XMLHttpRequest

## XHR前提资料

XMLHTTPRequest是一个Web环境中的对象。

在使用Chrome的devtool调试时，network的筛选项有一个`xhr`，那就是XMLHTTPRequest的缩写。同样，AJAX技术中的‘X’也是指代XHR。

[XMLHttpRequest - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)

[XMLHttpRequest Level 2 使用指南 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)

## Axios的Feature

截取Axios的Feature，有这么几条。

>- Make [XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) from the browser
>- Make [http](http://nodejs.org/api/http.html) requests from node.js
>- Supports the [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) API
>- Client side support for protecting against [XSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery)

