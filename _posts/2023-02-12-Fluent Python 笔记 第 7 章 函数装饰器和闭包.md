---
title: Fluent Python 笔记 第 7 章 函数装饰器和闭包.md
tags: Python
pageview: true
---

函数装饰器用于在源码中“标记”函数，以某种方式增强函数的行为。这是一项强大的功能，但是若想掌握，必须理解闭包。

nonlocal 是新近出现的保留关键字，在 Python 3.0 中引入。作为 Python 程序员，如果严格 遵守基于类的面向对象编程方式，即便不知道这个关键字也不会受到影响。然而，如果你想自己实现函数装饰器，那就必须了解闭包的方方面面，因此也就需要知道 nonlocal。

除了在装饰器中有用处之外，闭包还是回调式异步编程和函数式编程风格的基础。

本章的最终目标是解释清楚函数装饰器的工作原理，包括最简单的注册装饰器和较复杂的参数化装饰器。但是，在实现这一目标之前，我们要讨论下述话题:
- Python 如何计算装饰器句法
- Python 如何判断变量是不是局部的
- 闭包存在的原因和工作原理
- nonlocal 能解决什么问题

掌握这些基础知识后，我们可以进一步探讨装饰器:
- 实现行为良好的装饰器
- 标准库中有用的装饰器
- 实现一个参数化装饰器

若想真正理解装饰器，需要区分导入时和运行时，还要知道变量作用域、闭包和新增的 `nonlocal` 声明。

## 7.1 装饰器基础知识
装饰器是可调用的对象，其参数是另一个函数(被装饰的函数)。装饰器可能会处理被装饰的函数，然后把它返回，或者将其替换成另一个函数或可调用对象。
```
# 假如有个名为 decorate 的装饰器:
@decorate
def target():
    print('running target()')
# 上述代码的效果与下述写法一样:
def target():
    print('running target()')
target = decorate(target)
```
两种写法的最终结果一样:上述两个代码片段执行完毕后得到的 target 不一定是原来那个target 函数，而是 decorate(target) 返回的函数。

装饰器的一大特性是，能把被装饰的函数替换成其他函数。第二个特性是，装饰器在加载模块时立即执行。

## 7.2 Python何时执行装饰器
装饰器的一个关键特性是，它们在被装饰的函数定义之后立即运行。这通常是在导入时(即 Python 加载模块时)。

主要想强调，函数装饰器在导入模块时立即执行，而被装饰的函数只在明确调用时运行。这突出了 Python 程序员所说的导入时和运行时之间的区别。

- 装饰器函数与被装饰的函数在同一个模块中定义。实际情况是，装饰器通常在一个模块中定义，然后应用到其他模块中的函数上。
- register 装饰器返回的函数与通过参数传入的相同。实际上，大多数装饰器会在内部定义一个函数，然后将其返回。

## 7.3 使用装饰器改进“策略”模式
```
promos = []
def promotion(promo_func):
    promos.append(promo_func)
    return promo_func
@promotion
def fidelity(order):
"""为积分为1000或以上的顾客提供5%折扣"""
    return order.total() * .05 if order.customer.fidelity >= 1000 else 0

...
```
优点：
- 促销策略函数无需使用特殊的名称(即不用以 `_promo` 结尾)。
- `@promotion` 装饰器突出了被装饰的函数的作用，还便于临时禁用某个促销策略:只需把
装饰器注释掉。
- 促销折扣策略可以在其他模块中定义，在系统中的任何地方都行，只要使用 `@promotion` 装饰即可。

## 7.4 变量作用域规则
Python 编译函数的定义体时，它先判断 `b` 是局部变量还是全局变量，如果在函数中声明了，那必须在声明之后用。否则要先用 `global b` 声明，那就是用全局变量 `b`。

## 7.5 闭包
闭包指延伸了作用域的函数，其中包含函数定义体中引用、但是不在定义体中定义的非全局变量。函数是不是匿名的没有关系，关键是它能访问定义体之外定义的非全局变量。
```
def make_averager():
    series = []
    def averager(new_value):
        series.append(new_value)
        total = sum(series)
        return total/len(series)
    return averager
```
调用 make_averager 时，返回一个 averager 函数对象。每次调用 averager 时，它会把参数添加到系列值中，然后计算当前平均值。

`series` 是 `make_averager` 函数的局部变量，因为那个函数的定义体中初始化了 `series:series = []`。可是，调用 `avg(10)` 时，`make_averager` 函数已经返回了，而它的本地作用域也一去不复返了。
在 `averager` 函数中，`series` 是自由变量(free variable)。这是一个技术术语，指未在本地作用域中绑定的变量。
![5](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/5.png)

series 的绑定在返回的 avg 函数的 `__closure__` 属性中。`avg.__closure__` 中的各个元 素对应于 `avg.__code__.co_freevars` 中的一个名称。这些元素是 cell 对象，有个 `cell_ contents` 属性，保存着真正的值。
```
>>> avg.__code__.co_freevars
('series',)
>>> avg.__closure__
(<cell at 0x107a44f78: list object at 0x107a91a48>,)
>>> avg.__closure__[0].cell_contents
[10, 11, 12]
```

## 7.6 nonlocal声明
改进一下：
```
def make_averager():
    count = 0
    total = 0

    def averager(new_value):
        count += 1
        total += new_value
        return total / count
    return averager
```
不过这个运行的时候会出错：
`UnboundLocalError: local variable 'count' referenced before assignment`

问题是，当count是数字或任何不可变类型时，`count += 1`语句的作用其实与`count = count + 1`一样。因此，我们在`averager`的定义体中为`count`赋值了，这会把`count`变成局部变量。`total` 变量也受这个问题影响。

前面没遇到这个问题，因为我们没有给 `series` 赋值，我们只是调用 `series.append`， 并把它传给 `sum` 和 `len`。也就是说，我们利用了列表是可变的对象这一事实。

但是对数字、字符串、元组等不可变类型来说，只能读取，不能更新。如果尝试重新绑定，例如`count = count + 1`，其实会隐式创建局部变量`count`。这样，`count`就不是自由变量了，因此不会保存在闭包中。

为了解决这个问题，Python 3 引入了 `nonlocal` 声明。它的作用是把变量标记为自由变量，即使在函数中为变量赋予新值了，也会变成自由变量。如果为 `nonlocal` 声明的变量赋予新值，闭包中保存的绑定会更新。
`nonlocal count, total`

## 7.7 实现一个简单的装饰器
```
import time

def clock(func):
    def clocked(*args):
        t0 = time.perf_counter()
        result = func(*args)
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_str = ', '.join(repr(arg) for arg in args)
        print('[%0.8fs] %s(%s) -> %r' % (elapsed, name, arg_str, result))
        return result
    return clocked
```

实现层面，Python 装饰器与《设计模式:可复用面向对象软件的基础》中所述的“装饰器”没有多少相似之处。

上面实现的 `clock` 装饰器有几个缺点:不支持关键字参数，而且遮盖了被装饰函数 的 `__name__` 和 `__doc__` 属性。下面使用 `functools.wraps` 装饰器把相关的属性从 `func` 复制到 `clocked` 中。此外，这个新版还能正确处理关键字参数。

```
import time
import functools

def clock(func):
    @functools.wraps(func)
    def clocked(*args, **kwargs):
        t0 = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - t0
        name = func.__name__
        arg_lst = []
        if args:
            arg_lst.append(', '.join(repr(arg) for arg in args))
        if kwargs:
            pairs = ['%s=%r' % (k, w) for k, w in sorted(kwargs.items())]
            arg_lst.append(', '.join(pairs))
        arg_str = ', '.join(arg_lst)
        print('[%0.8fs] %s(%s) -> %r ' % (elapsed, name, arg_str, result))
        return result
    return clocked
```
functools.wraps 只是标准库中拿来即用的装饰器之一。

## 7.8 标准库中的装饰器
### 7.8.1 使用functools.lru_cache做备忘
（笔记作者：可以拿来刷题的时候加速？）
Least Recently Used, 把耗时的函数的结果保存起来，避免传入相同的参数时重复计算
```
@functools.lru_cache(maxsize=128, typed=False)
```
`maxsize` 参数指定存储多少个调用的结果。缓存满了之后，旧的结果会被扔掉，腾出空间。 为了得到最佳性能，`maxsize` 应该设为 2 的幂。`typed` 参数如果设为 `True`，把不同参数类型得到的结果分开保存，即把通常认为相等的浮点数和整数参数(如 1 和 1.0)区分开。顺便说一下，因为 `lru_cache` 使用字典存储结果，而且键根据调用时传入的定位参数和关键字参数创建，所以被 `lru_cache` 装饰的函数，它的所有参数都必须是可散列的。

### 7.8.2 单分派泛函数

像重载。

使用 `@singledispatch` 装饰的普通函数会变成 泛函数(generic function):根据第一个参数的类型，以不同方式执行相同操作的一组函数。
```
from functools import singledispatch
from collections import abc
import numbers
import html

@singledispatch
def htmlize(obj):
    content = html.escape(repr(obj))
    return '<pre>{}</pre>'.format(content)

@htmlize.register(str)
def _(text):
    content = html.escape(text).replace('\n', '<br>\n')
    return '<p>{0}</p>'.format(content)

@htmlize.register(numbers.Integral)
def _(n):
    return '<pre>{0} (0x{0:x})</pre>'.format(n)

@htmlize.register(tuple)
@htmlize.register(abc.MutableSequence)
def _(seq):
    inner = '</li>\n<li>'.join(htmlize(item) for item in seq)
    return '<ul>\n<li>' + inner + '</li>\n</ul>'
```
只要可能，注册的专门函数应该处理抽象基类(如 `numbers.Integral` 和 `abc.MutableSequence`)，不要处理具体实现(如 int 和 list)。这样，代码支持的兼容类型更广泛。


## 7.9 叠放装饰器
```
@d1
@d2
def f():
    print('f')
# 等同于
def f():
    print('f')
f = d1(d2(f))
```

## 7.10 参数化装饰器
简单的装饰器是没有调用符号（）的，要传参的话：创建一个装饰器工厂函数，把参数传给它，返回一个装饰器。
### 7.10.1 一个参数化的注册装饰器
```
def register(active=True):
    def decorate(func):
        print('running register(active=%s)->decorate(%s)' % (active, func))
        if active:
            registry.add(func)
        else:
            registry.discard(func)
        return func
    return decorate

@register(active=False)
def f1():
    print('running f1()')
```
这里的关键是，`register()` 要返回 `decorate`，然后把它应用到被装饰的函数上。

### 7.10.2 参数化clock装饰器
很简单就不解释了。

Graham Dumpleton 和 Lennart Regebro(本书的技术审校之一)认为，装饰器 最好通过实现 `__call__` 方法的类实现，不应该像本章的示例那样通过函数实现。我同意使用他们建议的方式实现非平凡的装饰器更好，但是使用函数解说这个语言特性的基本思想更易于理解。
```
def clock(fmt=DEFAULT_FMT):
    def decorate(func):
        def clocked(*_args):
            t0 = time.time()
            _result = func(*_args)
            elapsed = time.time() - t0
            name = func.__name__
            args = ', '.join(repr(arg) for arg in _args)
            result = repr(_result)
            print(fmt.format(**locals()))
            return _result
        return clocked
    return decorate
```
