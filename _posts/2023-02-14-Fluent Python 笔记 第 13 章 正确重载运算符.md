---
title: Fluent Python 笔记 第 13 章 正确重载运算符
tags: Python
pageview: true
---

运算符重载的作用是让用户定义的对象使用中缀运算符(如 + 和 |)或一元运算符(如 - 和 ~)。说得宽泛一些，在 Python 中，函数调用(())、属性访问(.)和元素访问 / 切片 ([])也是运算符，不过本章只讨论一元运算符和中缀运算符。

## 13.1 运算符重载基础
Python 中的限制：
- 不能重载内置类型的运算符
- 不能新建运算符，只能重载现有的
- 某些运算符不能重载：is、and、or 和 not(不过位运算符 &、| 和 ~ 可以)

## 13.2 一元运算符
要遵守运算符的一个基本规则：始终返回一个 新对象。也就是说，不能修改 self，要创建并返回合适类型的新实例。

### -(__neg__)
一元取负算术运算符。如果 x 是 -2，那么 -x == 2。

### +(__pos__)
一元取正算术运算符。通常，x == +x，但也有一些例外。如果好奇，请阅读“x和+x何时不相等”附注栏。

### ~(__invert__)
对整数按位取反，定义为 ~x == -(x+1)。如果 x 是 2，那么 ~x == -3。

### x 和 +x 何时不相等
第一例与 decimal.Decimal 类有关。如果 x 是 Decimal 实例，在算术运算的上下文中创建，然后在不同的上下文中计算 +x。
```
ctx = decimal.getcontext()
ctx.prec = 40
one_third = decimal.Decimal('1') / decimal.Decimal('3')
ctx.prec = 28
one_third == +one_third  # False
```

第二例在`collections.Counter`的文档中，Counter 相加时，负值和零值计数会从结果中剔除。而一元运算符 + 等同于加上一个空 Counter，因此它产生一个新的 Counter 且仅保留大于零的计数器。
```
ct['r'] = -3
ct['d'] = 0
>>> ct
Counter({'a': 5, 'b': 2, 'c': 1, 'd': 0, 'r': -3})
>>> +ct
Counter({'a': 5, 'b': 2, 'c': 1})
```

## 13.3 重载向量加法运算符+
```
def __add__(self, other):  # 用下面的版本
    pairs = itertools.zip_longest(self, other, fillvalue=0.0)
    return Vector(a + b for a, b in pairs)  # 返回一个新 Vector 实例

v1 + (10, 20, 30)
```
a+b：

先 `a.__add__(b)` 再尝试 `b.__radd__(a)`
```
def __radd__(self, other):
    return self + other
```
![0](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230214/0.png)

别把 NotImplemented 和 NotImplementedError 搞混了。前者是特殊的单例值， 如果中缀运算符特殊方法不能处理给定的操作数，那么要把它返回(return) 给解释器。而 NotImplementedError 是一种异常，抽象类中的占位方法把它抛出(raise)，提醒子类必须覆盖。

如果由于类型不兼容而导致运算符特殊方法无法返回有效的结果，那么应该返回 NotImplemented，而不是抛出 TypeError。返回 NotImplemented 时，另一个操作数所属的类型还有机会执行运算，即 Python 会尝试调用反向方法。
```
 def __add__(self, other):
    try:
        pairs = itertools.zip_longest(self, other, fillvalue=0.0)
        return Vector(a + b for a, b in pairs)
    except TypeError:
        return NotImplemented
```

## 13.4 重载标量乘法运算符*
```
def __mul__(self, scalar):
    if isinstance(scalar, numbers.Real):
        return Vector(n * scalar for n in self)
    else:
        return NotImplemented
def __rmul__(self, scalar):
    return self * scalar
```

![1](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230214/1.png)

![2](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230214/2.png)

## 13.5 众多比较运算符

![3](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230214/3.png)


```
def __eq__(self, other):
    if isinstance(other, Vector):
        return (len(self) == len(other) and all(a == b for a, b in zip(self, other)))
    else:
        return NotImplemented
```
那么 != 运算符呢?我们不用实现它，因为从 object 继承的 `__ne__` 方法的后备行为满足了 我们的需求:定义了 `__eq__` 方法，而且它不返回 NotImplemented，`__ne__` 会对 `__eq__` 返 回的结果取反。

## 13.6 增量赋值运算符
如果一个类没有实现表13-1列出的就地运算符，增量赋值运算符只是语法糖:a += b的 作用与 `a = a + b` 完全一样。对不可变类型来说，这是预期的行为，而且，如果定义了 `__add__` 方法的话，不用编写额外的代码，+= 就能使用。
然而，如果实现了就地运算符方法，例如 `__iadd__`，计算 a += b 的结果时会调用就地运算 符方法。这种运算符的名称表明，它们会就地修改左操作数，而不会创建新对象作为结果。

```
class AddableBingoCage(BingoCage):
    def __add__(self, other):
        if isinstance(other, Tombola):
            return AddableBingoCage(self.inspect() + other.inspect())
        else:
            return NotImplemented
    def __iadd__(self, other):
        if isinstance(other, Tombola):
            other_iterable = other.inspect()
        else:
            try:
                other_iterable = iter(other)
            except TypeError:
                self_cls = type(self).__name__
                msg = "right operand in += must be {!r} or an iterable"
                raise TypeError(msg.format(self_cls))
        self.load(other_iterable)
        return self  # 增量赋值特殊方法必须返回 self。
```

一般来说，如果中缀运算符的正向方法(如 `__mul__`)只处理与 self 属于同 一类型的操作数，那就无需实现对应的反向方法(如 `__rmul__`)，因为按照 定义，反向方法是为了处理类型不同的操作数。
























```
![6](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/6.png)
```
