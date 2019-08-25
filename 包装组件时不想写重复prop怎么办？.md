# 包装组件时不想写重复prop怎么办？

## 背景

会有这样的场景，我们需要对一个已存在的组件进一步封装，可能是组合，可能是添加数据逻辑，但同时希望能够“继承”props，于是出现如下场景：

```tsx
@Component
class OutterComponent extends Vue {
    @Prop() prop1!: type
	@Prop() prop2!: type
    @Prop() prop3!: type
    
    render () {
        return (
            <InnerComponent prop1={this.prop1} prop2={this.prop2} prop3={this.prop3}/>
        )
    }
}
```

我们需要把上文中的InnerComponent的所有Prop重复一遍（虽然这也有好处，比如我们可以重新设定默认值，比如限制一些prop的使用）。

## 解决思路

在模板语法的写法中，可以使用`v-bind="$attrs"`的写法（可以看到vant的field之类组件就是用这种方式的）

> - **类型**：`{ [key: string]: string }`
>
> - **只读**
>
> - **详细**：
>
> 包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 (`class` 和 `style` 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (`class`和 `style` 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件——在创建高级别的组件时非常有用。

在JSX语法的写法中，使用相同的思路，可以使用JSX Spread语法

> JSX spread is supported, and this plugin will intelligently merge nested data properties. 

所以有以下写法

```tsx
// OutterComponent上可以使用InnerComponent的prop
// <OutterComponent prop1={prop1} prop2={prop2} prop3={prop3}/>
@Component
class OutterComponent extends Vue {
    render () {
        return (
            <InnerComponent {...{ props: this.$attrs }}/>
        )
    }
}

// 如果只需要传递一部分
@Component
class OutterComponent extends Vue {
    get childProps () {
        return _.pick(this.$attrs, ['prop1', 'prop2', 'prop3'])
    }
    render () {
        return (
            <InnerComponent {...{ props: this.childProps }}/>
        )
    }
}

// 如果将以上包装成装饰器
@Component
class OutterComponent extends Vue {
    @Attrs(['prop1', 'prop2', 'prop3'])
    childProps!: any

    render () {
        return (
            <InnerComponent {...{ props: this.childProps }}/>
        )
    }
}
```



## 总结

以上，是一种利用\$attrs与JSX Spread语法优化部分代码的思路。

具体应用到生产中，可能还需要一些调整，或者探索更合适的写法。