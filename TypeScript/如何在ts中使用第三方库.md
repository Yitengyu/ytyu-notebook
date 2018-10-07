## 如何在ts中使用第三方库?

>背景介绍：在MS发布的typescript-vue-starter中玩耍，想使用Object.assign，ts报错说
>
>**Error: Property 'assign' does not exist on type 'ObjectConstructor'**



本次仅介绍如何使用Object.assign，其他类型的第三方库之后再讨论。

#### 解决思路一： polyfill

按照以往的习惯，从MDN中拷贝一份Object.assign的polyfill代码，作为一个单独的文件，import进来。发现仍不奏效，错误提示没有变化。

学习之后，猜测，ts的大致原则是**先声明再调用**。尽管引入的polyfill代码实现了Object.assign，但对于编译器而言，它并没有声明。所以使用interface声明了该方法。

```typescript
// xx.d.ts
declare interface ObjectConstructor {  
    assign(target:Object, ...args: Object[]): void 
}
```

引发两个问题：

1. 声明文件在ts中处于什么地位，有什么作用？
2. 为什么是`ObjectConstructor` ?
3. 声明合并在这个过程中起了什么作用？



进而查阅资料（stackoverflow），得到以下两种解决办法：



#### 解决思路二： 断言转换Object的类型

```typescript
(<any>Object).assign({}, {})
```

不过这仅仅是表面上解决了这个问题，将ts以js的方式处理了。

**既然使用了TS，就尽量少用any**



#### 解决思路三： 利用库

之所以在`ObjectConstructor`中没有assign方法，是因为当前的`tsconfig.json`中，使用的target是"es5"，将其改为如下就可以解决问题。

```json
options: {
    target: 'es6'
}
```

但这样会导致最终编译结果不能在不支持es6的环境中运行。更好的办法是编辑配置文件中的lib字段，增加"es2015"（注意，lib原本的默认值是["dom", "es5", "scripthost"]，原先的代码可能依赖这些库）

```json
"lib": [
    "es2015",
    "dom",
    "es5",
    "scripthost"
]
```

**总的来说，这是以上三种方法中最好的。**因为Object.assign本来就是标准库的方法，不应该重写。并且重写后，一方面不一定对，另一方面丢失了可能的注释信息。





### 扩展

一些库 有配套的声明库。

比如lodash和@types/lodash