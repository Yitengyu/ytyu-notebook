## 简介JavaScript事件循环

#### 从单线程讲起

> A Job is an abstract operation that initiates an ECMAScript computation when no other ECMAScript computation is currently in progress. 
>
> Every ECMAScript implementation has at least the Job Queues defined in [Table 25](https://tc39.github.io/ecma262/#table-26) 
>
> 1. ScriptJobs： Jobs that validate and evaluate ECMAScript [Script](https://tc39.github.io/ecma262/#prod-Script) and [Module](https://tc39.github.io/ecma262/#prod-Module) source text.
> 2. PromiseJobs： Jobs that are responses to the settlement of a Promise
>
> [ES2019, Jobs and Job Queues]: https://tc39.github.io/ecma262/#sec-jobs-and-job-queues

js一次只能执行一个任务（Job）。

一带而过运行栈（excutation stack），代码报错与尾调用。

// 参考网站（回顾温习）

// 代码

单任务黑盒。

——JS引擎中没有事件循环

#### 事件循环是环境提供的

从浏览器环境来说。

异步的接口是浏览器提供的，微任务宏任务是浏览器定义的，安排给JS引擎去处理。

约定了执行的顺序。

#### 应用



