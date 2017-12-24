---
title: Pythonic
date: 2017-12-09 21:45:23
tags:
- python
categories: 
- python
---

## 交换两个数字
其他语言:
```c
t = a
a = b
b = t
```
python:
```python
a, b = b, a
```
## 列表推导
列表推导是C、C++、Java里面没有的语法，但是，是Python里面使用非常广泛，是特别推荐的用法。
与列表推导对应的，还有集合推导和字典推导。我们来演示一下。
+ 列表：30~40 所有偶数的平方
```python
[ i*i for i in range(30, 41) if i% 2 == 0 ]
```
+ 集合：1~20所有奇数的平方的集合
```python
{ i*i for i in range(1, 21) if i % 2 != 0 }
```
+ 字典：30~40 所有奇数的平方
```python
{ i:i*i for i in range(30, 40) if i% 2 != 0 }
```

再举两个实用的例子：
+ 当前用户home目录下所有的文件列表
```python
[ item for item in os.listdir(os.path.expanduser('~')) if os.path.isfile(item) ]
```
+ 当前用户home目录下所有的目录列表
```python
[ item for item in os.listdir(os.path.expanduser('~')) if os.path.isdir(item) ]
```
+ 当前用户home目录下所有目录的目录名到绝对路径之间的字典
```python
{ item: os.path.realpath(item) for item in os.listdir(os.path.expanduser('~')) if os.path.isdir(item) }
```

## 上线文管理器
我们要打开文件进行处理，在处理文件过程中可能会出错，但是，我们需要在处理文件出错的情况下，也顺利关闭文件。
Java风格/C++风格的Python代码：
```python
myfile= open(r'C:\misc\data.txt')
try:
    for line in myfile:
        ...use line here...
finally:
    myfile.close()
```
Pythonic的代码：
```python
with open(r'C:\misc\data.txt') as myfile:
    for line in myfile:
        ...use line here...
```
这里要说的是，上下文管理器是Python里面比较推荐的方式，如果用try...finally而不用with，就会被认为不够Pythonic。此外，上线文管理器还可以应用于锁和其他很多类似必须需要关闭的地方。
