---
title: Fluent Python 笔记 第 4 章 文本和字节序列
tags: Python
pageview: true
---

Python 3 明确区分了人类可读的文本字符串和原始的字节序列。隐式地把字节序列转换成 Unicode 文本已成过去。本章将要讨论 Unicode 字符串、二进制序列，以及在二者之间转 换时使用的编码。
没啥可看的，就一句话，一定不能依赖默认编码，传参数 utf-8。

## 4.1 字符问题

```
>>> s = 'café'
>>> len(s)
4
>>> b = s.encode('utf8')
>>> b
b'caf\xc3\xa9'
>>> len(b)
5
>>> b.decode('utf8')
'café'
```

## 4.2 字节概要

## 4.3 基本的编解码器

## 4.4 了解编解码问题

### 4.4.1 处理 UnicodeEncodeError
```
city.encode('cp437', errors='ignore')
city.encode('cp437', errors='replace')
city.encode('cp437', errors='xmlcharrefreplace')
```
### 4.4.2 处理 UnicodeDecodeError
```
octets.decode('utf_8', errors='replace')
```

### 4.4.3 使用预期之外的编码加载模块时抛出的 SyntaxError
py 文件用 UTF-8

### 4.4.4 如何找出字节序列的编码
不能。必须有人告诉你。

或者用 Chardet 库。对应的命令行程序是 chardetect。

### 4.4.5 BOM:有用的鬼符
BOM，即字节序标记(byte-order mark)

`b'\xff\xfe'` 指明编码时使用 Intel CPU 的小字节序。在小字节序设备中，各个码位的最低有效字节在前面。

UTF-16 有两个变种:UTF-16LE，显式指明使用小字节序;UTF-16BE，显式指明使用大字 节序。如果使用这两个变种，不会生成 BOM。

根据标准，如果文件使用 UTF-16 编码，而且没有 BOM，那么应 该假定它使用的是 UTF-16BE(大字节序)编码。然而，Intel x86 架构用的是小字节序， 因此有很多文件用的是不带 BOM 的小字节序 UTF-16 编码。

## 4.5 处理文本文件
处理文本的最佳实践是“Unicode 三明治”。
要尽早把输入(例 如读取文件时)的字节序列解码成字符串。对输出来说，则 要尽量晚地把字符串编码成字节序列。

需要在多台设备中或多种场合下运行的代码，一定不能依赖默认编码。打开文件时始终应该明确传入 encoding= 参数，因为不同的设备使用的默认编码可能不同，有时隔一天也会发生变化。

![0](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230211/0.png)

## 4.6 为了正确比较而规范化Unicode字符串
因为 Unicode 有组合字符(变音符号和附加到前一个字符上的记号，打印时作为一个整
体)，所以字符串比较起来很复杂。
例如，“café”这个词可以使用两种方式构成，分别有 4 个和 5 个码位，但是结果完全一样。

这个问题的解决方案是使用 `unicodedata.normalize` 函数提供的 Unicode 规范化。这个函数 的第一个参数是这 4 个字符串中的一个:'NFC'、'NFD'、'NFKC' 和 'NFKD'。下面先说明前 两个。

`NFC(Normalization Form C)` 使用最少的码位构成等价的字符串，而 NFD 把组合字符分 解成基字符和单独的组合字符。
安全起见，保存文本之前，最好使用 `normalize('NFC', user_text)` 清洗字符串。

在另外两个规范化形式(NFKC 和 NFKD)的首字母缩略词中，字母 K 表示“compatibility” (兼容性)。这两种是较严格的规范化形式，对“兼容字符”有影响。使用 NFKC 和 NFKD 规范化形式时要小心，而且只能在特殊情况中使用，例 如搜索和索引，而不能用于持久存储，因为这两种转换会导致数据损失。

### 4.6.1 大小写折叠
绝大多数情况下， `s.casefold()` 得到的结果与 `s.lower()` 一样

### 4.6.2 规范化文本匹配实用函数
```
from unicodedata import normalize

def nfc_equal(str1, str2):
    return normalize('NFC', str1) == normalize('NFC', str2)
def fold_equal(str1, str2):
    return (normalize('NFC', str1).casefold() == normalize('NFC', str2).casefold())
```

### 4.6.3 极端“规范化”:去掉变音符号
好像也用不太到，省略。

## 4.7 Unicode文本排序
在 Python 中，非 ASCII 文本的标准排序方式是使用 locale.strxfrm 函数。根据 locale 模块的文档(https://docs.python.org/3/library/locale.html?highlight=strxfrm#locale.strxfrm)，这个函数会“把字符串转换成适合所在区域进行比较的形式”。

直接用 PyUCA 库。

### 使用Unicode排序算法排序
```
>>> import pyuca
>>> coll = pyuca.Collator()
>>> fruits = ['caju', 'atemoia', 'cajá', 'açaí', 'acerola']
>>> sorted_fruits = sorted(fruits, key=coll.sort_key)
>>> sorted_fruits
['açaí', 'acerola', 'atemoia', 'cajá', 'caju']
```

## 4.8 Unicode数据库
Unicode 标准提供了一个完整的数据库(许多格式化的文本文件)，不仅包括码位与字符名称之间的映射，还有各个字符的元数据，以及字符之间的关系。例如，Unicode 数据库记录了字符是否可以打印、是不是字母、是不是数字，或者是不是其他数值符号。字符串的 isidentifier、isprintable、isdecimal 和 isnumeric 等方法就是靠这些信息作判断的。str.casefold 方法也用到了 Unicode 表中的信息。

## 4.9 支持字符串和字节序列的双模式API
