---
title: Fluent Python 笔记 第 16 章 协程
tags: Python
pageview: true
---

从句法上看，协程与生成器类似，都是定义体中包含 yield 关键字的函数。可是，在协程中，yield通常出现在表达式的右边(例如，datum = yield)，可以产出值，也可以不产出——如果 yield 关键字后面没有表达式，那么生成器产出 None。协程可能会从调用方接收数据，不过调用方把数据提供给协程使用的是 .send(datum) 方法，而不是 next(...) 函数。通常，调用方会把值推送给协程。

yield 关键字甚至还可以不接收或传出数据。不管数据如何流动，yield 都是一种流程控制 工具，使用它可以实现协作式多任务:协程可以把控制器让步给中心调度程序，从而激活 其他的协程。

从根本上把 yield 视作控制流程的方式，这样就好理解协程了。

## 16.1 生成器如何进化成协程
## 16.2 用作协程的生成器的基本行为

```
def simple_coroutine():
    print('-> coroutine started')
    x = yield
    print('-> coroutine received:', x)

>>> my_coro = simple_coroutine()
>>> my_coro
<generator object simple_coroutine at 0x100c2be10>
>>> next(my_coro)
-> coroutine started
>>> my_coro.send(42)
-> coroutine received: 42
Traceback (most recent call last):
       ...
     StopIteration
```
与创建生成器的方式一样，调用函数得到生成器对象。

四个状态：

'GEN_CREATED'

等待开始执行。

'GEN_RUNNING'

解释器正在执行。只有在多线程应用中才能看到这个状态。此外，生成器对象在自己身上调用 getgeneratorstate 函数也行，不过这样做没什么用。

'GEN_SUSPENDED'

在 yield 表达式处暂停。

'GEN_CLOSED'

执行结束。

始终要调用 next(my_coro) 激活协程——也可以调用 my_coro.send(None)，效果一样。

## 16.3 示例：使用协程计算移动平均值
```
def averager():
    total = 0.0
    count = 0 
    average = None 
    while True:
        term = yield average
        total += term
        count += 1
        average = total/count

>>> coro_avg = averager()
>>> next(coro_avg)
>>> coro_avg.send(10)
10.0
>>> coro_avg.send(30)
20.0
>>> coro_avg.send(5)
15.0

```
先输出再等待输入。

## 16.4 预激协程的装饰器

```
from functools import wraps

def coroutine(func):
"""装饰器:向前执行到第一个`yield`表达式，预激`func`"""
    @wraps(func)
    def primer(*args,**kwargs):
        gen = func(*args,**kwargs)
        next(gen)
        return gen
    return primer
```
使用`yield from`句法(参见16.7节)调用协程时，会自动预激。

## 16.5 终止协程和异常处理
```
exc_coro.close()
```
```
exc_coro.throw(DemoException)
```
```
class DemoException(Exception):
"""为这次演示定义的异常类型。"""
def demo_finally():
    print('-> coroutine started')
    try:
        while True:
            try:
                x = yield
            except DemoException:
                print('*** DemoException handled. Continuing...')
            else:
                print('-> coroutine received: {!r}'.format(x))
    finally:
        print('-> coroutine ending')
```

## 16.6 让协程返回值

```
...
return Result(count, average)


try:
    coro_avg.send(None)
except StopIteration as exc:
    result = exc.value
```

## 16.7 使用 yield from
`yield from x`表达式对x对象所做的第一件事是，调用`iter(x)`，从中获取迭代器。

`yield from` 的主要功能是打开双向通道，把最外层的调用方与最内层的子生成器连接起来。

委派生成器

包含 `yield from <iterable>` 表达式的生成器函数。

子生成器

从 `yield from` 表达式中 `<iterable>` 部分获取的生成器。

调用方

指代调用委派生成器的客户端代码。

示例用法：

```
# 子生成器
def averager():
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield
        if term is None:
            break
        total += term
        count += 1
        average = total/count
    return Result(count, average)

# 委派生成器
def grouper(results, key):
    while True:
        results[key] = yield from averager()

# 调用方
def main(data):
    results = {}
    for key, values in data.items():
        group = grouper(results, key)
        next(group)
        for value in values:
            group.send(value)
        group.send(None) # 重要!
    # print(results) # 如果要调试，去掉注释
    report(results)
```
如果子生成器不终止，委派生成器会在 `yield from` 表达式处永远暂停。

## 16.8 yield from 的意义

## 16.9 使用案例:使用协程做离散事件仿真
