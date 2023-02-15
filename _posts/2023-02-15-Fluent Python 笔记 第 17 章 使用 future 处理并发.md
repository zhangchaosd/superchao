---
title: Fluent Python 笔记 第 17 章 使用 future 处理并发
tags: Python
pageview: true
---
future 指一种对象，表示异步执行的操作。这个概念的作用很大，是 concurrent.futures 模块和 asyncio 包(第 18 章讨论)的基础。

## 17.1 示例:网络下载的三种风格
### 17.1.1 依序下载的脚本
### 17.1.2 使用 concurrent.futures 模块下载
```
from concurrent import futures

workers = min(MAX_WORKERS, len(cc_list))
with futures.ThreadPoolExecutor(workers) as executor:
    res = executor.map(download_one, sorted(cc_list))
```
### 17.1.3 future 在哪里
从 Python 3.4 起，标准库中有两个名为 Future 的 类：`concurrent.futures.Future` 和 `asyncio.Future`。这两个类的作用相同：两个 Future 类的实例都表示可能已经完成或者尚未完成的延迟计算。

通常情况下自己不应该创建 `future`。

这两种 `future` 都有 `.done()` 方法，这个方法不阻塞，返回值是布尔值，指明 `future` 链接的 可调用对象是否已经执行。客户端代码通常不会询问 `future` 是否运行结束，而是会等待通知。

因此，两个 Future 类都有 `.add_done_callback()` 方法。

还有 `.result()` 方法。在 future 运行结束后调用的话，这个方法在两个 Future 类 中的作用相同：返回可调用对象的结果，或者重新抛出执行可调用的对象时抛出的异常。 可是，如果 future 没有运行结束，result 方法在两个 Future 类中的行为相差很大。对 `concurrency.futures.Future` 实例来说，调用 `f.result()` 方法会阻塞调用方所在的线程，直到有结果可返回。此时，result 方法可以接收可选的 timeout 参数，如果在指定的时间内 future 没有运行完毕，会抛出 TimeoutError 异常。`asyncio. Future.result` 方法不支持设定超时时间，在那个库中获取 future 的结果最好使用 yield from 结构。不过，对 concurrency.futures.Future 实例不能这么做。

```
def download_many(cc_list):
    cc_list = cc_list[:5]
    with futures.ThreadPoolExecutor(max_workers=3) as executor:
        to_do = []
        for cc in sorted(cc_list):
            future = executor.submit(download_one, cc)
            to_do.append(future)
            msg = 'Scheduled for {}: {}'
            print(msg.format(cc, future))

        results = []
        for future in futures.as_completed(to_do):
            res = future.result()
            msg = '{} result: {!r}'
            print(msg.format(future, res))
            results.append(res)
    return len(results)
```
严格来说，我们目前测试的并发脚本都不能并行下载。使用 `concurrent.futures` 库实现的 那两个示例受 GIL(Global Interpreter Lock，全局解释器锁)的限制，而 flags_asyncio.py 脚本在单个线程中运行。

GIL 几乎对 I/O 密集型处理无害。

## 17.2 阻塞型I/O和GIL
CPython 解释器本身就不是**线程**安全的，因此有全局解释器锁(GIL)，一次只允许使用一个**线程**执行 Python 字节码。因此，一个 Python 进程通常不能同时使用多个 CPU 核心。这是 CPython 解释器的局限，与 Python 语言本身无关。标准库中所有执行阻塞型 I/O 操作的函数，在等待操作系统返回结果时都会释放 GIL。

## 17.3 使用 concurrent.futures 模块启动进程
这个模块实现的是真正的并行计算，因为它使用 ProcessPoolExecutor 类把工作分配给多个 Python 进程处理。因此，如果需要做 CPU 密集型处理，使用这个模块能绕开 GIL，利用所有可用的 CPU 核心。
```
def download_many(cc_list):
    workers = min(MAX_WORKERS, len(cc_list))
    with futures.ThreadPoolExecutor(workers) as executor:
```
改成:
```
def download_many(cc_list):
    with futures.ProcessPoolExecutor() as executor:
```
`ThreadPool Executor.__init__` 方需要 `max_workers` 参数， 指定线程池中线程的数量。 在`ProcessPoolExecutor` 类中，那个参数是可选的，而且大多数情况下不使用——默认值是 `os.cpu_count()` 函数返回的 CPU 数量。

如果使用 Python 处理 CPU 密集型工作，应该试试 PyPy(http://pypy.org)。

## 17.4 实验 Executor.map 方法
```
executor = futures.ThreadPoolExecutor(max_workers=3)
results = executor.map(loiter, range(5))  # 返回的是一个生成器
for i, result in enumerate(results):
...
```
for 循环中的 enumerate 函数会隐式调用 next(results)，这个函数又会在(内部)表示第一个任务(loiter(0))的 _f future 上调用 _f.result() 方法。result 方法会阻塞，直到 future 运行结束，因此这个循环每次迭代时都要等待下一个结果做好准备。

Executor.map 函数返回结果的顺序与调用开始的顺序一致。

`executor.submit` 和 `futures.as_completed` 这个组合比 `executor.map` 更灵活， 因为 `submit` 方法能处理不同的可调用对象和参数，而 `executor.map` 只能处理参数不同的同一个可调用对象。此外，传给 `futures.as_completed` 函数的 `future` 集合可以来自多个 `Executor` 实例，例如一些由 `ThreadPoolExecutor` 实例创建，另一些由 `ProcessPoolExecutor` 实例创建。

## 17.5 显示下载进度并处理错误

### 17.5.1 flags2系列示例处理错误的方式

### 17.5.2 使用 futures.as_completed 函数
```
with futures.ThreadPoolExecutor(max_workers=concur_req) as executor:
    to_do_map = {}
    for cc in sorted(cc_list):
        future = executor.submit(download_one, cc, base_url, verbose)
        to_do_map[future] = cc
    done_iter = futures.as_completed(to_do_map)
    for future in done_iter:
        try:
            res = future.result()
        except requests.exceptions.HTTPError as exc:
            error_msg = 'HTTP {res.status_code} - {res.reason}'
            error_msg = error_msg.format(res=exc.response)
        except requests.exceptions.ConnectionError as exc:
            error_msg = 'Connection error'
        else:
            error_msg = ''
            status = res.status
```
futures.as_completed 函数特别有用的惯用法：构建一个字典， 把各个 future 映射到其他数据(future 运行结束后可能有用)上。这里，在 to_do_map 中， 我们把各个 future 映射到对应的国家代码上。这样，尽管 future 生成的结果顺序已经乱了，依然便于使用结果做后续处理。

### 17.5.3 线程和多进程的替代方案
multiprocessing 模块 还能解决协作进程遇到的最大挑战：在进程之间传递数据。
