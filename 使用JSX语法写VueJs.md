### 使用JSX语法写VueJs



### 为什么要用TSX语法写？

在模板中使用的变量可以进行校验与补全，而模板语法需要用字符串。



#### 为什么可以用JSX语法写VueJs？

1. Vue本身提供了[渲染函数](https://cn.vuejs.org/v2/guide/render-function.html)

createElement函数返回 VNode

> Vue 通过建立一个**虚拟 DOM** 对真实 DOM 发生的变化保持追踪。请仔细看这行代码：
>
> ```
> return createElement('h1', this.blogTitle)
> ```
>
> `createElement` 到底会返回什么呢？其实不是一个*实际的* DOM 元素。它更准确的名字可能是 `createNodeDescription`，因为它所包含的信息会告诉 Vue 页面上需要渲染什么样的节点，及其子节点。我们把这样的节点描述为“虚拟节点 (Virtual Node)”，也常简写它为“VNode”。“虚拟 DOM”是我们对由 Vue 组件树建立起来的整个 VNode 树的称呼。



> 你可以通过 [`this.$slots`](https://cn.vuejs.org/v2/api/#vm-slots) 访问静态插槽的内容，得到的是一个 VNodes 数组

> 也可以通过 [`this.$scopedSlots`](https://cn.vuejs.org/v2/api/#vm-scopedSlots) 访问作用域插槽，得到的是一个返回 VNodes 的函数

1. Babel插件提供了对JSX语法编译成对应渲染函数的支持



#### 相关环境

1. @vue/cli-plugin-babel
   1. @vue/babel-preset-app
   2. [babel-plugin-transform-vue-jsx](https://github.com/vuejs/babel-plugin-transform-vue-jsx)
2. babel-plugin-jsx-v-model
3. babel-plugin-vue-jsx-sync



#### 实验环境

如果想要看到具体的jsx的语法会被转译成什么样子，可以在babel官网的[Try it out](https://babeljs.io/repl)中，添加插件`babel-plugin-transform-vue-jsx`后进行实验，可以直观看到转换后的效果。



#### 语法DEMO

插件babel-plugin-transform-vue-jsx文档中有如下demo

```jsx
render (h) {
  return (
    <div
      // normal attributes or component props.
      id="foo"
      // DOM properties are prefixed with `domProps`
      domPropsInnerHTML="bar"
      // event listeners are prefixed with `on` or `nativeOn`
      onClick={this.clickHandler}
      nativeOnClick={this.nativeClickHandler}
      // other special top-level properties
      class={{ foo: true, bar: false }}
      style={{ color: 'red', fontSize: '14px' }}
      key="key"
      ref="ref"
      // assign the `ref` is used on elements/components with v-for
      refInFor
      slot="slot">
    </div>
  )
}
```

以上写法会被转译成

```jsx
render (h) {
  return h('div', {
    // Component props
    props: {
      msg: 'hi'
    },
    // normal HTML attributes
    attrs: {
      id: 'foo'
    },
    // DOM props
    domProps: {
      innerHTML: 'bar'
    },
    // Event handlers are nested under "on", though
    // modifiers such as in v-on:keyup.enter are not
    // supported. You'll have to manually check the
    // keyCode in the handler instead.
    on: {
      click: this.clickHandler
    },
    // For components only. Allows you to listen to
    // native events, rather than events emitted from
    // the component using vm.$emit.
    nativeOn: {
      click: this.nativeClickHandler
    },
    // class is a special module, same API as `v-bind:class`
    class: {
      foo: true,
      bar: false
    },
    // style is also same as `v-bind:style`
    style: {
      color: 'red',
      fontSize: '14px'
    },
    // other special top-level properties
    key: 'key',
    ref: 'ref',
    // assign the `ref` is used on elements/components with v-for
    refInFor: true,
    slot: 'slot'
  })
}
```



**但该demo没有说明所有接口**，更全的可以参考Vue的ts接口声明，如下

```jsx
/* /node_modules/vue/type/index.d.ts */
interface CreateElement {
  (tag?: string | Component<any, any, any, any> | AsyncComponent<any, any, any, any> | (() => Component), children?: VNodeChildren): VNode;
  (tag?: string | Component<any, any, any, any> | AsyncComponent<any, any, any, any> | (() => Component), data?: VNodeData, children?: VNodeChildren): VNode;
}
```

```jsx
/* /node_modules/vue/type/vnode.d.ts */
export interface VNodeData {
  key?: string | number;
  slot?: string;
  scopedSlots?: { [key: string]: ScopedSlot };
  ref?: string;
  tag?: string;
  staticClass?: string;
  class?: any;
  staticStyle?: { [key: string]: any };
  style?: object[] | object;
  props?: { [key: string]: any };
  attrs?: { [key: string]: any };
  domProps?: { [key: string]: any };
  hook?: { [key: string]: Function };
  on?: { [key: string]: Function | Function[] };
  nativeOn?: { [key: string]: Function | Function[] };
  transition?: object;
  show?: boolean;
  inlineTemplate?: {
    render: Function;
    staticRenderFns: Function[];
  };
  directives?: VNodeDirective[];
  keepAlive?: boolean;
}
```

