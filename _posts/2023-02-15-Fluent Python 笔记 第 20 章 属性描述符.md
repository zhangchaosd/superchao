---
title: Fluent Python 笔记 第 20 章 属性描述符
tags: Python
pageview: true
---
描述符是实现了特定协议的类，这个协议包括 `__get__`、`__set__` 和 `__delete__` 方法。 property 类实现了完整的描述符协议。通常，可以只实现部分协议。其实，我们在真实的代码中见到的大多数描述符只实现了 `__get__` 和 `__set__` 方法，还有很多只实现了其中的一个。
描述符是 Python 的独有特征，不仅在应用层中使用，在语言的基础设施中也有用到。除了特性之外，使用描述符的 Python 功能还有方法及 classmethod 和 staticmethod 装饰器。理解描述符是精通 Python 的关键。本章的话题就是描述符。

## 20.1 描述符示例：验证属性
### 20.1.1 LineItem 类第3版:一个简单的描述符
实现了 `__get__`、`__set__` 或 `__delete__` 方法的类是描述符。描述符的用法是，创建一个实例，作为另一个类的类属性。
```
class Quantity:
    def __init__(self, storage_name):
        self.storage_name = storage_name

    def __set__(self, instance, value):
        if value > 0:
            instance.__dict__[self.storage_name] = value
        else:
            raise ValueError('value must be > 0')

class LineItem:
    weight = Quantity('weight')
    price = Quantity('price')

    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight
        self.price = price

    def subtotal(self):
        return self.weight * self.price
```
编写 `__set__` 方法时，要记住 self 和 instance 参数的意思:self 是描述符实例，instance 是托管实例。管理实例属性的描述符应该把值存储在托管实例中。因此，Python 才为描述符中的那个方法提供了 instance 参数。

为了理解错误的原因，可以想想 `__set__` 方法前两个参数(self 和 instance)的意思。 这里，self 是描述符实例，它其实是托管类的类属性。同一时刻，内存中可能有几千个 LineItem 实例，不过只会有两个描述符实例:`LineItem.weight` 和 `LineItem.price`。因此， 存储在描述符实例中的数据，其实会变成 LineItem 类的类属性，从而由全部 LineItem 实例共享。

### 20.1.2 LineItem 类第4版:自动获取储存属性的名称
```
class Quantity:
    __counter = 0

    def __init__(self):
        cls = self.__class__
        prefix = cls.__name__
        index = cls.__counter
        self.storage_name = '_{}#{}'.format(prefix, index)
        cls.__counter += 1

    def __get__(self, instance, owner):
        if instance is None:
            return self
        else:
            return getattr(instance, self.storage_name)

    def __set__(self, instance, value):
        if value > 0:
            setattr(instance, self.storage_name, value)
        else:
            raise ValueError('value must be > 0')

class LineItem:
    weight = Quantity()
    price = Quantity()

    def __init__(self, description, weight, price):
        self.description = description
        self.weight = weight
        self.price = price

    def subtotal(self):
        return self.weight * self.price
```

### 20.1.3 LineItem类第5版:一种新型描述符
重构后的描述符类
```
import abc

class AutoStorage:
    __counter = 0

    def __init__(self):
        cls = self.__class__
        prefix = cls.__name__
        index = cls.__counter
        self.storage_name = '_{}#{}'.format(prefix, index)
        cls.__counter += 1

    def __get__(self, instance, owner):
        if instance is None:
            return self
        else:
            return getattr(instance, self.storage_name)

    def __set__(self, instance, value):
        setattr(instance, self.storage_name, value)

class Validated(abc.ABC, AutoStorage):
    def __set__(self, instance, value):
        value = self.validate(instance, value)
        super().__set__(instance, value)

    @abc.abstractmethod
    def validate(self, instance, value)
    """return validated value or raise ValueError"""

class Quantity(Validated):
"""a number greater than zero"""
    def validate(self, instance, value):
        if value <= 0:
            raise ValueError('value must be > 0')
        return value

class NonBlank(Validated):
"""a string with at least one non-space character"""
    def validate(self, instance, value):
        value = value.strip()
        if len(value) == 0:
            raise ValueError('value cannot be empty or blank')
        return value
```

## 20.2 覆盖型与非覆盖型描述符对比
### 20.2.1 覆盖型描述符
实现 `__set__` 方法的描述符属于覆盖型描述符，因为虽然描述符是类属性，但是实现 `__set__` 方法的话，会覆盖对实例属性的赋值操作。
### 20.2.2 没有`__get__`方法的覆盖型描述符

### 20.2.3 非覆盖型描述符









先计算被装饰的类 ClassThree 的定义体，然后运行装饰器函数。








