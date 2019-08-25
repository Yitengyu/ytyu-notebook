# 单入口vue-router路由栈管理

## 背景

App的webview中有一个以`vue-router`做路由控制的SPA网页，没有提供关闭退出的功能，用户从首页进入后，只能不断返回到首页才能退出。

常常出现这样的情景，用户的访问路径如下

```
A ---push---> B ---push---> C
或
A ---push---> C
```

而在C页面希望直接返回到A页面，那么常规做法中，就需要判断C的上一个页面是什么，然后使用`this.$router.go(-1)`或`this.$router.go(-2)`来返回。

这种状态的判断与路由设置的关联性很高，随着业务的变动非常容易错。

而作为SPA，并且用户只能返回，而不能直接输入网址。那么理论上**路由的轨迹应该是可以完全保存下来，进而实现返回时的正确跳转。**

## 可行性探究

通过阅读`vue-router`的代码可知，在hash模式下，并且浏览器支持`pushstate`（已被广泛支持），`vue-router`内部维护了一个`_key`与state的关系

```ts
// key的生成函数
function genKey (): string {
  return Time.now().toFixed(3)
}

// push操作完成后，生成新key
_key = genKey()
history.pushState({ key: _key }, '', url)

// replace操作不生成key
history.replaceState({ key: _key }, '', url)

// 返回(popstate)时。很奇怪的是，这个逻辑仅有配置中scrollBehavior不为空才执行。
window.addEventListener('popstate', e => {
  // ...
  if (e.state && e.state.key) {
    setStateKey(e.state.key) // 重置当前_key为返回到的state页面的key
  }
})

// next(false)拦截路由跳转后
// 相当于popstate + push, 等同于将当前state的key更新
```

由以上，我们可以认为，**每个不同的state对应不同的key**

## 方案设计

总的方案就是**维护一个记录state的访问历史栈**

```
A ---push---> B ---push---> C
或
A ---push---> C
```

在上面两种情况中，如果在C页面想返回到A页面，就查询A在历史栈中的位置，以此确定`this.$router.go(n)`中这个`n`的值。

`访问历史栈`的数据结构如下：

```ts
// key对应vue-router维护的key
// paths是这个state上出现过的路由路径. 比如A replace 到B, 那么AB在同一个state上，对应同一个key
const historyStates: { key: string, paths: string[] }[] = []
```

同时维护`当前访问帧`

```ts
// 当前访问帧表示当前state的对应状态
let currentState: { key: string, paths: string[] } = { key: '', paths: [] }
```

算法如下：

```
// 用户前进（push）
1. 将currentState入栈historyStates
2. 更新currentState

// 用户后退
1. 将historyStates出栈到后退的对应的位置
2. 更新currentState
```

