---
title: Fluent Python 笔记 第 10 章 序列的修改、散列和切片
tags: Python
pageview: true
---

本章将以第 9 章定义的二维向量 Vector2d 类为基础，向前迈出一大步，定义表示多维向量的 Vector 类。这个类的行为与 Python 中标准的不可变扁平序列一样。

## 10.3 协议和鸭子类型
在 Python 中创建功能完善的序列类型无需使用继承，只需实现符合序列协议的方法。

Python 的序列协议只需要 `__len__` 和 `__getitem__` 两个方法。任何类(如 Spam)，只要使用标准的签名和语义实现了这两个方法，就能用在任何期待序列的地方。

## 10.4 Vector类第2版:可切片的序列

### 10.4.1 切片原理
`S.indices(len) -> (start, stop, stride)`
给定长度为 len 的序列，计算 S 表示的扩展切片的起始(start)和结尾(stop)索引，以及步幅(stride)。超出边界的索引会被截掉，这与常规切片的处理方式一样。

### 10.4.2 能处理切片的__getitem__方法
```
def __len__(self):
    return len(self._components)
def __getitem__(self, index):
    cls = type(self)
    if isinstance(index, slice):
        return cls(self._components[index])
    elif isinstance(index, numbers.Integral):
        return self._components[index]
    else:
        msg = '{cls.__name__} indices must be integers'
        raise TypeError(msg.format(cls=cls))
```

## 10.5 Vector类第3版:动态存取属性
属性查找失败后，解释器会调用 `__getattr__` 方法。简单来说，对 `my_obj.x` 表达式，Python 会检查 `my_obj` 实例有没有名为 x 的属性；如果没有，到类(`my_obj.__class__`)中查找;如果 还没有，顺着继承树继续查找。如果依旧找不到，调用 `my_obj` 所属类中定义的 `__getattr__` 方法，传入 `self` 和属性名称的字符串形式(如 'x')。

```
shortcut_names = 'xyzt'

def __getattr__(self, name):
    cls = type(self)
    if len(name) == 1:
        pos = cls.shortcut_names.find(name)
        if 0 <= pos < len(self._components):
            return self._components[pos]
    msg = '{.__name__!r} object has no attribute {!r}'
    raise AttributeError(msg.format(cls, name))
```


```
>>> v = Vector(range(5))
>>> v
Vector([0.0, 1.0, 2.0, 3.0, 4.0])
>>> v.x # ➊
0.0
>>> v.x = 10
>>> v.x # ➌
10
>>> v
Vector([0.0, 1.0, 2.0, 3.0, 4.0]) # ➍
```
`v.x = 10`这样赋值之 后，v 对象有 x 属性了，因此使用 v.x 获取 x 属性的值时不会调用 `__getattr__` 方法了，解释器直接返回绑定到 v.x 上的值，即 10。

修改：
```
def __setattr__(self, name, value):
    cls = type(self)
    if len(name) == 1:
        if name in cls.shortcut_names:
            error = 'readonly attribute {attr_name!r}' 
        elif name.islower():
            error = "can't set attributes 'a' to 'z' in {cls_name!r}"
        else:
            error = ''
        if error:
            msg = error.format(cls_name=cls.__name__, attr_name=name)
            raise AttributeError(msg)
    super().__setattr__(name, value)
```
多数时候，如果实 现了 `__getattr__` 方法，那么也要定义 `__setattr__` 方法，以防对象的行为不一致。

## 10.6 Vector类第4版:散列和快速等值测试
```
def __eq__(self, other):
    return len(self) == len(other) and all(a == b for a, b in zip(self, other))

def __hash__(self):
    hashes = map(hash, self._components)
    return functools.reduce(operator.xor, hashes, 0)
```

## 10.7 Vector类第5版:格式化
```
def __format__(self, fmt_spec=''):
    if fmt_spec.endswith('h'): # 超球面坐标
        fmt_spec = fmt_spec[:-1]
        coords = itertools.chain([abs(self)],self.angles())
        outer_fmt = '<{}>'
    else:
        coords = self
        outer_fmt = '({})'
    components = (format(c, fmt_spec) for c in coords)
    return outer_fmt.format(', '.join(components))
```
