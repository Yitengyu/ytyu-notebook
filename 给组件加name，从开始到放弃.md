# 给组件加name，从开始到放弃

## 需求背景

希望方便地使用vue的组件类名，而类名会在混淆时被改变，所以希望在编译时能够对其进行保留。

## 工程背景

项目的结构是vue-cli-service + ts + vue + vue-property-decorator，编译的过程是 ts-loader转到esnext级别代码，然后用 babel-loader中的babel-preset-env 转到对应级别的代码，生产环境编译完成后混淆。



## 错误的方向

原想name加在组件类的构造函数上.

目标效果如下

```js
// 原代码
@Component
class Bar extends vue {
    
}

// 插件处理后
@Component
class Bar extends vue {
    name: 'bar'
}
```

所以写了这样的babel插件

```js
'use strict'

module.exports = function (babel) {
  var t = babel.types

  return {
    visitor: {
      Class: function (path) {
        var node = path.node
        var className = node.id.name
        var isComponent = 
        node.superClass === 'vue' 
          || node.body.body.find(memberNode => t.isClassMethod(memberNode) && memberNode.key.name === 'render')

        var isWithName = 
          node.body.body.find(
            memberNode => {
              if (t.isClassMethod(memberNode, { kind: 'constructor'})) {
                return memberNode.body.body.find(
                  exp => 
                  t.isAssignmentExpression(exp.expression) 
                  && t.isMemberExpression(exp.expression.left) 
                  && t.isThisExpression(exp.expression.left.object)
                  && exp.expression.left.property.name === 'name'
                )
              }
            }
          )

        if (!isComponent || isWithName) return

        path.get('body').unshiftContainer(
          'body', 
          t.classProperty(t.Identifier('name'), t.StringLiteral(className), null, [])
        )
      }
    }
  }
}

// AST https://github.com/babel/babylon/blob/master/ast/spec.md
// JSX AST https://github.com/facebook/jsx/blob/master/AST.md
// babel types https://github.com/jamiebuilds/babel-types

```

需要注意的是，vue-cli-service的配置中包含cache-loader，因此在开发过程中需要注意去手动删除.cache目录来让代码生效。

但实际上，愚蠢的我忽略了以上代码的作用其实是在component的data里面塞了一个name成员变量。而vue-class-component中填充name的代码如下。

```js
function componentFactory(Component, options) {
    if (options === void 0) { options = {}; }
    // ...
    options.name = options.name || Component._componentTag || Component.name;
    // ...
}
```

所以并没有什么用。

## 那能不能放到decorator里面去呢

```js
// 原代码
@Component
class Bar extends vue {}

// 编译后
@Component({
    name: 'Bar'
})
class Bar extends vue {}
```

看起来仿佛很容易，但是操蛋的是，ts认为decorator的特性还在商议中，只是一个实验性质的，所以即使ts的target设置成'esnext'，它依旧会把decorator编译成它自己的东西，类似于下面这样

```js
// 原代码
@Component
export default class Bar extends Vue {
  render () {
    return ''
  }
}

// 编译后
let Bar = class Bar extends Vue {
    render() {
        return '';
    }
};
Bar = tslib_1.__decorate([
    Component
], Bar);
```

这样虽然也能用插件改，但是改起来耦合度就太高了，相当不好看。于是想看看ts有没有计划像保留jsx一样，能够把装饰器也保留，丢给babel去处理（人家也能处理装饰器的）。接着就看到了一些[issue](https://github.com/Microsoft/TypeScript/issues/18713)，凉凉。

我放弃，放弃...

