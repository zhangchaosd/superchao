---
title: Fluent Python 笔记 第 9 章 符合 Python 风格的对象
tags: Python
pageview: true
---

得益于 Python 数据模型，自定义类型的行为可以像内置类型那样自然。实现如此自然的行为，靠的不是继承，而是鸭子类型(duck typing)：我们只需按照预定行为实现对象所需的方法即可。


## 9.1 对象表示形式
实现 `__repr__` 和 `__str__` 特殊方法，为 `repr()` 和 `str()` 提供支持。

## 9.2 再谈向量类
```
from array import array
import math

class Vector2d:
    typecode = 'd'  # typecode 是类属性，在 Vector2d 实例和字节序列之间转换时使用。

    def __init__(self, x, y):
        self.x = float(x)  # 把 x 和 y 转换成浮点数，尽早捕获错误
        self.y = float(y)

    def __iter__(self):  #把 Vector2d 实例变成可迭代的对象，这样才能拆包  也可以写成yield self.x; yield.self.y
        return (i for i in (self.x, self.y))

    def __repr__(self):  # 使用 {!r} 获取各个分量的表示形式，然后插值，构成一个字符串；因为Vector2d 实例是可迭代的对象，所以 *self 会把 x 和 y 分量提供给 format 函数
        class_name = type(self).__name__
        return '{}({!r}, {!r})'.format(class_name, *self)

    def __str__(self):
        return str(tuple(self))

    def __bytes__(self):
        return (bytes([ord(self.typecode)]) + bytes(array(self.typecode, self)))

    def __eq__(self, other):  # 拿 Vector2d 实例与其他具有相同数值的可迭代对象相比，结果也是 True (如Vector(3, 4) == [3, 4])。这个行为可以视作特性，也可以视作缺陷。
        return tuple(self) == tuple(other)

    def __abs__(self):
        return math.hypot(self.x, self.y)

    def __bool__(self):
        return bool(abs(self))
```

## 9.3 备选构造方法
```
@classmethod
def frombytes(cls, octets):
    typecode = chr(octets[0])
    memv = memoryview(octets[1:]).cast(typecode)
    return cls(*memv)
```

## 9.4 classmethod 与 staticmethod
classmethod

前面展示了它的用法：定义操作类，而不是操作实例的方法。 classmethod 改变了调用方法的方式，因此类方法的第一个参数是类本身，而不是实例。 classmethod 最常见的用途是定义备选构造方法，按照约定，类方法的第一个参数名为 cls(但是 Python 不介意具体怎么命名)。

staticmethod

装饰器也会改变方法的调用方式，但是第一个参数不是特殊的值。其实，静态方法就是普通的函数，只是碰巧在类的定义体中，而不是在模块层定义。

## 9.5 格式化显示


```
>>> format(42, 'b')  # 'x'是十六进制
'101010'
>>> format(2/3, '.1%')
'66.7%'
```

```
# 在Vector2d类中定义
def __format__(self, fmt_spec=''):
    components = (format(c, fmt_spec) for c in self) #
    return '({}, {})'.format(*components) #

>>> format(v1, '.2f')
'(3.00, 4.00)'
>>> format(v1, '.3e')
'(3.000e+00, 4.000e+00)'
```
```
def __format__(self, fmt_spec=''): 
    if fmt_spec.endswith('p'):
        fmt_spec = fmt_spec[:-1]
        coords = (abs(self), self.angle())
        outer_fmt = '<{}, {}>'
    else:
        coords = self
        outer_fmt = '({}, {})'
    components = (format(c, fmt_spec) for c in coords)
    return outer_fmt.format(*components)

>>> format(Vector2d(1, 1), 'p')
'<1.4142135623730951, 0.7853981633974483>'
>>> format(Vector2d(1, 1), '.3ep')
'<1.414e+00, 7.854e-01>'
>>> format(Vector2d(1, 1), '0.5fp')
'<1.41421, 0.78540>'
```

## 9.6 可散列的 Vector2d
把 x 和 y 分量设为只读特性

```
class Vector2d:
    typecode = 'd'

    def __init__(self, x, y):
        self.__x = float(x)  # 两个前导下划线(尾部没有下划线，或者有一个下划线)，把属性标记为私有的，但不符合建议
        self.__y = float(y)

    @property  # @property 装饰器把读值方法标记为特性。
    def x(self):
        return self.__x

    @property
    def y(self):
        return self.__y

    def __iter__(self):
        return (i for i in (self.x, self.y))
```
最好使用位运算符异或(^)混合各分量的散列值。
```
    def __hash__(self):
        return hash(self.x) ^ hash(self.y)
```

## 9.7 Python 的私有属性和“受保护的”属性

有人编写了一个名为 Dog 的类，这个类的内部用到了 `mood` 实例属性，但是没有 将其开放。现在，你创建了 `Dog` 类的子类:`Beagle`。如果你在毫不知情的情况下又创建了名为 `mood` 的实例属性，那么在继承的方法中就会把 `Dog` 类的 `mood` 属性覆盖掉。这是个难以调试的问题。为了避免这种情况，如果以 `__mood` 的形式(两个前导下划线，尾部没有或最多有一个下划线)命名实例属性，Python 会把属性名存入实例的 `__dict__` 属性中，而且会在前面加上一个下划线和类名。因此，对 `Dog` 类来说，`__mood` 会变成 `_Dog__mood`;对 Beagle 类来说，会变成 `_Beagle__mood`。这个语言特性叫名称改写(name mangling)。

不过，只要知道改写私有属性名的机制，任何人都能直接读取私有属性。

## 9.8 使用 __slots__ 类属性节省空间

让解释器在元组中存储实例属性，而不用字典。在类中定义 `__slots__` 属性之后，实例不能再有 `__slots__` 中所列名称之外的其他属性。是用于优化的，不是为了约束程序员。
```
class Vector2d:
    __slots__ = ('__x', '__y')
```
用户定义的类中默认就有 `__weakref__` 属性。可是，如 果类中定义了 `__slots__` 属性，而且想把实例作为弱引用的目标，那么要把 '`__weakref__`'
添加到 `__slots__` 中。

### __slots__ 的问题
- 每个子类都要定义 `__slots__` 属性，因为解释器会忽略继承的 `__slots__` 属性。
- 实例只能拥有 `__slots__` 中列出的属性，除非把 '`__dict__`' 加入 `__slots__` 中(这样做就失去了节省内存的功效)。
- 如果不把 '`__weakref__`' 加入 `__slots__`，实例就不能作为弱引用的目标。

与其他优化措施一样，仅当权衡当下的需求并仔细搜集资料后证明确实有必要时，才应该使用 `__slots__` 属性。

## 9.9 覆盖类属性
如果为不存在的实例属性赋值，会新建实例属性。假如我们为 typecode 实例属性赋值，那么同名类属性不受影响。然而，自此之后，实例读取的 self.typecode 是实例属性 typecode，也就是把同名类属性遮盖了。借助这一特性，可以为各个实例的 typecode 属性定制不同的值。

类属性是公开的，因此会被子类继承，于是经常会创建一个子类，只用于定制类的数据属性。
