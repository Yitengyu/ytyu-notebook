# 组件model辅助装饰器

### 前言

在使用vuejs的开发中，有时候需要在组件上使用`v-model`，所以我们在子组件实现中常常有如下代码这样的实践，使用get和set能够方便地使属性`this._value`与`this.value`时刻保持一致。

```jsx
/**
* luobinhang (November 5th, 2018 9:55am)
* A - 自定义日期选择器、入离管理
*/
@Component
class CustomDatePicker extends Vue {
  @Prop({
    default: ''
  })
  private value!: string

  get _value () {
    return this.value
  }

  set _value (value: string) {
    this.$emit('input', value)
  }
}
```

因为这种实践的可重用性，所以我想到可以使用装饰器来把这部分内容封装起来。



### 实现

装饰器的实现基于`vue-class-component`，仿照了`vue-property-decorator`的实现，代码如下:

```jsx
import { createDecorator, VueDecorator } from 'vue-class-component'
import Vue, { ComponentOptions } from 'vue'

export function HoldModel (event: string = 'input', prop: string = 'value'): VueDecorator {
  return createDecorator((componentOptions: ComponentOptions<Vue>, k: string) => {
    componentOptions.model = { prop, event };

    ((componentOptions.props || (componentOptions.props = {})) as any)[prop] = {};
    (componentOptions.computed || (componentOptions.computed = {}))[k] = {
      get: function () { return (this as any)[prop]},
      set: function (val: any) { return (this as any)['$emit'](event, val) }
    }
  })
}
```

使用了该装饰器后，有以下效果

```jsx
@Component
class ChildComponent extends Vue {
  @HoldModel()
  private example!: number
}

/* 近似于 ↓ */

@Component
class ChildComponent extends Vue {
  @Prop()
  private value!: any
  
  private get example () {
      return this.value
  }

  private set example (val: any) {
      this.$emit('input', val)
  }
}
```

个人认为可读性与便捷性还是有不少提升的，并支持了原本的`@Model`的功能，能够对model事件类型与相关联的prop进行定义。但自然也减少了灵活性：

1. 在ts的环境中无法直接访问`this.value`了。`
2. 无法在model的set和get中加入其它操作了。

不过以上两个缺点，也正是这种实践所建议的。



另外需要注意的是，在tsx的环境中，如果使用插件`babel-plugin-jsx-v-model`实现`v-model`的话，由于这个插件只支持如下默认的'value'属性与'input'事件组合，所以在使用以上装饰器时，不能传别的参数。

```jsx
<component value={this.foo} onInput={$$v => {this.foo = $$v}}/>
```



### 总结

总之是实现了一个有助于在组件上使用`v-model`的装饰器，不知道它的适用性是不是好。欢迎探讨。