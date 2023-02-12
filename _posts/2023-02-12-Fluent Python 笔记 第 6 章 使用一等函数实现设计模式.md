---
title: Fluent Python 笔记 第 6 章 使用一等函数实现设计模式
tags: Python
pageview: true
---

虽然设计模式与语言无关，但这并不意味着每一个模式都能在每一门语言中使用。1996 年，Peter Norvig 在题为“Design Patterns in Dynamic Languages”(http://norvig.com/design- patterns/)的演讲中指出，Gamma 等人合著的《设计模式:可复用面向对象软件的基础》一 书中有 23 个模式，其中有 16 个在动态语言中“不见了，或者简化了”(参见第 9 张幻灯片)。他讨论的是 Lisp 和 Dylan，不过很多相关的动态特性在 Python 中也能找到。具体而言，Norvig 建议在有一等函数的语言中重新审视“策略”“命令”“模板方法”和“访问者”模式。通常，我们可以把这些模式中涉及的某些类的实例替换成简单的函数， 从而减少样板代码。

## 6.1 案例分析:重构“策略”模式
### 6.1.1 经典的“策略”模式
![3](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/3.png)
### 6.1.2 使用函数实现“策略”模式
就是使用函数代替各种策略。函数可以直接调用，也不需要 `Promotion` 基类。

### 6.1.3 选择最佳策略:简单的方式
```
def best_promo(order):
    """选择可用的最佳折扣
    """
    return max(promo(order) for promo in promos)
```
但是有些重复可能会导致不易察觉的缺陷:若想添加新的促销策略，要定义相应的函数，还要记得把它添加到 promos 列表中。

### 6.1.4 找出模块中的全部策略
globals()

返回一个字典，表示当前的全局符号表。这个符号表始终针对当前模块(对函数或方法 来说，是指定义它们的模块，而不是调用它们的模块)。

`inspect.getmembers` 函数用于获取对象(这里是 promotions 模块)的属性，第二个参数是可选的判断条件(一个布尔值函数)。我们使用的是 `inspect.isfunction`，只获取模块中的函数。
```
promos = [globals()[name] for name in globals() if name.endswith('_promo') and name != 'best_promo'] 
```
## 6.2 “命令”模式
![4](https://github.com/zhangchaosd/superchao/raw/master/_posts/assets/20230212/4.png)
站在一定高度上看，这里采用的方式 与“策略”模式所用的类似:把实现单方法接口的类的实例替换成可调用对象。毕竟，每个 Python 可调用对象都实现了单方法接口，这个方法就是 `__call__`。
