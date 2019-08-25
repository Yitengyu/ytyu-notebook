# tsx环境中vue组件如何实现prop代码提示

## 背景

vuejs的tsx环境由以下工具组成：

1. TypeScript
2. @vue/babel-preset-app（包含@vue/babel-preset-jsx）
3. vue-class-component

之前用React写JSX时，发现组件只要声明了`PropsType`，在JSX使用组件时，就能得到props的代码提示，写起来非常愉快。于是就想vuejs的TSX环境一定也可以.

查阅了一下ts介绍jsx的[篇章](https://www.typescriptlang.org/docs/handbook/jsx.html)，答案就呼之欲出了。JSX组件上的属性，类型为`JSX.ElementAttributesProperty`，只要声明对应的属性，就可以实现代码提示了



## 简单版本

```tsx
declare global {
  namespace JSX {
    interface ElementAttributesProperty {
      $__propTypes: any	// 声明ElementAttributesProperty对应的是属性$__propTypes
    }
  }
}

class tsComponent<P> extends Vue {	// 构建一个类，能注入prop的类型
  $__propTypes: P
}
  
// 使用
import { Component, Vue, Props } from 'vue-property-decorator'

@Component
export default class App extends Vue<{ name: string }> {
  @Props()
  private name!: string
  // ...
}

<App name="Vue.js" /> // OK
<App /> // ERROR
<App name={123} /> // ERROR
```

可以看到这个版本基本就实现了功能，能够在写组件时进行props类型检查与代码提示。
但有一个很明显的缺陷——**需要把props写两遍**

分析一下，可以知道，我们只是注入了类型，但vue的组件声明还是需要注册props的。
那么思路很明确了：

1. 能不能在注册props的时候注入类型？
2. 或者，在注入类型的同时注册props

直观感觉上，第一个方向实现出来会比较友好。改写`vue-property-decorator`的`Props`装饰器，或者改写`vue-class-component`的`Component`。但实际上并未想到可行办法，因为类型声明是静态的，你不能通过装饰器改变一个类的属性的声明。

所以考虑第二个方法。

## Demo版本

```tsx
//... 相关声明省略

// 输出的类型
class VueP<CustomProps extends Record<string, any>> extends Vue {
  $__propTypes!: CustomProps
}

// 注册props
export function tsComponent<PC extends Record<string, any>> (propsClass: { new (...args: any[]): PC }) {
  let x = new propsClass()
  let props: { [key: string]: any } = {}
  _.forEach(x, (v, key) => {
    if (v === undefined) {
      props[key] = v
    } else {
      // 构造时初始化的值作为默认值
      props[key] = { default: _.isArray(v) || _.isObject(v) ? () => v : v}
    }
  })
  return Vue.extend({
    props
  }) as unknown as VueConstructor<VueP<PC>>
}

// 我们只需要声明一次props的类就可以了
class PropTypes {
  name: string = '123'
}
@Component
export default class App extends tsComponent<PropTypes>(PropTypes) {
  // ...
}

<App name="Vuejs" /> // OK
<App /> // ERROR
<App name={123} /> // ERROR
```

可以看到这个思路是可行的。

但是投入生产的话，还是会有许多问题：

1. 组件内怎么ts友好地访问prop？
2. 组件使用时，class等内置类型的检测怎么办？
3. 能不能把事件处理函数的也声明了？
4. 想在组件上用一个其他值，比如`data-id`，还一定要声明一下吗？

## 优化版本

```ts
import Vue, { VueConstructor, VNodeData } from "vue";
import _ from 'lodash'

type AllRequiredProps<T> = { [L in keyof T]-?: T[L] };

// 将内置的参数声明
type KnownAttrs = Pick<
  VNodeData,
  "class" | "staticClass" | "key" | "ref" | "slot" | "scopedSlots"
> & {
  style?: VNodeData["style"] | string;
  id?: string;
  refInFor?: boolean;
  domPropsInnerHTML?: string;
};

// Events设置默认类型，可以不写
class VueP<CustomProps extends Record<string, any>, Events extends Record<string, any> = {}> extends Vue {
  // 保留Record<string, any>提供自定义空间
  $__propTypes!: CustomProps & Events & KnownAttrs & Record<string, any>
  // 声明$props供组内访问，对于props，没有可选参数（可选参数也要声明默认值）
  $props!: AllRequiredProps<CustomProps>
}

export function tsComponent<PC extends Record<string, any>, Events extends Record<string, any> = {}> (propsClass: { new (...args: any[]): PC }) {
  let x = new propsClass()
  let props: { [key: string]: any } = {}
  _.forEach(x, (v, key) => {
    if (v === undefined) {
      props[key] = v
    } else {
      props[key] = { default: _.isArray(v) || _.isObject(v) ? () => v : v}
    }
  })
  return Vue.extend({
    props
  }) as unknown as VueConstructor<VueP<PC, Events>> & AllRequiredProps<PC>
}
```

这样子的话，就可以如下使用

```tsx
// 无Events
class PropTypes {
  name: string = '123'
  disabled?: boolean = false
}
class Events {
  onConfirm?: (msg: string) => any
}
@Component
export default class App extends tsComponent<PropTypes>(PropTypes) {
  render () {
    let { name } = this.$props
    return <div onClick={() => { this.$emit('confirm', 'hello')}}>{name}</div>
  }
}

<App name="Vuejs" disabled onConfirm={msg => console.log(msg)}/> // OK
<App name="Vuejs" class='xls'/> // OK
<App name={123} /> // ERROR
<App /> // ERROR
```

## 总结

总得来说只是一个语法糖的包装，可能能提高一点点组件使用时的效率。

也有一些问题没有解决，比如组件内访问props只能使用`this.$props.x`这样的方式了，并不是很书写友好。

不过提出思路，并且解决，这个过程比较有趣。
并且在这个过程中能够让自己去整合一些知识，比如ts的类型声明。