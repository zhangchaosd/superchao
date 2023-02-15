---
title: Fluent Python 笔记 第 18 章 动态属性和特性
tags: Python
pageview: true
---
在 Python 中，数据的属性和处理数据的方法统称属性(attribute)。其实，方法只是可调用的属性。

## 19.1 使用动态属性转换数据
```
from collections import abc

class FrozenJSON: """一个只读接口，使用属性表示法访问JSON类对象 """
    def __init__(self, mapping):
        self.__data = dict(mapping)

    def __getattr__(self, name):
        if hasattr(self.__data, name):
            return getattr(self.__data, name)
        else:
            return FrozenJSON.build(self.__data[name])

    @classmethod
    def build(cls, obj):
        if isinstance(obj, abc.Mapping):
            return cls(obj)
        elif isinstance(obj, abc.MutableSequence):
            return [cls.build(item) for item in obj]
        else:
            return obj
```

## 19.1.2 处理无效属性名
检查传给 `FrozenJSON.__init__` 方法的映射中是否有键的名称为关键字，如果有，那么在键名后加上 _
```
def __init__(self, mapping):
    self.__data = {}
    for key, value in mapping.items():
        if keyword.iskeyword(key):
            key += '_'
        self.__data[key] = value
```
str 类提供的 `s.isidentifier()` 方法能根据语言的语法判断 s 是否为有效的 Python 标识符。

### 19.1.3 使用`__new__`方法以灵活的方式创建对象
其实，用于构建实 例的是特殊方法 `__new__`:这是个类方法(使用特殊方式处理，因此不必使用 `@classmethod` 装饰器)，必须返回一个实例。返回的实例会作为第一个参数(即 self)传给 `__init__` 方 法。因为调用 `__init__` 方法时要传入实例，而且禁止返回任何值，所以 `__init__` 方法其实是“初始化方法”。真正的构造方法是 `__new__`。
```
from collections import abc

class FrozenJSON:
"""一个只读接口，使用属性表示法访问JSON类对象 """

    def __new__(cls, arg):
        if isinstance(arg, abc.Mapping):
            return super().__new__(cls)
        elif isinstance(arg, abc.MutableSequence):
            return [cls(item) for item in arg]
        else:
            return arg

    def __init__(self, mapping):
        self.__data = {}
        for key, value in mapping.items():
            if iskeyword(key):
                key += '_'
            self.__data[key] = value

    def __getattr__(self, name):
        if hasattr(self.__data, name):
            return getattr(self.__data, name)
        else:
            return FrozenJSON(self.__data[name])
```

### 19.1.4 使用shelve模块调整OSCON数据源的结构

## 19.2 使用特性验证属性

### 19.2.1 LineItem 类第1版：表示订单中商品的类

### 19.2.2 LineItem 类第2版：能验证值的特性
```
class LineItem:
    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight  # 这里已经使用特性的设值方法了，确保所创建实例的 weight 属性不能为负值。
        self.price = price

    def subtotal(self):
        return self.weight * self.price

    @property
    def weight(self):
        return self.__weight  # 真正的值存储在私有属性 __weight 中。

    @weight.setter 
    def weight(self, value):
        if value > 0:
            self.__weight = value
        else:
            raise ValueError('value must be > 0')
```

## 19.3 特性全解析
内置的 property 经常用作装饰器，但它其实是一个类。

`property(fget=None, fset=None, fdel=None, doc=None)`
```
class LineItem:
    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight
        self.price = price

    def subtotal(self):
        return self.weight * self.price

    def get_weight(self):
        return self.__weight

    def set_weight(self, value):
        if value > 0:
            self.__weight = value
        else:
            raise ValueError('value must be > 0')

    weight = property(get_weight, set_weight)
```
### 19.3.1 特性会覆盖实例属性
实例和所属的类有同名数据属性，那么实例属性会覆盖(或称遮盖)类属性。实例属性不会遮盖类特性。新添的类特性遮盖现有的实例属性。
### 19.3.2 特性的文档

## 19.4 定义一个特性工厂函数
将定义一个名为 quantity 的特性工厂函数。
```
class LineItem:
    weight = quantity('weight')
    price = quantity('price')

    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight
        self.price = price
 
    def subtotal(self):
        return self.weight * self.price
```
```
def quantity(storage_name):

    def qty_getter(instance):
        return instance.__dict__[storage_name]
    def qty_setter(instance, value):
        if value > 0:
            instance.__dict__[storage_name] = value
        else:
            raise ValueError('value must be > 0')

    return property(qty_getter, qty_setter)
```
weight 特性覆盖了 weight 实例 属性，因此对 `self.weight` 或 `nutmeg.weight` 的每个引用都由特性函数处理，只有直接存取 `__dict__` 属性才能跳过特性的处理逻辑。

## 19.5 处理属性删除操作

## 19.6 处理属性的重要属性和函数
### 19.6.1 影响属性处理方式的特殊属性
`__class__`

对象所属类的引用(即 `obj.__class__` 与 type(obj) 的作用相同)。Python 的某些特殊方法，例如 `__getattr__`，只在对象的类中寻找，而不在实例中寻找。

`__dict__`

一个映射，存储对象或类的可写属性。有 `__dict__` 属性的对象，任何时候都能随意设置新属性。如果类有 `__slots__` 属性，它的实例可能没有 `__dict__` 属性。参见下面对 `__slots__` 属性的说明。

`__slots__`

类可以定义这个这属性，限制实例能有哪些属性。`__slots__` 属性的值是一个字符串组成的元组，指明允许有的属性。如果 `__slots__` 中没有 '`__dict__`'，那么该类的实例没有 `__dict__` 属性，实例只允许有指定名称的属性。

### 19.6.2 处理属性的内置函数
dir([object])

列出对象的大多数属性。官方文档(https://docs.python.org/3/library/functions.html#dir) 说，dir 函数的目的是交互式使用，因此没有提供完整的属性列表，只列出一组“重要的”属性名。dir 函数能审查有或没有 `__dict__` 属性的对象。dir 函数不会列出 `__dict__` 属性本身，但会列出其中的键。dir 函数也不会列出类的几个特殊属性，例如 `__mro__`、`__bases__` 和 `__name__`。如果没有指定可选的 object 参数，dir 函数会列出当 前作用域中的名称。

getattr(object, name[, default])

从 object 对象中获取 name 字符串对应的属性。获取的属性可能来自对象所属的类或超类。如果没有指定的属性，getattr 函数抛出 AttributeError 异常，或者返回 default 参数的值(如果设定了这个参数的话)。

hasattr(object, name)

如果 object 对象中存在指定的属性，或者能以某种方式(例如继承)通过 object 对象获取指定的属性，返回True。文档(https://docs.python.org/3/library/functions. html#hasattr)说道:“这个函数的实现方法是调用getattr(object, name)函数，看看是否抛出 AttributeError 异常。”

setattr(object, name, value)

把 object 对象指定属性的值设为 value，前提是 object 对象能接受那个值。这个函数可能会创建一个新属性，或者覆盖现有的属性。

vars([object])
返回 object 对象的 `__dict__` 属性;如果实例所属的类定义了 `__slots__` 属性，实例没 有 `__dict__` 属性，那么 vars 函数不能处理那个实例(相反，dir 函数能处理这样的实例)。如果没有指定参数，那么 `vars()` 函数的作用与 `locals()` 函数一样:返回表示本 地作用域的字典。

### 19.6.3 处理属性的特殊方法

`__delattr__(self, name)`

只要使用del语句删除属性，就会调用这个方法。例如，del obj.attr语句触发 `Class.__delattr__(obj, 'attr')` 方法。

`__dir__(self)`

把对象传给 dir 函数时调用，列出属性。例如，dir(obj) 触发 `Class.__dir__(obj)` 方法。

`__getattr__(self, name)`

仅当获取指定的属性失败，搜索过 obj、Class 和超类之后调用。表达式 `obj.no_such_ attr`、`getattr(obj, 'no_such_attr')`和`hasattr(obj, 'no_such_attr')`可能会触发 `Class.__getattr__(obj, 'no_such_attr')`方法，但是，仅当在obj、Class和超类中找不到指定的属性时才会触发。

`__getattribute__(self, name)`

尝试获取指定的属性时总会调用这个方法，不过，寻找的属性是特殊属性或特殊方法 时除外。点号与 `getattr` 和 `hasattr` 内置函数会触发这个方法。调用 `__getattribute__` 方法且抛出 AttributeError 异常时，才会调用 `__getattr__` 方法。为了在获取 obj 实例的属性时不导致无限递归，`__getattribute__` 方法的实现要使用 `super().__ getattribute__(obj, name)`。

`__setattr__(self, name, value)`

尝试设置指定的属性时总会调用这个方法。点号和 setattr 内置函数会触发这个方法。例如，`obj.attr = 42`和`setattr(obj, 'attr', 42)`都会触发`Class.__setattr__(obj, ‘attr’, 42)` 方法。
