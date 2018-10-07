### JavaScript事件循环

> 1. 第一手参考资料是什么？
> 2. 事件循环在js语言中占什么位置？
> 3. 一个循环中发生了什么事情？
> 4. 代码举例说明
> 5. 应用： vue1.x 到 vue2.x的变化

#### 0. 准备工作

**第一手资料**： [HTML标准](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)

**术语定义**

> [HTML标准]: https://html.spec.whatwg.org/multipage/webappapis.html#event-loops

 	1. 事件循环（event loop）
 	2. 两种事件循环： 浏览器环境（browsing context）和woker
 	3. 一个事件循环有一个或者多个任务队列（task queue）
 	4. 执行上下文，执行栈（JavaScript execution stack)，与编译原理一起看



#### 1. Chrome环境中的事件循环

> [菲利普·罗伯茨：到底什么是Event Loop呢？ | 欧洲 JSConf 2014]: https://www.youtube.com/watch?v=8aGhZQkoFbQ&amp;index=3&amp;list=PLNTAOpJOozeMGH8HorB9Oijo0_kbWbpm0&amp;t=58s

![event-loop](/Users/ytyu/Library/Mobile Documents/com~apple~CloudDocs/Documents/notes/assets/event-loop.png)

这张图基本解释了Chrome环境中JS执行栈与任务队列的关系。在搜这张图时，一些类似的更清晰的图，去掉了左上角的Chrome图标，也把JS图标中的V8去掉了，所以我还是用了这个比较模糊的。

如Philip提出的，**在V8引擎的代码中，没有setTimeout这些函数**，这些函数是浏览器提供的API。因此在这张图中强调是chrome环境是有必要的，比如在Node环境中就没有图中的DOM API，setTimeout相关的timer实现也会不一样。

#### 2. 一段脚本的运行

Philip在视频中没有提及的一点，是他所讨论的前提应该是一段脚本（比如一个script标签中的js代码）的运行的模型。

##### 2.1 单线程的JavaScript与执行栈

**JavaScript是单线程的，体现在它仅有一个执行栈**，执行栈就是编译原理中的概念，空栈进入一个主函数后，为主函数创建执行上下文，执行主函数，遇到子函数就为子函数创建新的执行上下文入栈执行，以此类推；子函数完成后出栈，继续执行其父函数，直到栈空。

如以下代码会使执行栈溢出

```javascript
function foo() {
    foo()
}
foo() // Uncaught RangeError: Maximum call stack size exceeded
```

可以试试栈深最大是多少，我在我的Chrome下跑这段代码

```javascript
function computeMaxCallStackSize() {
    try {
        return 1 + computeMaxCallStackSize();
    } catch (e) {
        // Call stack overflow
        return 1;
    }
}
computeMaxCallStackSize() //12531
```

知道这个概念后，可以去看看ES6的尾调用优化，其思想就是减少执行栈的深度增加。

##### 2.2 实现异步调用

所以setTimeout这些函数不存在于引擎之中就很自然了，因为执行栈顶只能有一个函数，不能一边执行timer一边继续执行主程序。

**实现（异步）回调的关键是两个，一是js运行环境提供的API支持，二是js本身的任务队列机制。**在此以浏览器作为运行环境，一段带异步操作的代码运行过程如下。

1. 脚本代码按顺序运行
2. 遇到异步操作，调用浏览器API，交由浏览器执行，同时脚本继续运行。
3. 当浏览器API对应操作完成后将回调函数放入任务队列。
4. 主脚本运行完成后栈空，检查任务队列，若任务队列不为空，将队列头部任务入栈；
5. 拿到的队列任务在栈中执行完毕后，再次检查任务队列，若任务队列不为空，将队列头部任务入栈；
6. ~~直到执行完毕~~

在浏览器环境下不会执行完毕，毕竟之前的异步调用可能绑定了DOM事件，我们不知道用户什么时候会操作触发异步调用。所以当栈为空时，应该会循环检查任务队列是否为空。

这里带来两个问题：

1. 任务队列的实现，是在JS引擎内部吗？
2. 当栈为空，任务队列也为空时，多久检查一次任务队列是否为空？还是任务队列不空时将提醒执行栈？

尝试解答：

1. 出于设计原则的考虑，任务队列的实现应该在引擎之外；除此以外，诸如宏任务微任务在不同的浏览器以及在node上有不同的实现，所以任务队列的实现应该独立于引擎。
2. JS引擎不知道外面的任务队列。

*补： 目前认为以上的猜测基本是对的*

#### 3. 几个概念

##### 3.1 ES标准中的Job queue


> A Job is an abstract operation that initiates an ECMAScript computation when no other ECMAScript computation is currently in progress. 
>
> Every ECMAScript implementation has at least the Job Queues defined in [Table 25](https://tc39.github.io/ecma262/#table-26) 
>
> 1. ScriptJobs： Jobs that validate and evaluate ECMAScript [Script](https://tc39.github.io/ecma262/#prod-Script) and [Module](https://tc39.github.io/ecma262/#prod-Module) source text.
> 2. PromiseJobs： Jobs that are responses to the settlement of a Promise
>
> [ES2019, Jobs and Job Queues]: https://tc39.github.io/ecma262/#sec-jobs-and-job-queues

ECMAScript标准中的`Job`，是其内部实现的一种表达，与Event Loop没有直接关系。

JS到目前为止依然是一个并发的语言。其中的`Job Queue`与任务队列无关。自ES6提出的必须有的`PromiseJobs Queue`是Promise实现规范的一部分。

##### 3.2 HTML标准中的Event Loop

> An [event loop](https://www.w3.org/TR/html5/webappapis.html#event-loop) has one or more task queues. A [task queue](https://www.w3.org/TR/html5/webappapis.html#task-queues) is an ordered list of tasks, which are algorithms that are responsible for such work as:
>
> - Events
>
>   Dispatching an `Event` object at a particular `EventTarget` object is often done by a dedicated task.Not all events are dispatched using the [task queue](https://www.w3.org/TR/html5/webappapis.html#task-queues), many are dispatched during other tasks.
>
> - Parsing
>
>   The [HTML parser](https://www.w3.org/TR/html5/syntax.html#html-parser) tokenizing one or more bytes, and then processing any resulting tokens, is typically a task.
>
> - Callbacks
>
>   Calling a callback is often done by a dedicated task.
>
> - Using a resource
>
>   When an algorithm [fetches](https://fetch.spec.whatwg.org/#concept-fetch) a resource, if the fetching occurs in a non-blocking fashion then the processing of the resource once some or all of the resource is available is performed by a task.
>
> - Reacting to DOM manipulation
>
>   Some elements have tasks that trigger in response to DOM manipulation, e.g., when that element is [inserted into the document](https://www.w3.org/TR/html5/infrastructure.html#document-inserted-into).
>
>   [Event-Loop]: https://www.w3.org/TR/html5/webappapis.html#event-loops

从以上可知，

1. 任务队列可能不仅仅有一个。
2. 任务可以有很多种类。

令人疑惑的一点是标准中的https://www.w3.org/TR/html5/webappapis.html#integration-with-the-javascript-job-queue

##### 3.3 微任务(microtask)

> Each [event loop](https://www.w3.org/TR/html5/webappapis.html#event-loop) has a microtask queue. A microtask is a [task](https://www.w3.org/TR/html5/webappapis.html#tasks) that is originally to be queued on the [microtask queue](https://www.w3.org/TR/html5/webappapis.html#microtask-queue) rather than a [task queue](https://www.w3.org/TR/html5/webappapis.html#task-queues).

在任务队列的一个任务完成后，会检查微任务队列，如果微任务队列不为空，则将微任务队列中的任务全部完成后，再将执行权交还。

标准如此，但是将哪些来源的任务放入微任务队列，就是一个实现的问题了。

#### 4. 一个伪代码模型

> [Further Adventures of the Event Loop - Erin Zimmer - JSConf EU 2018]: https://www.youtube.com/watch?v=u1kqx6AenYw&amp;list=PL37ZVnwpeshG2YXJkun_lyNTtM-Qb3MKa&amp;index=21&amp;t=0s

```javascript
while (true) {
    let queue = getNextQueue()
    let task = queue.pop()
    execute(task)
    
    while (microtaskQueue.hasTasks()) {
        doMicrotask()
    }
    if (isRepaintTime()) {
        repaint()
    }
}
```



以上代码表现了事件循环是怎么样的。（注意，任务执行压入执行栈在这里是隐去的）

1. 可能有不止一个任务队列
2. 相同来源的任务排在一个任务队列中。
3. 一个任务队列中的任务按照FIFO的顺序执行。
4. 当没有任务或者任务完成后，如果微任务队列中存在微任务，需要将微任务执行完。
5. 如果微任务没执行完，又产生的新的微任务，新的微任务也会在本次循环中执行。
6. 浏览器（如果没有阻塞）每16秒重绘一次页面。