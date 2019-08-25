### Vuejs是怎样实现双向绑定的？

```vue
<template>
  <!-- 一次点击后，这个dom元素会被改变几次？ -->
  <div @click="onClick">{{foo}}</div>
</template>
<script>
export default {
  data () {
    return {
      foo: 123
    }
  },
  methods: {
    onClick () {
      debugger
      ++ this.foo // 第一行
      ++ this.foo // 第二行
      setTimeout(() => ++this.foo)	// 第三行
      Promise.resolve().then(() => ++this.foo) // 第四行
    }
  }
}
</script>

```

1. 一个组件是一个对象，变量存在对象中
2. 对象的属性如果与dom有绑定，会成为其一个内禀属性。
3. 当变量更新后，会通知相应的dom更新。
4. dom更新不是同步的。所以在主脚本运行过程中，无论example更新多少次，dom只更新一次。

这里只会有两次更新。第一次是一二四行导致的修改，第二次是第三行导致的修改。为什么呢？