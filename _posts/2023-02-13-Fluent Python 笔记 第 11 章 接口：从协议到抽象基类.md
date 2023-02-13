---
title: Fluent Python 笔记 第 11 章 接口：从协议到抽象基类
tags: Python
pageview: true
---

本章讨论的话题是接口:从鸭子类型的代表特征动态协议，到使接口更明确、能验证实现是否符合规定的抽象基类(Abstract Base Class，ABC)。


## 11.1 Python 文化中的接口和协议
对 Python 程序员来说，“X 类对象”“X 协 议”和“X 接口”都是一个意思。

## 11.2 Python 喜欢序列
Python 数据模型的哲学是尽量支持基本协议。

为了迭代对象，解释器会尝试调用两个不同的方法。

## 11.3 使用猴子补丁在运行时实现协议
`shuffle` 函数要调换集合中元素的位置，而 FrenchDeck 只实现了不可变的序列协议。可变的序列还必须提供 `__setitem__` 方法。
```
>>> def set_card(deck, position, card):
...     deck._cards[position] = card
...
>>> FrenchDeck.__setitem__ = set_card
>>> shuffle(deck) 
```
这种技术叫猴子补丁：在运行时修改类或模块，而不改动源码。

## 11.4 Alex Martelli的水禽
基本上不需要自己编写新的抽象基类，只要正确使用现有的抽象基类，就能获得 99.9% 的好处，而不用冒着设计不当导致的巨大风险。

## 11.5 定义抽象基类的子类
## 11.6 标准库中的抽象基类
### 11.6.1 collections.abc 模块中的抽象基类
#### Iterable、Container 和 Sized
各个集合应该继承这三个抽象基类，或者至少实现兼容的协议。Iterable 通过 `__iter__` 方法支持迭代，Container 通过 `__contains__` 方法支持 in 运算符，Sized 通过 `__len__` 方法支持 `len()` 函数。
#### Sequence、Mapping 和 Set
这三个是主要的不可变集合类型，而且各自都有可变的子类。
#### MappingView
映射方法 `.items()`、`.keys()` 和 `.values()` 返回的对象分别是 `ItemsView`、 `KeysView` 和 `ValuesView` 的实例。前两个类还从 `Set` 类继承了丰富的接口。
#### Callable 和 Hashable
这两个抽象基类与集合没有太大的关系，只不过因为 `collections.abc` 是标准库中定义 抽象基类的第一个模块，而它们又太重要了，因此才把它们放到 `collections.abc` 模块 中。我从未见过 `Callable` 或 `Hashable` 的子类。这两个抽象基类的主要作用是为内置函 数 `isinstance` 提供支持，以一种安全的方式判断对象能不能调用或散列。·
#### Iterator
注意它是 Iterable 的子类。

### 11.6.2 抽象基类的数字塔
- Number
- Complex
- Real
- Rational
- Integral

## 11.7 定义并使用一个抽象基类
```
import abc

class Tombola(abc.ABC):

@abc.abstractmethod
def load(self, iterable):
"""从可迭代对象中添加元素。"""

@abc.abstractmethod
def pick(self):
"""随机删除元素，然后将其返回。 如果实例为空，这个方法应该抛出`LookupError`。
"""

def loaded(self):
"""如果至少有一个元素，返回`True`，否则返回`False`。"""
    return bool(self.inspect())

def inspect(self):
"""返回一个有序元组，由当前元素构成。"""
    items = []
    while True:
        try:
            items.append(self.pick())
        except LookupError:
            break
    self.load(items)
    return tuple(sorted(items))
```
- 自己定义的抽象基类要继承 abc.ABC。
- 抽象方法使用 @abstractmethod 装饰器标记，而且定义体中通常只有文档字符串。
- 抽象基类可以包含具体方法抽象基类中的具体方法只能依赖抽象基类定义的接口(即只能使用抽象基类中的其他具体方法、抽象方法或特性)。
- 即便实现了，子类也必须覆盖抽象方法，但是在子类中可以使用 `super()` 函数调用抽象方法，为它添加功能，而不是从头开始实现。

### 11.7.1 抽象基类句法详解
### 11.7.2 定义Tombola抽象基类的子类
### 11.7.3 Tombola的虚拟子类
```
@Tombola.register
class TomboList(list):
```
注册之后，可以使用 issubclass 和 isinstance 函数判断 TomboList 是不是 Tombola 的子类。Tombolist 没有从 Tombola 中继承任何方法。

## 11.8 Tombola子类的测试方法

## 11.9 Python使用register的方式
```
Sequence.register(tuple)
Sequence.register(str)
Sequence.register(range)
Sequence.register(memoryview)
```

## 11.10 鹅的行为有可能像鸭子
