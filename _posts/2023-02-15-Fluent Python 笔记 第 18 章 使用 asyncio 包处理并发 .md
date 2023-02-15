---
title: Fluent Python 笔记 第 18 章 使用 asyncio 包处理并发
tags: Python
pageview: true
---
本章介绍 asyncio 包，这个包使用事件循环驱动的协程实现并发。

## 18.1 线程与协程对比
Python 没有提供终止线程的 API，这是有意为之的。若想关闭线程，必须给线程发送消息。

交给 asyncio 处理的协程要使用` @asyncio.coroutine` 装饰。这不是强制要求，但是强烈建议这么做。因为这样能在一众普通的函数中把协程凸显出来，也有助于调试:如果还没从中产出值，协程就被垃圾回收了(意味着有操作未完成，因此有可能是个缺陷)，那就可以发出警告。这个装饰器不会预激协程。

使用 `yield from asyncio.sleep(.1)` 代替 `time.sleep(.1)`，这样的休眠不会阻塞事件循环。

- `asyncio.Task` 对象差不多与 `threading.Thread` 对象等效。Victor Stinner(本章的特约技术审校)指出“，Task 对象像是实现协作式多任务的库(例如 gevent)中的绿色线程(green thread)”。
- Task 对象用于驱动协程，Thread 对象用于调用可调用的对象。
- Task 对象不由自己动手实例化，而是通过把协程传给 `asyncio.async(...)` 函数或 `loop.create_task(...)` 方法获取。
- 获取的 Task 对象已经排定了运行时间(例如，由 asyncio.async 函数排定);Thread 实例则必须调用 start 方法，明确告知让它运行。
- 在线程版 supervisor 函数中，slow_function 函数是普通的函数，直接由线程调用。在异步版 supervisor 函数中，slow_function 函数是协程，由 yield from 驱动。
- 没有 API 能从外部终止线程，因为线程随时可能被中断，导致系统处于无效状态。如果想终止任务，可以使用 Task.cancel() 实例方法，在协程内部抛出 CancelledError 异常。协程可以在暂停的 yield 处捕获这个异常，处理终止请求。
- supervisor 协程必须在 main 函数中由 `loop.run_until_complete` 方法执行。

## 18.1.1 asyncio.Future：故意不阻塞
`asyncio.Future` 类的 `.result()` 方法没有参数，因此不能指定超时时间。此外，如果调用 `.result()` 方法时 future 还没运行完毕，那么 `.result()` 方法不会阻塞去等待结果，而是抛出 `asyncio.InvalidStateError` 异常。

获取 `asyncio.Future` 对象的结果通常使用 `yield from`

使用`yield from`处理 future，等待 future 运行完毕这一步无需我们关心，而且不会阻塞事 件循环，因为在 asyncio 包中，`yield from` 的作用是把控制权还给事件循环。

注意，使用`yield from`处理`future`与使用`add_done_callback`方法处理协程的作用一样: 延迟的操作结束后，事件循环不会触发回调对象，而是设置 `future` 的返回值;而 `yield from` 表达式则在暂停的协程中生成返回值，恢复执行协程。

总之，因为 `asyncio.Future` 类的目的是与 `yield from` 一起使用，所以通常不需要使用以下方法。

- 无需调用 `my_future.add_done_callback(...)`，因为可以直接把想在 `future` 运行结束后 执行的操作放在协程中 `yield from my_future` 表达式的后面。这是协程的一大优势:协 程是可以暂停和恢复的函数。
- 无需调用`my_future.result()`，因为`yield from`从`future`中产出的值就是结果(例如， `result = yield from my_future`)。

## 18.1.2 从 future、任务和协程中产出

对协程来说，获取 Task 对象有两种主要方式：

#### asyncio.async(coro_or_future, *, loop=None)

这个函数统一了协程和 future:第一个参数可以是二者中的任何一个。如果是 Future 或 Task 对象，那就原封不动地返回。如果是协程，那么 async 函数会调用 `loop.create_ task(...)` 方法创建 Task 对象。loop= 关键字参数是可选的，用于传入事件循环；如果 没有传入，那么 async 函数会通过调用 `asyncio.get_event_loop()` 函数获取循环对象。

#### BaseEventLoop.create_task(coro)

这个方法排定协程的执行时间，返回一个 `asyncio.Task` 对象。如果在自定义的 BaseEventLoop 子类上调用，返回的对象可能是外部库(如 Tornado)中与 Task 类兼容的某个类的实例。



















![9](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/9.png)
