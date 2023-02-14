---
title: Fluent Python 笔记 第 14 章 可迭代的对象、迭代器和生成器
tags: Python
pageview: true
---

迭代是数据处理的基石。扫描内存中放不下的数据集时，我们要找到一种惰性获取数据项的方式，即按需一次获取一个数据项。这就是迭代器模式(Iterator pattern)。本章说明 Python 语言是如何内置迭代器模式的，这样就避免了自己手动去实现。

在 Python 中，所有集合都可以迭代。在 Python 语言内部，迭代器用于支持:

- for循环
- 构建和扩展集合类型
- 逐行遍历文本文件
- 列表推导、字典推导和集合推导
- 元组拆包
- 调用函数时，使用 * 拆包实参

## 14.1 Sentence 类第1版:单词序列
### 序列可以迭代的原因：iter 函数
解释器需要迭代对象 x 时，会自动调用 iter(x)。内置的 iter 函数有以下作用：
- 检查对象是否实现了 `__iter__` 方法，如果实现了就调用它，获取一个迭代器。
- 如果没有实现 `__iter__` 方法，但是实现了 `__getitem__` 方法，Python 会创建一个迭代器，尝试按顺序(从索引 0 开始)获取元素。
- 如果尝试失败，Python 抛出 TypeError 异常，通常会提示“C object is not iterable”(C
对象不可迭代)，其中 C 是目标对象所属的类。

从 Python 3.4 开始，检查对象 x 能否迭代，最准确的方法是:调用 `iter(x)` 函数，如果不可迭代，再处理 TypeError 异常。这比使用 `isinstance(x, abc.Iterable)` 更准确，因为 `iter(x)` 函数会考虑到遗留的 `__getitem__` 方 法，而 `abc.Iterable` 类则不考虑。

## 14.2 可迭代的对象与迭代器的对比
关系：Python 从可迭代的对象中获取迭代器。

标准的迭代器接口有两个方法。

### `__next__`
返回下一个可用的元素，如果没有元素了，抛出 StopIteration 异常。
### `__iter__`
返回 self，以便在应该使用可迭代对象的地方使用迭代器，例如在 for 循环中。

## 14.3 Sentence 类第2版：典型的迭代器

这里之所以这么做，是为了清楚地说明可迭代的对象和迭代器之间的重要区别，以及二者之间的联系。

```
class Sentence:
    def __init__(self, text):
        self.text = text
        self.words = RE_WORD.findall(text)

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)

    def __iter__(self):
        return SentenceIterator(self.words)


class SentenceIterator:
    def __init__(self, words):
        self.words = words
        self.index = 0

    def __next__(self):
        try:
            word = self.words[self.index]
        except IndexError:
            raise StopIteration()
        self.index += 1
        return word

    def __iter__(self):
        return self
```

### 把 Sentence 变成迭代器：坏主意
可迭代的对象一定不能是自身的迭代器。也就是说，可迭代的对象必须实现 `__iter__` 方法，但不能实现 `__next__` 方法。

## 14.4 Sentence类第3版：生成器函数

```
def __iter__(self):
    for word in self.words:
        yield word
    return
```

### 生成器函数的工作原理
只要 Python 函数的定义体中有 yield 关键字，该函数就是生成器函数。调用生成器函数时，会返回一个生成器对象。

## 14.5 Sentence 类第4版:惰性实现
```
class Sentence:
    def __init__(self, text):
        self.text = text

    def __repr__(self):
        return 'Sentence(%s)' % reprlib.repr(self.text)

    def __iter__(self):
        for match in RE_WORD.finditer(self.text):
            yield match.group()
```

## 14.6 Sentence 类第5版：生成器表达式
惰性。
```
res2 = (x*3 for x in gen_AB())
```
```
def __iter__(self):
    return (match.group() for match in RE_WORD.finditer(self.text))
```

## 14.7 何时使用生成器表达式
如果生成器表达式要分成多行写，我倾向于定义生成器函数，以便提高可读性。此外，生成器函数有名称，因此可以重用。

## 14.8 另一个示例：等差数列生成器
把 `self.begin` 赋值给 `result`，不过会先强制转换成前面的加法算式得到的类型。
```
def __iter__(self):
    result = type(self.begin + self.step)(self.begin)
    forever = self.end is None
    index = 0
    while forever or result < self.end:
        yield result
        index += 1
        result = self.begin + self.step * index
```
如果一个类只是为了构建生成 器而去实现 `__iter__` 方法，那还不如使用生成器函数。

```
def aritprog_gen(begin, step, end=None):
    result = type(begin + step)(begin)
    forever = end is None
    index = 0
    while forever or result < end:
        yield result
        index += 1
        result = begin + step * index
```
### 使用 itertools 模块生成等差数列
```
def aritprog_gen(begin, step, end=None):
    first = type(begin + step)(begin)
    ap_gen = itertools.count(first, step)
    if end is not None:
        ap_gen = itertools.takewhile(lambda n: n < end, ap_gen)
    return ap_gen
```
实现生成器时要知道标准库中有什么可用，否则很可能会重新发明轮子。

## 14.9 标准库中的生成器函数
![4](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/4.png)
![5](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/5.png)
![6](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/6.png)
![7](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/7.png)
![8](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/8.png)

## 14.10 Python 3.3 中新出现的句法：yield from
`yield from i`完全代替了内层的for循环。i 是迭代器。
    
`yield from`还会创建通道，把内层生成器直接与外层生成器的客户端联系起来。把生成器当成协程使用时，这个通道特别重要，不仅能为客户端代码生成值，还能使用客户端代码提供的值。



## 14.11 可迭代的归约函数
all 和 any 函数会短路。

![9](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/9.png)


## 14.12 深入分析iter函数
在 Python 中迭代对象 x 时会调用 `iter(x)`。

iter 函数还有一个鲜为人知的用法:传入两个参数，使用常规的函数或任何可调用的对象创建迭代器。这样使用时，第一个参数必须是可调用的对象，用于不断调用(没有参数)，产出各个值;第二个值是哨符，这是个标记值，当可调用的对象返回这个值时，触发迭代器抛出 StopIteration 异常，而不产出哨符。
```
>>> def d6():
...     return randint(1, 6)
...
>>> d6_iter = iter(d6, 1)
>>> d6_iter
<callable_iterator object at 0x00000000029BE6A0>
>>> for roll in d6_iter:
...     print(roll)
...
4 3 6 3
```

## 14.13 案例分析:在数据库转换工具中使用生成器

## 14.14 把生成器当成协程
临时跳过。16 章讨论。
