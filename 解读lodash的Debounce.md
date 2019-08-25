# 解读Lodash的deboucne

# 前言

`Debounce`防抖是前端基本的算法，有很多的应用。最近看到lodash的debounce代码发现比想象中复杂，所以想探究一下复杂的原因（抛去提供了更多的接口的因素）。

## 初级版本

手写的debouce，最简单的可以是这样，不到十行代码就实现了基本的功能。

```javascript
function debounce(func, wait) {
  let timer
  return function() {
    timer && clearTimeout(timer)
    timer = setTimeout(() => func.apply(this, arguments), wait)
  }
}
```

