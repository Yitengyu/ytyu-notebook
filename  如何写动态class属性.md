### JSX - 如何写动态class

常见动态class属性的写法有以下几种：

#### 普通字符串拼接

```react
<div class={'class-a ' + status ? 'class-b' : 'class-c'></div>
```

#### 模板字符串拼接

```react
<div class=`class-a ${status ? 'class-b' : 'class-c'}`></div>
```

#### 带上变量

*此处的status不是true或false*

```react
<div class=`class-a class-${status}`></div>
```

#### 使用staticClass

```react
<div 
    staticClass='class-a' 
    class={status ? 'class-b' : 'class-c'}>
</div>
```

#### 使用数组语法

*此处的status不是true或false*

```react
<div class={['class-a', `class-${status}`]}></div>
```

#### 使用对象语法

```react
<div class={{'class-a': true, 'class-b': status, 'class-c': !status }}></div>
```



### 实践总结

可以看出不同场景适用的写法并不一样，并且是可以结合的

#### 有固定class的, 推荐使用staticClass

```react
<div 
    staticClass='static-class' 
    class={status ? 'class-b' : 'class-c'}>
</div>
```



#### 根据状态变化增减class，推荐使用对象写法

```react
<div class={'class-to-add': status}></div>
```

