---
title: Fluent Python 笔记 第 2 章 序列构成的数组
tags: Python
pageview: true
---


## 2.1 内置类型序列概览

容器序列（能存放不同类型的数据）：(作者分的类)

```
list、tuple 和 collections.deque
```

扁平序列（只能容纳一种类型）：
```
str、byes、bytearray、memoryview 和 array.array
```

可变：
```
list、bytearray、array.array、collections.deque 和 memoryview
```

不可变：
```
tuple、str 和 bytes
```

## 2.2 列表推导和生成器表达式

很多 Python 程序员都把列表推导(list comprehension)简称为 listcomps，生成器表达式(generator expression)则称为 genexps。

Python 会忽略代码里 []、{} 和 () 中的换行

### 2.2.1 列表推导和可读性
```
l2 = [fc(itm) for itm in l1]
```
### 2.2.2 列表推导同 filter 和 map 的比较
```
beyond_ascii = [ord(s) for s in symbols if ord(s) > 127]
beyond_ascii = list(filter(lambda c: c > 127, map(ord, symbols)))
```

### 2.2.3 笛卡儿积
```
tshirts = [(color, size) for color in colors for size in sizes]
```

### 2.2.4 生成器表达式
生成器表达式的语法跟列表推导差不多，只不过把方括号换成圆括号而已。
```
tuple(ord(symbol) for symbol in symbols)
```
生成器表达式之后， 内存里不会留下一个组合的列表，因为生成器表达式会在每次 for 循环运行时才生成一个组合。

## 2.3 元组不仅仅是不可变的列表
### 2.3.1 元组和记录
### 2.3.2 元组拆包
```
lax_coordinates = (33.9425, -118.408056)
latitude, longitude = lax_coordinates # 元组拆包
b, a = a, b
```
用 * 来处理剩下的元素
```
>>> a, b, *rest = range(5)
>>> a, b, rest
(0, 1, [2, 3, 4])
>>> a, b, *rest = range(3)
>>> a, b, rest
(0, 1, [2])
>>> a, b, *rest = range(2)
>>> a, b, rest
(0, 1, [])

>>> a, *body, c, d = range(5)
>>> a, body, c, d
(0, [1, 2], 3, 4)
>>> *head, b, c, d = range(5)
>>> head, b, c, d
([0, 1], 2, 3, 4)
```

### 2.3.3 嵌套元组拆包
```
name, cc, pop, (latitude, longitude) = ('Tokyo','JP',36.933,(35.689722,139.691667))
```
### 2.3.4 具名元组
collections.namedtuple 是一个工厂函数。用 namedtuple 构建的类的实例所消耗的内存跟元组是一样的，因为字段名都 被存在对应的类里面。这个实例跟普通的对象实例比起来也要小一些，因为 Python 不会用 `__dict__` 来存放这些实例的属性。

创建一个具名元组需要两个参数，一个是类名，另一个是类的各个字段的名字。后者可 以是由数个字符串组成的可迭代对象，或者是由空格分隔开的字段名组成的字符串
```
from collections import namedtuple

City = namedtuple('City', 'name country population coordinates')
tokyo = City('Tokyo', 'JP', 36.933, (35.689722, 139.691667))
```
元组有一些自己专有的属性。展示了几个最有用的:_fields 类属性、类方法 _make(iterable) 和实例方法 _asdict()。
```
>>> City._fields
('name', 'country', 'population', 'coordinates')
>>> LatLong = namedtuple('LatLong', 'lat long')
>>> delhi_data = ('Delhi NCR', 'IN', 21.935, LatLong(28.613889, 77.208889)) >>> delhi = City._make(delhi_data)
>>> delhi._asdict()
OrderedDict([('name', 'Delhi NCR'), ('country', 'IN'), ('population', 21.935), ('coordinates', LatLong(lat=28.613889, long=77.208889))])
```
### 2.3.5 作为不可变列表的元组
除了跟增减元素相关的方法之外，元组支持列表的其他所有方法。还有一个例 外，元组没有 `__reversed__` 方法，但是这个方法只是个优化而已，reversed(my_tuple) 这 个用法在没有 `__reversed__` 的情况下也是合法的。

## 2.4 切片
### 2.4.1 为什么切片和区间会忽略最后一个元素
在切片和区间操作里不包含区间范围的最后一个元素是 Python 的风格，这个习惯符合
Python、C 和其他语言里以 0 作为起始下标的传统。

### 2.4.2 对对象进行切片
```
>>> s = 'bicycle'
>>> s[::3]
'bye'
>>> s[::-1]
'elcycib'
>>> s[::-2]
'eccb'
```
可以给切片起名字增强可读性！：
```
SKU = slice(0, 6)
print(item[SKU]
```
### 2.4.3 多维切片和省略
省略（ellipsis）。ellipsis 是类名，全小写，而它的内置实例写作 Ellipsis。这其实跟 bool 是小写，但是它的两个实例写作 True 和 False 异曲同工。

### 2.4.4 给切片赋值
对原序列就地修改！如果赋值的对象是一个切片，那么赋值语句的右侧必须是个可迭代对象。即便只有单独 一个值，也要把它转换成可迭代的序列。

## 2.5 对序列使用 + 和 *
```
>>> l = [1, 2, 3]
>>> l * 5
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3, 1, 2, 3]
>>> 5 * 'abcd'
'abcdabcdabcdabcdabcd'
```
\+ 和 * 都遵循这个规律，不修改原有的操作对象，而是构建一个全新的序列。

如果在a * n这个语句中，序列a里的元素是对其他可变对象的引用的话， 你就需要格外注意了，因为这个式子的结果可能会出乎意料。比如，你想用 my_list = [[]] * 3来初始化一个由列表组成的列表，但是你得到的列表里 包含的 3 个元素其实是 3 个引用，而且这 3 个引用指向的都是同一个列表。 这可能不是你想要的效果。

正确方式：
```
board = [['_'] * 3 for i in range(3)]
```

## 2.6 序列的增量赋值
+= 背后的特殊方法是 `__iadd__`(用于“就地加法”)。但是如果一个类没有实现这个方法的
话，Python 会退一步调用 `__add__`。

如果 a 实现了 __iadd__ 方法，就会调用这个方法。同时对可变序列(例如 list、 bytearray 和 array.array)来说，a 会就地改动，就像调用了 a.extend(b) 一样。但是如 果 a 没有实现 __iadd__ 的话，a += b 这个表达式的效果就变得跟 a = a + b 一样了:首先 计算 a + b，得到一个新的对象，然后赋值给 a。也就是说，在这个表达式中，变量名会不 会被关联到新的对象，完全取决于这个类型有没有实现 __iadd__ 这个方法。而不可变序列根 本就不支持这个操作，对这个方法的实现也就无从谈起。
```
>>> l = [1, 2, 3]
>>> id(l) 4311953800
>>> l *= 2
>>> l
[1, 2, 3, 1, 2, 3]
>>> id(l)
4311953800
>>> t = (1, 2, 3)
>>> id(t)
4312681568
>>> t *= 2
>>> id(t)
4301348296
```

一个关于+=的谜题
```
>>> t = (1, 2, [30, 40])
>>> t[2] += [50, 60]
```
到底会发生下面 4 种情况中的哪一种?
- a. t 变成 (1, 2, [30, 40, 50, 60])。
- b. 因为 tuple 不支持对它的元素赋值，所以会抛出 TypeError 异常。
- c. 以上两个都不是。
- d. a 和 b 都是对的。

答案是 d
```
>>> t = (1, 2, [30, 40])
>>> t[2] += [50, 60]
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> t
(1, 2, [30, 40, 50, 60])
```
```
- 将 s[a] 的值存入 TOS(Top Of Stack，栈的顶端)。
- 计算 TOS += b。这一步能够完成，是因为 TOS 指向的是一个可变对象。
- s[a] = TOS 赋值。这一步失败，是因为 s 是不可变的元组。
```

## 2.7 list.sort方法和内置函数sorted
list.sort 方法会就地排序列表，返回 None。sorted，它会新建一个列表作为返回值。

两个可选的关键字参数。

reverse：

如果被设定为 True，被排序的序列里的元素会以降序输出(也就是说把最大值当作最 小值来排序)。这个参数的默认值是 False。

key：

一个只有一个参数的函数，这个函数会被用在序列里的每一个元素上，所产生的结果 将是排序算法依赖的对比关键字。

可选参数 key 还可以在内置函数 min() 和 max() 中起作用。另外，还有些标准库里的函数也接受这个参数，像 itertools.groupby() 和 heapq.nlargest() 等。

## 2.8 用bisect来管理已排序的序列
`import bisect`
### 2.8.1 用bisect来搜索
bisect 函数其实是 bisect_right 函数的别名，后者还有个姊妹函数叫 bisect_left。 它们的区别在于，bisect_left 返回的插入位置是原序列中跟被插入元素相等的元素的位置， 也就是新元素会被放置于它相等的元素的前面，而 bisect_right 返回的则是跟它相等的元素之后的位置。

两个可选参数 lo 和 hi 来缩小搜寻的范围。lo 的默认值是 0，hi 的默认值是序列的长度，即 len() 作用于该序列的返回值。

### 2.8.2 用bisect.insort插入新元素
`bisect.insort(my_list, new_item)`

insort 跟 bisect 一样，有 lo 和 hi 两个可选参数用来控制查找的范围。也有个变体叫 insort_left。

## 2.9 当列表不是首选时
### 2.9.1 数组
速度非常快
```
from array import array

floats = array('d', (random() for i in range(10**7)))
fp = open('floats.bin', 'wb')
floats.tofile(fp)
fp.close()

floats2 = array('d')
fp = open('floats.bin', 'rb')
floats2.fromfile(fp, 10**7)
fp.close()
```
从 Python 3.4 开始，数组(array)类型不再支持诸如 list.sort() 这种就地排序方法。要给数组排序的话，得用 sorted 函数新建一个数组: `a = array.array(a.typecode, sorted(a))` 想要在不打乱次序的情况下为数组添加新的元素，bisect.insort 还是能派上用场。

### 2.9.2 内存视图
低级视角
```
numbers = array.array('h', [-2, -1, 0, 1, 2]) >>> memv = memoryview(numbers)
len(memv)
# 5
>>> memv[0]
# -2
memv_oct = memv.cast('B')
memv_oct.tolist()
# [254, 255, 255, 255, 0, 0, 1, 0, 2, 0] >>> memv_oct[5] = 4
numbers
# array('h', [-2, -1, 1024, 1, 2])
```

### 2.9.3 NumPy 和 SciPy

### 2.9.4 双向队列和其他形式的队列

collections.deque 类(双向队列)是一个线程安全、可以快速从两端添加或者删除元素的数据类型。
```
dq = deque(range(10), maxlen=10)
```
maxlen 无法修改。
extendleft(iter) 方法会把迭代器里的元素逐个添加到双向队列的左边，因此迭代器里的元素会逆序出现在队列里。
```
>>> from collections import deque
>>> dq = deque(range(10), maxlen=10)
>>> dq
deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10)
>>> dq.rotate(3)
>>> dq
deque([7, 8, 9, 0, 1, 2, 3, 4, 5, 6], maxlen=10)
>>> dq.rotate(-4)
>>> dq
deque([1, 2, 3, 4, 5, 6, 7, 8, 9, 0], maxlen=10)
>>> dq.appendleft(-1)
>>> dq
deque([-1, 1, 2, 3, 4, 5, 6, 7, 8, 9], maxlen=10) >>> dq.extend([11, 22, 33])
>>> dq
deque([3, 4, 5, 6, 7, 8, 9, 11, 22, 33], maxlen=10) >>> dq.extendleft([10, 20, 30, 40])
>>> dq
deque([40, 30, 20, 10, 3, 4, 5, 6, 7, 8], maxlen=10)
```
append 和 popleft 都是原子操作，也就说是 deque 可以在多线程程序中安全地当作先进先 出的队列使用，而使用者不需要担心资源锁的问题。

#### queue
提供了同步(线程安全)类 Queue、LifoQueue 和 PriorityQueue，不同的线程可以利用 这些数据类型来交换信息。这三个类的构造方法都有一个可选参数 maxsize，它接收正 整数作为输入值，用来限定队列的大小。但是在满员的时候，这些类不会扔掉旧的元素 来腾出位置。相反，如果队列满了，它就会被锁住，直到另外的线程移除了某个元素而 腾出了位置。这一特性让这些类很适合用来控制活跃线程的数量。
#### multiprocessing
这个包实现了自己的 Queue，它跟 queue.Queue 类似，是设计给进程间通信用的。同时还有一个专门的 multiprocessing.JoinableQueue 类型，可以让任务管理变得更方便。
#### asyncio
Python 3.4 新 提 供 的 包， 里 面 有 Queue、LifoQueue、PriorityQueue 和 JoinableQueue， 这些类受到 queue 和 multiprocessing 模块的影响，但是为异步编程里的任务管理提供 了专门的便利。
#### heapq
跟上面三个模块不同的是，heapq 没有队列类，而是提供了 heappush 和 heappop 方法，让用户可以把可变序列当作堆队列或者优先队列来使用。