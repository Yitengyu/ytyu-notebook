## PivotElement组件开发心得

目前在项目`PivotElement`中已经有了不少的UI组件，这些组件产生于不同的时期，一部分组件用起来比较舒服，一部分组件用起来并不方便。其中有很多因素，这里写一些开发心得，希望能帮助建立更好的实践。



### 重用性之于组件化

重用性是组件化的目的。所谓重用性，就是组件能够被多次用到。

那么重用性就对组件化提出了要求。

1. 低业务相关，不能只适用于一种业务场景；
2. 既然是UI组件，会有样式，样式要有统一的风格（目前没做到，需要与产品合作）。
3. 友好的接口，统一的命名风格和方便的操控感。



### 组件的分类

目前组件的分类很难，一定程度上说明组件的设计没有一个很好的方法论，这里就简单地从设计想法上分几个类

#### 1. 函数式展示组件（如IconSvg、Avatar）

`函数式组件`内部无状态，输入什么，经过固定的处理后就展示什么，没有副作用，渲染开销也比较低（大约是一般组件的四分之一到三分之一）

因此如果封装常用的样式到组件，并且组件无状态，建议写成函数式组件，比如高频使用的基础展示组件、样式复杂但数据简单的组件等。

#### 2. 基础组件（如Check）

`基础组件`指UI样式通用，不适合进一步拆分的组件。

应该具备学习成本低廉，使用方便，内部状态简单的特点。

#### 3. 组合组件（如SelectGroup）

`组合组件`的业务相关性比基础组件更高，以组合基础组件的方式实现。

应该具有一定的业务友好度。

#### 4. 项目相关组件（如SelectDepart、CityPicker）

`项目相关组件`有很高的项目相关性，但没有业务模块相关性。也就是说，适用于某个项目，而不是只适用于项目中的一部分功能。

项目相关组件最好是由一系列基础组件和组合组件组成的，这样如果模块中有特别的需求，能够以比较小的成本写一个魔改版。



### 组件实现心得

*以下接口指props*

#### 1. 接口名称应该有友好的名称

比如组件`Avatar`的两个接口 avatar与name，前者其实指代头像图片的url，命名就比较糟糕。



#### 2. 样式的自定义不建议放在props中

除了‘z-index’与各种size外，样式的自定义放在props的体验并不友好。比如组件`Check`包含了各种定义样式的接口，名称不友好，使用时也不能在css中方便覆盖，体验很糟糕。



#### 3. 使用v-model

表单组件使用`v-model`的体验很友好，让人清晰地知道最重要的属性是什么。

而一些展示组件，可能使用了`v-model`来控制展示状态，比如`Collapse`，但之前版本的这个组件就遇到一个问题——"在使用组件时不声明`v-model`就无法使用，而声明`v-model`就要再写一个变量，没有必要且不方便"。所以展示组件的设计中应该考虑没有`v-model`也可以使用。



#### 4. 使用默认属性

除了v-model与API以外的Prop应该都有默认属性，让人能够开箱即用。



#### 5. 向前兼容，尽量避免break-change

修改一个已经使用中的组件时，可以新增props作为开关，设置默认值以兼容之前的版本。



#### 6. 处理组件数据结构

一种方法是把组件的数据结构作为interface export出去，比如组件`Breadcrumbs`的做法。

而如果组件依赖API的话，一种方式是建立映射关系，比如组件`SelectSummary`的做法。



### 一些技巧

#### 1. 使用[$scopedSlots](https://cn.vuejs.org/v2/guide/render-function.html#%E6%8F%92%E6%A7%BD)

使用$scopedSlots，可以在使用组件时传入一个函数到slot的位置，函数入参由组件内部提供，以此达到访问组件渲染时数据的目的，在组件的slot进行自定义列表渲染时非常好用。比如组件`SelectGroup`的做法。



#### 2. 组合组件中父子组件的绑定

有时候我们希望基础组件能够单独使用，又能够以松耦合的方式一起使用。比如`CheckGroup`和`Check`，我们希望`Check`可以单独使用，也能在它外面套一个`CheckGroup`统一管理数据。参考Vant的一个做法就是类似如下代码

```jsx
  created () {
    let parent = this.$parent
    while (parent) {
      if (parent.$options.name === 'CheckGroup') {
        this.group = parent as CheckGroup
        break
      }
      parent = parent.$parent
    }
  }
```



#### 3. 使用Mixin

如`PopPage`的做法，但目前还不知道怎么把它export出去。



#### 4. 使用指令与函数管理类名

```jsx
Vue.directive('cls', {
  inserted: function (el, binding, vnode: VNode) {
    let addOnClass: string[] = []
    let { value } = binding
    let prefix = PREFIX + vnode.context!.$options.name
    if (_.isEmpty(value)) {
      addOnClass = [prefix]
    } else if (_.isString(value)) {
      addOnClass = value.split(' ').map((cls: string) => prefix + '__' + cls)
    }

    el.classList
    ? el.classList.add(...addOnClass)
    : el.className += ' ' + addOnClass.join(' ')
  }
})

export const cls = (componentName: string) =>
  (value?: string) => {
    let prefix = PREFIX + componentName
    let classes: string[] = []
    if (_.isEmpty(value)) {
      classes = [prefix]
    } else if (_.isString(value)) {
      classes = value.split(' ').map((cls: string) => prefix + '__' + cls)
    }

    return classes.join(' ')
  }

```

