---
title: Fluent Python 笔记 第 8 章 对象引用、可变性和垃圾回收
tags: Python
pageview: true
---

本章先以一个比喻说明 Python 的变量：变量是标注，而不是盒子。如果你不知道引用式变量是什么，可以像这样对别人解释别名。

然后，本章讨论对象标识、值和别名等概念。随后，本章会揭露元组的一个神奇特性：元组是不可变的，但是其中的值可以改变，之后就引申到浅复制和深复制。接下来的话题是引用和函数参数：可变的参数默认值导致的问题，以及如何安全地处理函数的调用者传入的可变参数。

本章最后一节讨论垃圾回收、del 命令，以及如何使用弱引用“记住”对象，而无需对象本身存在。

本章的内容有点儿枯燥，但是这些话题却是解决 Python 程序中很多不易察觉的 bug 的关键。 首先，我们要抛弃变量是存储数据的盒子这一错误观念。

## 8.1 变量不是盒子
```
a = [1, 2, 3]
b = a
a.append(4)
b
# [1, 2, 3, 4]
```
![6](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/6.png)
```
>>> class Gizmo:
        def __init__(self):
        print('Gizmo id: %d' % id(self)) ...
>>> x = Gizmo()
Gizmo id: 4301489152
>>> y = Gizmo() * 10
Gizmo id: 4301489432
Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    TypeError: unsupported operand type(s) for *: 'Gizmo' and 'int'
>>> dir()
['Gizmo', '__builtins__', '__doc__', '__loader__', '__name__', '__package__', '__spec__', 'x']
```
为了理解 Python 中的赋值语句，应该始终先读右边。对象在右边创建或获取，在此之后左边的变量才会绑定到对象上，这就像为对象贴上标注。忘掉盒子吧!

## 8.2 标识、相等性和别名
看两个别名是不是表示同一个对象：
`a is not b`
对象 ID 的真正意义在不同的实现中有所不同。在 CPython 中，id() 返回对象的内存地址，但是在其他 Python 解释器中可能是别的值。关键是，ID 一定是唯一的数值标注，而且在对象的生命周期中绝不会变。

### 8.2.1 在==和is之间选择
== 运算符比较两个对象的值(对象中保存的数据)，而 is 比较对象的标识。
is 运算符比 == 速度快，因为它不能重载。而`a == b`是语法糖，等同于`a.__eq__(b)`。

### 8.2.2 元组的相对不可变性
```
t1 = (1, 2, [30, 40])
t1[-1].append(99)  # 可以运行
```

## 8.3 默认做浅复制
对列表和其他可变序列来说，还能使用简洁的 `l2 = l1[:]` 语句创建副本。

然而，构造方法或 `[:]` 做的是浅复制(即复制了最外层容器，副本中的元素是源容器中元素的引用)。

下图来自 https://pythontutor.com
![7](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/7.png)

注意元组的 `+=` 会连接两个元组，生成一个新的。

### 为任意对象做深复制和浅复制
`deepcopy` 函数会记住已经复制的对象，因此能优雅地处理循环引用。
```
bus1 = Bus(['Alice', 'Bill', 'Claire', 'David'])
bus2 = copy.copy(bus1)  # 浅复制
bus3 = copy.deepcopy(bus1)  # 深复制
```

## 8.4 函数的参数作为引用时
Python 唯一支持的参数传递模式是共享传参(call by sharing)。

函数可能会修改作为参数传入的可变对象，但是无法修改那些对象的标识(即不能把一个对象替换成另一个对象)。

```
>>> def f(a, b):
... a += b
     ...     return a
     ...
     >>> x = 1
     >>> y = 2
     >>> f(x, y)
     3
>>> x, y
(1, 2)
>>> a = [1, 2]
>>> b = [3, 4]
>>> f(a, b)
[1, 2, 3, 4]
>>> a, b
([1, 2, 3, 4], [3, 4]) >>> t = (10, 20)
>>> u = (30, 40)
>>> f(t, u)
(10, 20, 30, 40)
>>> t, u
((10, 20), (30, 40))

```

### 8.4.1 不要使用可变类型作为参数的默认值
默认值在定义函数时计算(通常在加载模块时)，因此默认值变成了函数对象的属性。因此，如果默认值是可变对象，而且修改了它的值，那么后续的函数调用都会受到影响。

除非方法确实想修改通过参数传入的对象，否则在类中直接把参数赋值给实例变量之前一定要三思，因为这样会为参数对象创建别名。如果不确定，那就创建副本。这样客户会少些麻烦。

```
def __init__(self, passengers=None):
    if passengers is None:
        self.passengers = []
    else:
        self.passengers = passengers  # 这里其实是别名，可以修改到外面的 list
```

## 8.5 del和垃圾回收
del 语句删除名称，而不是对象。del 命令可能会导致对象被当作垃圾回收，但是仅当删除的变量保存的是对象的最后一个引用，或者无法得到对象时。重新绑定也可能会导致对象的引用数量归零，导致对象被销毁。

有个 `__del__` 特殊方法，但是它不会销毁实例，不应该在代码中调用。即将销毁实例时，Python 解释器会调用 `__del__` 方法，给实例最后的机会，释放外部资源。自己编写的代码很少需要实现 `__del__` 代码，有些 Python 新手会花时间实现，但却吃力不讨好，因为 `__del__` 很难用对。

## 8.6 弱引用
弱引用不会增加对象的引用数量。引用的目标对象称为所指对象(referent)。因此我们说，弱引用不会妨碍所指对象被当作垃圾回收。弱引用在缓存应用中很有用，因为我们不想仅因为被缓存引用着而始终保存缓存对象。弱引用是可调用的对象，返回的是被引用的对象；如果所指对象不存在了，返回 None。

`weakref` 模块的文档(http://docs.python.org/3/library/weakref.html)指出，`weakref.ref` 类其实是低层接口，供高级用途使用，多数程序最好使用 `weakref` 集合和 `finalize`。也就是说，应该使用 `WeakKeyDictionary、WeakValueDictionary、WeakSet` 和 `finalize`(在内部使用弱引用)，不要自己动手创建并处理 `weakref.ref` 实例。我们在示例 8-17 中那么做是希望借 助实际使用 `weakref.ref` 来褪去它的神秘色彩。但是实际上，多数时候 Python 程序都使用 `weakref` 集合。

### 8.6.1 WeakValueDictionary简介
WeakValueDictionary 类实现的是一种可变映射，里面的值是对象的弱引用。被引用的对象 在程序中的其他地方被当作垃圾回收后，对应的键会自动从 WeakValueDictionary 中删除。 因此，WeakValueDictionary 经常用于缓存。

```
class Cheese:
    def __init__(self, kind):
        self.kind = kind
    def __repr__(self):
        return 'Cheese(%r)' % self.kind

>>> import weakref
>>> stock = weakref.WeakValueDictionary()
>>> catalog = [Cheese('Red Leicester'), Cheese('Tilsit'), Cheese('Brie'), Cheese('Parmesan')]
>>> for cheese in catalog:
        stock[cheese.kind] = cheese

>>> sorted(stock.keys())
['Brie', 'Parmesan', 'Red Leicester', 'Tilsit']
>>> del catalog
>>> sorted(stock.keys())
['Parmesan']
>>> del cheese
>>> sorted(stock.keys())
[]
```
`Parmesan` 存在的原因：`for` 循环中的变量 `cheese` 是全局变量，除非显式删除，否则不会消失。

与 `WeakValueDictionary` 对应的是 `WeakKeyDictionary，` 后者的键是弱引用。

(`WeakKeyDictionary` 实例)可以为应用中其他部分拥有的对象附加数据，这样就无需为对象添加属性。这对覆盖属性访问权限的对象尤其有用。

`weakref` 模块还提供了 `WeakSet` 类，按照文档的说明，这个类的作用很简单：“保存元素弱引用的集合类。元素没有强引用时，集合会把它删除。”如果一个类需要知道所有实例，一种好的方案是创建一个 `WeakSet` 类型的类属性，保存实例的引用。如果使用常规的 `set`，实例永远不会被垃圾回收，因为类中有实例的强引用，而类存在的时间与 Python 进程一样长，除非显式删除类。

## 8.6.2 弱引用的局限

不是每个 Python 对象都可以作为弱引用的目标(或称所指对象)。基本的 list 和 dict 实例不能作为所指对象，但是它们的子类可以轻松地解决这个问题。

set 实例可以作为所指对象，因此实例 8-17 才使用 set 实例。用户定义的类型也没问题， 这就解释了示例 8-19 中为什么使用那个简单的 Cheese 类。但是，int 和 tuple 实例不能作为弱引用的目标，甚至它们的子类也不行。

这些局限基本上是 CPython 的实现细节，在其他 Python 解释器中情况可能不一样。这些局限是内部优化导致的结果，下一节会以其中几个类型为例讨论(完全选读)。

## 8.7 Python对不可变类型施加的把戏
对元组 t 来说，`t[:]` 不创建副本，而是返回同一个对象的引用。此外， `tuple(t)` 获得的也是同一个元组的引用。str、bytes 和 frozenset 实例也有这种行为。

注意，frozenset 实例不是序列，因此不能使用 fs[:](fs 是一个 frozenset 实例)。但是，fs.copy() 具有相同的效果：它会欺骗你，返回同一个对象的引用，而不是创建一个副本。

共享字符串字面量是一种优化措施，称为驻留(interning)。CPython 还会在小的整数上使用这个优化措施，防止重复创建“热门”数字，如 0、—1 和 42。注意，CPython 不会驻留所有字符串和整数，驻留的条件是实现细节，而且没有文档说明。

千万不要依赖字符串或整数的驻留!比较字符串或整数是否相等时，应该使 用 ==，而不是 is。
