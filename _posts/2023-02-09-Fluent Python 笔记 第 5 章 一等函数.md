---
title: Fluent Python 笔记 第 5 章 一等函数
tags: Python
pageview: true
---

在 Python 中，函数是一等对象。编程语言理论家把“一等对象”定义为满足下述条件的程 序实体:
- 在运行时创建
- 能赋值给变量或数据结构中的元素 • 能作为参数传给函数
- 能作为函数的返回结果

## 5.1 把函数视作对象
会用 `map`。

## 5.2 高阶函数
接受函数为参数，或者把函数作为结果返回的函数是高阶函数(higher-order function)。

### map、filter 和 reduce 的现代替代品
```
>>> list(map(fact, range(6)))
[1, 1, 2, 6, 24, 120]
>>> [fact(n) for n in range(6)]
[1, 1, 2, 6, 24, 120]
>>> list(map(factorial, filter(lambda n: n % 2, range(6))))
[1, 6, 120]
>>> [factorial(n) for n in range(6) if n % 2
[1, 6, 120]
```

all(iterable)

如果 iterable 的每个元素都是真值，返回 True;all([]) 返回 True。 

any(iterable)

只要 iterable 中有元素是真值，就返回 True;any([]) 返回 False。

## 5.3 匿名函数
lambda 函数的定义体中不能赋值，也不能使用 while 和 try 等Python 语句。

![0](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/0.png)

## 5.4 可调用对象
如果想判断对象能否调用，可以使用内置的 callable() 函数。

## 5.5 用户定义的可调用类型
实现 `__call__` 方法

## 5.6 函数内省
![1](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/1.png)

![2](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/2.png)

## 5.7 从定位参数到仅限关键字参数
看懂下面这个怎么传。
```
def tag(name, *content, cls=None, **attrs):
    print('name:', name)
    print('content:', content)
    print('cls:', cls)
    print('attrs:', attrs)
```

<!-- `name` 每次必传。后面的除了 `cls=` 之外全是 content，`**d` 会传给最后。如果字典中有 cls 则会绑到前一个。 -->

## 5.8 获取关于参数的信息
内省：好像是指调用函数之前就知道这个函数需要什么参数，有没有默认参数。

函数对象有个 `__defaults__` 属性，它的值是一个元组，里面保存着定位参数和关键字参 数的默认值。仅限关键字参数的默认值在 `__kwdefaults__` 属性中。然而，参数的名称在 `__code__` 属性中，它的值是一个 code 对象引用，自身也有很多属性。

参数名称在 `__code__.co_varnames` 中，不过里面还有函数定义体中创建的局部变量。因此，参数名称是前 N 个字符串，N 的值由 `__code__`.co_argcount 确定。顺便说一下，这里不包含前缀为 * 或 ** 的变长参数。参数的默认值只能通过它们在 `__defaults__` 元组中的位置确定，因此要从后向前扫描才能把参数 和默认值对应起来。在这个示例中 clip 函数有两个参数，text 和 max_len，其中一个有默认值，即 80，因此它必然属于最后一个参数，即 max_len。这有违常理。

所以使用 `inspect`。
```
>>> from clip import clip
>>> from inspect import signature
>>> sig = signature(clip)
>>> sig  # doctest: +ELLIPSIS
<inspect.Signature object at 0x...>
>>> str(sig)
'(text, max_len=80)'
>>> for name, param in sig.parameters.items():
...     print(param.kind, ':', name, '=', param.default)
...
POSITIONAL_OR_KEYWORD : text = <class 'inspect._empty'>
POSITIONAL_OR_KEYWORD : max_len = 80
```
kind 属性的值是 _ParameterKind 类中的 5 个值之一，列举如下。 POSITIONAL_OR_KEYWORD
可以通过定位参数和关键字参数传入的形参(多数 Python 函数的参数属于此类)。 VAR_POSITIONAL
定位参数元组。
VAR_KEYWORD
关键字参数字典。
KEYWORD_ONLY
仅限关键字参数(Python 3 新增)。 POSITIONAL_ONLY
仅限定位参数;目前，Python 声明函数的句法不支持，但是有些使用 C 语言实现且不 接受关键字参数的函数(如 divmod)支持。

inspect.Signature 对象有个 bind 方法，它可以把任意个参数绑定到签名中的形参上，所用的规则与实参到形参的匹配方式一样。框架可以使用这个方法在真正调用函数前验证参数，
```
>>> import inspect
>>> sig = inspect.signature(tag)
>>> my_tag = {'name': 'img', 'title': 'Sunset Boulevard', ... 'src': 'sunset.jpg', 'cls': 'framed'}
>>> bound_args = sig.bind(**my_tag)
>>> bound_args
<inspect.BoundArguments object at 0x...>
>>> for name, value in bound_args.arguments.items():
... print(name, '=', value)
...
name = img
cls = framed
attrs = {'title': 'Sunset Boulevard', 'src': 'sunset.jpg'}
```

## 5.9 函数注解
函数声明中的各个参数可以在 : 之后增加注解表达式。
```
def clip(text:str, max_len:'int > 0'=80) -> str:
"""在max_len前面或后面的第一个空格处截断文本
"""
```

## 5.10 支持函数式编程的包
### 5.10.1 operator模块
operator模块为多个算术运算符提供了对应的函数，从而避免编写lambda a, b: a*b 这种平凡的匿名函数。

```
# 阶乘
reduce(mul, range(1, n+1))
```

operator 模块中还有一类函数，能替代从序列中取出元素或读取对象属性的 lambda 表达式:因此，itemgetter 和 attrgetter 其实会自行构建函数。

`itemgetter(1)` 的 作用与 `lambda fields: fields[1]` 一样。如果把多个参数传给 itemgetter，它构建的函数会返回提取的值构成的元组。itemgetter 使用 [] 运算符，因此它不仅支持序列，还支持映射和任何实现 `__getitem__` 方法的类。

`attrgetter` 与 `itemgetter` 作用类似，它创建的函数根据名称提取对象的属性。如果把 多个属性名传给 `attrgetter`，它也会返回提取的值构成的元组。此外，如果参数名中包含 .(点号)`，attrgetter` 会深入嵌套对象，获取指定的属性。

下面是 operator 模块中定义的部分函数(省略了以 _ 开头的名称，因为它们基本上是实现细节):
```
>>> [name for name in dir(operator) if not name.startswith('_')]
['abs', 'add', 'and_', 'attrgetter', 'concat', 'contains','countOf', 'delitem', 'eq', 'floordiv', 'ge', 'getitem', 'gt',
'iadd', 'iand', 'iconcat', 'ifloordiv', 'ilshift', 'imod', 'imul',
'index', 'indexOf', 'inv', 'invert', 'ior', 'ipow', 'irshift',
'is_', 'is_not', 'isub', 'itemgetter', 'itruediv', 'ixor', 'le',
'length_hint', 'lshift', 'lt', 'methodcaller', 'mod', 'mul', 'ne',
'neg', 'not_', 'or_', 'pos', 'pow', 'rshift', 'setitem', 'sub',
'truediv', 'truth', 'xor']

```
以 i 开头、后面是另一个运算符的那些名称(如 iadd、iand 等)，对应的是增量赋值运算符(如 +=、&= 等)。如果第一个参数是可变的，那 么这些运算符函数会就地修改它；否则，作用与不带 i 的函数一样，直接返回运算结果。

#### methodcaller
还可以冻结部分参数。
```
>>> from operator import methodcaller
>>> s = 'The time has come'
>>> upcase = methodcaller('upper')
>>> upcase(s)
'THE TIME HAS COME'
>>> hiphenate = methodcaller('replace', ' ', '-')
>>> hiphenate(s)
'The-time-has-come'
```

### 5.10.2 使用functools.partial冻结参数

```
triple = partial(mul, 3)
triple(7)
21
```
```
triple.func # 可以访问原函数。
triple.keywords
```


