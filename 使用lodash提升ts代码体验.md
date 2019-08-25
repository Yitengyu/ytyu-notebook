# 使用lodash提升代码体验

> 注意！本篇文章的食用前提是，对原生JS的各种写法都比较熟悉，不容易被lodash带跑偏。

### 使用lodash的迭代缩写（iteratee shorthand）

代码改造示例

```jsx
// map抽取对象数组的某属性
this.selectedGoals.map((item: PerformanceGoal) => item.id)
// 使用lodash
_.map(this.selectedGoals, 'id')
```

```jsx
// filter筛选对象数组某属性为真
let res = this.data.filter((dataItem: KinshipVacationVOS, index: number) => {
   return dataItem.selected
})
// 使用lodash
_.filter(this.data, 'selected')
```

```jsx
// filter筛选对象数组某属性为假
this.approverInfo.filter((approver: DismissApproverInfo) => !approver.groupIndex)
// 使用lodash
_.reject(this.approverInfo, 'groupIndex')  // 原生js没有相应的reject，所以也不太建议用

```

```jsx
// filter筛选对象数组某属性为特定值
result.filter(row => row.isManger === 1)
// 使用lodash
_.filter(result, { 'isManger': 1 })
```



### lodash的链式操作

有人说使用lodash而非原生js的话，一些链式操作不太友好。

这个说法我还是蛮支持的，但架不住lodash的一些函数比较方便，可读性又好。

比如以下两个哪种更好读？

```jsx
// 原生js
return scoreData
  .filter(item => scores.indexOf(item.score) > -1)
  .map(item => item.total)
  .reduce((a, b) => a + b)

// 使用lodash
return _
  .chain(scoreData)
  .filter(item => _.includes(scores, item.score))
  .sumBy('total')
  .value()
```

