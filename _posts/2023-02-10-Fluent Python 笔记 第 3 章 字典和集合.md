---
title: Fluent Python 笔记 第 3 章 字典和集合
tags: Python
pageview: true
---


## 3.1 泛映射类型

只有可散列 的数据类型才能用作这些映射里的键

字典构造方法：
```
>>> a = dict(one=1, two=2, three=3)
>>> b = {'one': 1, 'two': 2, 'three': 3}
>>> c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
>>> d = dict([('two', 2), ('one', 1), ('three', 3)])
>>> e = dict({'three': 3, 'one': 1, 'two': 2})
>>> a == b == c == d == e
True
```
还有字典推导

## 3.2 字典推导
```
DIAL_CODES = [
    (86, 'China'),
    (91, 'India'),
    ]
country_code = {country: code for code, country in DIAL_CODES}
```
## 3.3 常见的映射方法
用setdefault处理找不到的键

```
index.setdefault(word, []).append(location)
```

## 3.4 映射的弹性键查询

### 3.4.1 defaultdict:处理找不到的键的一个选择
在实例化一个 defaultdict 的时候，需要给构造方法提供一个可调用对象，这 个可调用对象会在 `__getitem__` 碰到找不到的键的时候被调用，让 `__getitem__` 返回某种 默认值。

defaultdict 里的 default_factory 只会在 `__getitem__` 里被调用，在其他的 方法里完全不会发挥作用。比如，dd 是个 defaultdict，k 是个找不到的键， dd[k] 这个表达式会调用 default_factory 创造某个默认值，而 dd.get(k) 则 会返回 None。
```
index = collections.defaultdict(list)
```
### 3.4.2 特殊方法__missing__

`__missing__` 方法只会被 `__getitem__` 调用(比如在表达式 d[k] 中)。提供 `__missing__` 方法对 get 或者 `__contains__`(in 运算符会用到这个方法)这些方法的使用没有影响。这也是在上一节最后的警告中提到，defaultdict 中 的 default_factory 只对 `__getitem__` 有作用的原因。

```
class StrKeyDict0(dict):
    def __missing__(self, key):
        if isinstance(key, str):
            raise KeyError(key) return self[str(key)]
    def get(self, key, default=None):
        try:
            return self[key]
        except KeyError:
            return default
    def __contains__(self, key):
        return key in self.keys() or str(key) in self.keys()
```

下面来看看为什么 isinstance(key, str) 测试在上面的 `__missing__` 中是必需的。
如果没有这个测试，只要 str(k) 返回的是一个存在的键，那么 `__missing__` 方法是没问题 的，不管是字符串键还是非字符串键，它都能正常运行。但是如果 str(k) 不是一个存在的 键，代码就会陷入无限递归。这是因为 `__missing__` 的最后一行中的 self[str(key)] 会调 用 `__getitem__，而这个` str(key) 又不存在，于是 `__missing__` 又会被调用。为了保持一致性，`__contains__` 方法在这里也是必需的。这是因为 k in d 这个操作会调用 它，但是我们从 dict 继承到的 `__contains__` 方法不会在找不到键的时候调用 `__missing__` 方法。`__contains__` 里还有个细节，就是我们这里没有用更具 Python 风格的方式——k in my_dict——来检查键是否存在，因为那也会导致 `__contains__` 被递归调用。为了避免这 一情况，这里采取了更显式的方法，直接在这个 self.keys() 里查询

## 3.5 字典的变种

### collections.OrderedDict

这个类型在添加键的时候会保持顺序，因此键的迭代次序总是一致的。OrderedDict 的 popitem 方法默认删除并返回的是字典里的最后一个元素，但是如果像 my_odict. popitem(last=False) 这样调用它，那么它删除并返回第一个被添加进去的元素。

### collections.ChainMap

该类型可以容纳数个不同的映射对象，然后在进行键查找操作的时候，这些对象会被当 作一个整体被逐个查找，直到键被找到为止。这个功能在给有嵌套作用域的语言做解 释器的时候很有用，可以用一个映射对象来代表一个作用域的上下文。在 collections 文 档 介 绍 ChainMap 对 象 的 那 一 部 分(https://docs.python.org/3/library/collections.html# collections.ChainMap)里有一些具体的使用示例，其中包含了下面这个 Python 变量查 询规则的代码片段:
```
import builtins
pylookup = ChainMap(locals(), globals(), vars(builtins))
```

### collections.Counter

这个映射类型会给键准备一个整数计数器。每次更新一个键的时候都会增加这个计数 器。所以这个类型可以用来给可散列表对象计数，或者是当成多重集来用——多重集合 就是集合里的元素可以出现不止一次。Counter 实现了 + 和 - 运算符用来合并记录，还 有像 most_common([n]) 这类很有用的方法。most_common([n]) 会按照次序返回映射里最 常见的 n 个键和它们的计数

```
>>> ct = collections.Counter('abracadabra')
>>> ct
Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})
>>> ct.update('aaaaazzz')
>>> ct
Counter({'a': 10, 'z': 3, 'b': 2, 'r': 2, 'c': 1, 'd': 1})
>>> ct.most_common(2)
[('a', 10), ('z', 3)]
```

### colllections.UserDict

这个类其实就是把标准 dict 用纯 Python 又实现了一遍。

## 3.6 子类化UserDict
```
import collections

class StrKeyDict(collections.UserDict): ➊
    def __missing__(self, key): ➋ 
        if isinstance(key, str):
            raise KeyError(key)
        return self[str(key)]
    def __contains__(self, key):
        return str(key) in self.data ➌
    def __setitem__(self, key, item): 
        self.data[str(key)] = item
```
- `__contains__` 则更简洁些。这里可以放心假设所有已经存储的键都是字符串。因此，只
要在 self.data 上查询就好了，并不需要像 StrKeyDict0 那样去麻烦 self.keys()。
- `__setitem__` 会把所有的键都转换成字符串。由于把具体的实现委托给了 self.data 属
 性，这个方法写起来也不难。

 ## 3.7 不可变映射类型

```
from types import MappingProxyType

d = {1:'A'}
d_proxy = MappingProxyType(d)
```
可以通过 d 改，d_proxy 可以看到改动，但不能修改。

## 3.8 集合论

集合中的元素必须是可散列的，set 类型本身是不可散列的，但是 frozenset 可以。因此可以创建一个包含不同 frozenset 的 set。

### 3.8.1 集合字面量

如果要创建一个空集，你必须用不带任何参数的构造方法 set()。
```
frozenset(range(10))
frozenset({0, 1, 2, 3, 4, 5, 6, 7, 8, 9})
```

### 3.8.2 集合推导
```
{fc(i) for i in *** if ***)}
```

### 3.8.3 集合的操作

![0](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230210/0.png)
![1](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230210/1.png)
![2](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230210/2.png)


## 3.9 dict和set的背后

### 3.9.1 一个关于效率的实验
快就完事



### 3.9.2 字典中的散列表
#### 1. 散列值和相等性
如果两个对象在比较的时候是相等的，那它们的散列值必须相等，否则散列表就不能正常运行了。
#### 2. 散列表算法
Python 首 先 会 调 用 hash(search_key) 来 计 算 search_key 的散列值，把这个值最低的几位数字当作偏移量，在散列表里查找表元(具 体取几位，得看当前散列表的大小)。若找到的表元是空的，则抛出 KeyError 异常。若不 是空的，则表元里会有一对found_key:found_value。这时候Python会检验search_key == found_key 是否为真，如果它们相等的话，就会返回 found_value。

### 3.9.3 dict的实现及其导致的结果
1. 键必须是可散列的
2. 字典在内存上的开销巨大。如果你需要存放数量巨大的记录，那么放在由元组或是具名元组构成的列表中会是比较好的选择。
3. 键查询很快
4. 键的次序取决于添加顺序
5. 往字典里添加新键可能会改变已有键的顺序

不要对字典同时进行迭代和修改。如果想扫描并修改一个字典，最好分成两步 来进行:首先对字典迭代，以得出需要添加的内容，把这些内容放在一个新字典里;迭代 结束之后再对原有字典进行更新。

### 3.9.4 set的实现以及导致的结果

和前一部分一样

