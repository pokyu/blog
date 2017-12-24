---
title: Python基础
date: 2017-12-16 22:00:21
tags:
- python
categories: 
- python
---

## 前言
&emsp;&emsp;本文所有代码基于python 2版本编写，如有涉及python 3会有特别说明。

## Python包
### 包定义
&emsp;&emsp;为了组织好模块，会将多个模块分为包。Python 处理包也是相当方便的。简单来说，包就是文件夹，但该文件夹下必须存在 \_\_init__.py 文件。

&emsp;&emsp;最简单的情况下，只需要一个空的 __init__.py 文件即可。当然它也可以执行包的初始化代码，或者定义稍后介绍的 __all__ 变量。当然包底下也能包含包，这和文件夹一样，还是比较好理解的。

### 导入包

&emsp;&emsp;包的导入仍使用 import 、 from ... import 语句，使用 “圆点模块名” 的结构化模块命名空间。 下面来看一个包的例子来了解下具体的运作。（官方文档中的例子）

&emsp;&emsp;假设你现在想要设计一个模块集（一个“包”）来统一处理声音文件和声音数据。存在几种不同的声音格式（通常由它们的扩展名来标识，例如： .wav, .aiff, .au ）于是，为了在不同类型的文件格式之间转换，你需要维护一个不断增长的包集合。可能你还想要对声音数据做很多不同的操作（例如混音，添加回声，应用平衡 功能，创建一个人造效果）所以你要加入一个无限流模块来执行这些操作。你的包可能会是这个样子（通过分级的文件体系来进行分组）：

&emsp;&emsp;用户可以每次只导入包里的特定模块，例如： import sound.efforts.echo   这样就导入了 sound.effects.echo 子模块。它必须通过完整的名称来引用：
```
sound.effects.echo.echofilter(input, output, delay=0.7, atten=4) 
```
导入包时有一个可以选择的方式： from sound.effects import echo   这样就加载了 echo 子模块，并且使得它在没有包前缀的情况下也可以使用，所以它可以如下方式调用：
```
echo.echofilter(input, output, delay=0.7, atten=4) 
```
还有另一种变体用于直接导入函数或变量： from sound.effects.echo import echofilter   这样就又一次加载了 echo 字模块，但这样就可以直接调用它的 echofilter() 函数：
```
echo.echofilter(input, output, delay=0.7, atten=4) 
```
需要注意的是  from package import item    方式导入包时，这个子项(item)既可以是子包也可以是其他命名，如函数、类、变量等。若无，会引发ImportError异常。

而用类似 import item.subitem.subsubitem 这样的语法时，这些子项必须是包，最后的子项可以是包或模块，但不能是类、函数、变量等。

### 从 * 导入包

import * 这样的语句理论上是希望文件系统找出包中所有的子模块，然后导入它们。这可能会花长时间，并出现边界效应等。Python 解决方案是提供一个明确的包索引。

这个索引由\_\_init\_\_.py  定义 \_\_all\_\_ 变量，该变量为一列表，如上例 sound/effects 下的 \_\_init\_\_.py 中，可定义  \_\_all\_\_ = ["echo","surround","reverse"] 

这意味着，  from sound.effects import * 会从对应的包中导入以上三个子模块； 尽管提供 import * 的方法，仍不建议在生产代码中使用这种写法。

### 包内引用

如果是子包内的引用，可以按相对位置引入子模块 以 echo 模块为例，可以引用如下：
```
1 from . import reverse              # 同级目录 导入 reverse
2 from .. import frormats            # 上级目录 导入 frormats
3 from ..filters import equalizer    # 上级目录的filters模块下 导入 equalizer
```
### 多重目录包搜索

包支持一个更为特殊的特性， \_\_path\_\_在包的 \_\_init\_\_.py 文件代码执行前，该变量初始化一个目录名列表。作用于子包和模块的搜索功能。
该功能可以用于扩展包中的模块集，不过不常用。