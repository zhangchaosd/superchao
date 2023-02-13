---
title: Fluent Python 笔记 第 11 章 接口：从协议到抽象基类
tags: Python
pageview: true
---

重点是说明对 Python 而言尤为重要的两个细节:
- 子类化内置类型的缺点
- 多重继承和方法解析顺序

## 12.1 子类化内置类型很麻烦
内置类型(使用 C 语言编写)不会调用用户定义的类覆盖的特殊方法。

不要子类化内置类型，用户自己定义的类应 该继承 collections 模块(http://docs.python.org/3/library/collections.html)中的类，例如 UserDict、UserList 和 UserString，这些类做了特殊设计，因此易于扩展。

## 12.2 多重继承和方法解析顺序
两种调用方法：
```
d.pong()
pong: <diamond.D object at 0x10066c278>
C.pong(d)

A.ping(self)  # 类里面访问
```
Python 能区分 `d.pong()` 调用的是哪个方法，是因为 Python 会按照特定的顺序遍历继承图。 这个顺序叫方法解析顺序(Method Resolution Order，MRO)。类都有一个名为 `__mro__` 的 属性，它的值是一个元组，按照方法解析顺序列出各个超类，从当前类一直向上，直到 `object` 类。

方法解析顺序不仅考虑继承图，还考虑子类声明中列出超类的顺序。方法解析顺序使用 C3 算法计算。

## 12.4 处理多重继承
使用多重继承时，一定要明确一开始为什么创建子类。
