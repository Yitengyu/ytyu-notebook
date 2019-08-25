## vuejs父子组件的生命周期

### 实验案例

请猜测以下父子组件初始化时的输出和点击按钮后的输出

父组件：

```jsx
@Component
export default class parent extends Vue {
  private prop: string = 'prop'
  $refs!: {
    child: ChildComponent
  }
  beforeCreate () {
    console.log('parent beforeCreate')
  }
  created () {
    console.log('parent created')
  }

  mounted () {
    console.log('parent mounted')
  }

  modifyProp () {
    this.prop += ' updated'
    console.log('parent modifyProp, child prop: ' + this.$refs.child.getProp())
  }

  render () {
    return <div>
      parent
      <ChildComponent ref='child' prop={this.prop}/>
      <button onClick={this.modifyProp}>修改prop</button>
    </div>
  }
}

```

子组件：

```jsx
@Component
export default class VuejsChildComponent extends Vue {
  private prop: string = 'prop'
  $refs!: {
    child: ChildComponent
  }
  beforeCreate () {
    console.log('parent beforeCreate')
  }
  created () {
    console.log('parent created')
  }

  mounted () {
    console.log('parent mounted')
  }

  modifyProp () {
    this.prop += ' updated'
    console.log('parent modifyProp, child prop: ' + this.$refs.child.getProp())
  }

  render () {
    return <div>
      parent
      <ChildComponent ref='child' prop={this.prop}/>
      <button onClick={this.modifyProp}>修改prop</button>
    </div>
  }
}
```

------

### 实验结果

在按钮`修改prop`点击之前

```bash
parent beforeCreate
parent created
child beforeCreate
child computed
prop: prop
data: prop
getComputed: true123
child created
child computed
child mounted
parent mounted
```

在点击`修改prop`之后

```
parent modifyProp, child prop: prop
child beforeUpdated, prop: prop updated
child updated, prop: prop updated
```

### 初步简单结论

1. 父组件在子组件之前create
2. 子组件在父组件之前mount
3. prop的传递不是以get set钩子的方式，具体实现还未知。



欢迎一起探讨关于父子组件实例化与更新的源码实现问题。