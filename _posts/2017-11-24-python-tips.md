---
title: Python Tips
date: 2017-11-24 19:06:58
tags: 
    - Python
categories:
    - Python
---

# Python Tips

## Python 的函数参数传递

Python中有可变对象（比如列表List）和不可变对象（比如字符串），所以在参数传递时分为两种情况：
对于不可变对象作为函数参数，相当于C系语言的值传递；
对于可变对象作为函数参数，相当于C系语言的引用传递。

```python
num = {1:2}
n = 2
def change(num, n):
    num[1] = 1
    n = 1

print num
print n
```
如例子中的参数，num会被修改，而n则不会被值成1
假如需要对可变对象进行计算，不修改原有值，可以使用copy方法，如果参数内容仍然是可变参数时，则需要使用deepcopy方法

```python
import copy

num = {1:[2,3]}

def change(num):
    num_copy = num
    num_copy[1] = [2,2]

def change_copy(num):
    num_copy = copy.deepcopy(num)
    num_copy[1] = [2,2]

change_copy(num)
print num
change(num)
print num


```
