---
title: Fluent Python 笔记 第 15 章 上下文管理器和 else 块
tags: Python
pageview: true
---

## 15.1 先做这个，再做那个:if语句之外的else块

else 子句不仅能在 if 语句中使用，还能在 for、while 和 try 语句中使用。

### for
仅当 for 循环运行完毕时(即 for 循环没有被 break 语句中止)才运行 else 块。

### while
仅当 while 循环因为条件为假值而退出时(即 while 循环没有被 break 语句中止)才运行 else 块。

### try
仅当 try 块中没有异常抛出时才运行 else 块。官方文档(https://docs.python.org/3/ reference/compound_stmts.html)还指出:“else 子句抛出的异常不会由前面的 except 子句处理。”

## 15.2 上下文管理器和with块
与函数和模块不同，with 块没有定义新的作用域。
```
class LookingGlass:
    def __enter__(self):
        import sys
        self.original_write = sys.stdout.write
        sys.stdout.write = self.reverse_write
        return 'JABBERWOCKY'  # 存到 as 后面

    def reverse_write(self, text):
        self.original_write(text[::-1])

    def __exit__(self, exc_type, exc_value, traceback):
        import sys
        sys.stdout.write = self.original_write
        if exc_type is ZeroDivisionError:
            print('Please DO NOT divide by zero!')
            return True
```

## 15.3 contextlib 模块中的实用工具


closing

如果对象提供了 close() 方法，但没有实现 `__enter__`/`__exit__` 协议，那么可以使用这个函数构建上下文管理器。

suppress

构建临时忽略指定异常的上下文管理器。

@contextmanager

这个装饰器把简单的生成器函数变成上下文管理器，这样就不用创建类去实现管理器协议了。

ContextDecorator

这是个基类，用于定义基于类的上下文管理器。这种上下文管理器也能用于装饰函数，在受管理的上下文中运行整个函数。

ExitStack

这个上下文管理器能进入多个上下文管理器。with 块结束时，ExitStack 按照后进先出的 顺序调用栈中各个上下文管理器的 `__exit__` 方法。如果事先不知道 with 块要进入多少个上下文管理器，可以使用这个类。例如，同时打开任意一个文件列表中的所有文件。

## 15.4 使用 @contextmanager
`@contextmanager` 装饰器能减少创建上下文管理器的样板代码量，因为不用编写一个完整的 类，定义 `__enter__` 和 `__exit__` 方法，而只需实现有一个 yield 语句的生成器，生成想让 `__enter__` 方法返回的值。

yield 语句的作用是把函数的定义体分成两部分: `yield` 语句前面的所有代码在 `with` 块开始时(即解释器调用 `__enter__` 方法时)执行， `yield` 语句后面的代码在 `with` 块结束时(即调用 `__exit__` 方法时)执行。

```
@contextlib.contextmanager
def looking_glass():
    import sys
    original_write = sys.stdout.write

    def reverse_write(text):
        original_write(text[::-1])

    sys.stdout.write = reverse_write
    msg = ''
    try:
        yield 'JABBERWOCKY'
    except ZeroDivisionError:
        msg = 'Please DO NOT divide by zero!'
    finally:
        sys.stdout.write = original_write
        if msg:
            print(msg)
```
contextlib.contextmanager 装饰器会把函数包装成实现 `__enter__` 和 `__exit__` 方法的类。

这个类的 `__enter__` 方法有如下作用。

(1) 调用生成器函数，保存生成器对象(这里把它称为 gen)。

(2) 调用 `next(gen)`，执行到 `yield` 关键字所在的位置。

(3) 返回 `next(gen)` 产出的值，以便把产出的值绑定到 `with/as` 语句中的目标变量上。

`with` 块终止时，`__exit__` 方法会做以下几件事。

(1) 检查有没有把异常传给 `exc_type`;如果有，调用 `gen.throw(exception)`，在生成器函数定义体中包含 `yield` 关键字的那一行抛出异常。

(2) 否则，调用 `next(gen)`，继续执行生成器函数定义体中 yield 语句之后的代码。
