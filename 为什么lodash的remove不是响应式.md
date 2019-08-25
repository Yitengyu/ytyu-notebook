## 为什么lodash的remove在vuejs中不是响应式的？

#### 问题引出

当我们开发中希望从数组中按照某种筛选条件移除数组的一个元素时，很容易想到使用splice或者filter来操作

```typescript
/* 从数组arr中移除值为val的元素 */
let index = arr.indexOf(val)
index !== -1 && arr.splice(index, 1)	

/* 从数组arr中移除满足predicate条件的元素 */
arr = arr.filter(predicate)
```

可以看到，splice方法的可读性并不好，而且还需要考虑val不是arr的元素的情况；filter可读性还不错，但实际上得到了一个新的数组。比较好的办法是循环使用splice，但那样写就太麻烦了。

所以就有了lodash这种原生js库来帮助我们。lodash的remove方法语义明确，在后面可以看到它使用一个循环的splice操作实现元素移除。

```typescript
/* 从数组arr中移除满足predicate条件的元素 */
_.remove(arr, predicate)
```

但美中不足的是，如果在vuejs的开发中使用这个方法，你会发现使用这个方法并不能触发vuejs的DOM更新响应。

这是为什么呢？本篇文章不会深入地讨论vuejs双向绑定的实现原理，我们可以简单地认为数组操作后会触发一种机制进行DOM更新。那么问题就转化成了两个问题：

1. 数组操作是怎么触发vuejs响应机制的？
2. lodash的remove实现和普通的数组操作有什么区别？

#### 数组操作是怎么触发vuejs响应机制的？

简单来说，vuejs用修改后的方法替换了观察的数组本身的原型方法，实现了拦截，增加了触发响应的部分。替换原型方法的代码如下：

```javascript
// project: vue
// version: 2.5.6
// file: src/core/observer/index.js
if (Array.isArray(value)) {	// value: 观察的对象
    const augment = hasProto
    ? protoAugment
    : copyAugment
    augment(value, arrayMethods, arrayKeys) // 用arrayMethods替换掉value的原型
    this.observeArray(value)
}
```

其中arrayMethods就是vuejs修改后的原型对象，它的实现代码如下：

```javascript
// project: vue
// version: 2.5.6
// file: src/core/observer/array.js
const arrayProto = Array.prototype
export const arrayMethods = Object.create(arrayProto)

const methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]

/**
 * 修改会改变数组元素的方法，实现拦截
 */
methodsToPatch.forEach(function (method) {
  const original = arrayProto[method]
  def(arrayMethods, method, function mutator (...args) {	// def类似Object.defineProperty
    const result = original.apply(this, args)	// 先执行原方法
    // 省略部分代码
    ob.dep.notify() // 触发响应更新
    return result
  })
})
```

结合两份代码，可以看到如下过程：

1. vuejs在数组原型的基础上，创建了新的原型对象，
2. 修改新的原型对象中的'push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'方法。
3. 将观察的数组的原型替换成新的原型对象。

#### lodash的remove方法的实现

lodash的源码非常易读，其remove的关键实现就是通过筛选条件找到对应值在数组中的index，然后使用`basePullAt`方法将array中对应序号的元素剔除，其中`basePullAt`的方法实现如下

```javascript
// project: lodash
// version: 4.17.10-npm
// file: _basePullAt.js 
var baseUnset = require('./_baseUnset'),
    isIndex = require('./_isIndex');

var arrayProto = Array.prototype;
var splice = arrayProto.splice;	// 关键：使用Array.prototype中的splice方法

function basePullAt(array, indexes) {
  var length = array ? indexes.length : 0,
      lastIndex = length - 1;

  while (length--) {
    var index = indexes[length];
    if (length == lastIndex || index !== previous) {
      var previous = index;
      if (isIndex(index)) {
        splice.call(array, index, 1);	// splice操作
      } else {
        baseUnset(array, index);
      }
    }
  }
  return array;
}
```

#### 结论

看完lodash中remove方法的实现代码，题目问题的答案就很明朗了：

**vue通过改造观察数组的原型方法使它操作对应方法时会触发更新响应，而lodash的remove方法使用Array原型中的splice方法对数组进行操作，因此不会触发响应更新。**

我们也得到了一些更多的问题。

#### 更多的问题

1. 为什么lodash要使用Array原型中的splice方法，而不是直接使用数组对象上的splice？

可能是为了兼容一些类数组对象。但还有一个奇怪的地方，lodash的开发分支上早在17年四月就已经修改basePullAt的实现为直接使用splice，而不是用Array的原型。而npm即使是最新的4.17.11-npm，也还是沿用Array原型中的splice方法。可能是因为npm包需要尽量向前兼容吧。

2. 那有什么办法方便地实现响应式地从数组移除元素呢？

建议自己开发工具包方法。

3. 如果用户在vue中使用`arr.splice(0, 0)` 操作，并不会对原数组产生修改，而同样会触发响应更新，那么是不是会影响效率？

猜测： 使用了key属性与vue的diff算法大概可以让这个效率影响降低，而如果在拦截方法中实现对原数组元素是否变更的需要可能比较影响性能