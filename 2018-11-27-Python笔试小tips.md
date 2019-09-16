# 工具与环境：
我个人是使用VSCode来写，非常好用。
代码补全，代码提示，doc，代码跳转等IntelliSense功能和code lint只要安上MS官方的Python插件就可以使用，非常省事。
Debug也非常方便。

对VSCode学习有兴趣的可以看看这个视频：`https://www.youtube.com/watch?v=6YLMWU-5H9o`。（YouTube需要翻墙）

# 小tips
- 灵活使用list, tuple, dict, set内置数据结构。
- 读入输入的一行数，以list储存：
```python
>>> import sys
>>> list(map(int, sys.stdin.readline().strip().split()))
```
- 每次跑程序都要手动输入好麻烦，怎么办？
```
解决：
将输入写到一个文件中如："input.txt"
测试及Debug时：
> input_stream = open(file) 
提交时再改为：
> input_stream = sys.stdin 
这样就可以将输入写入一个文件中，不必每次都重新输入，浪费时间。
```
- 非常有用的built-in函数：map, reduce, filter, zip等
- 非常有用的库：sys, itertools, collection, functools, operator等，可以看看文档。
- collection: 
```
# collections.Counter lets you find the most common
# elements in an iterable:

>>> import collections
>>> c = collections.Counter('helloworld')

>>> c
Counter({'l': 3, 'o': 2, 'e': 1, 'd': 1, 'h': 1, 'r': 1, 'w': 1})

>>> c.most_common(3)
[('l', 3), ('o', 2), ('e', 1)]
```
- itertools:提供了一些现成的函数，用来产生一些便于使用的iterator。
```
In [1]: import itertools

In [2]: itertools.repeat('X', 3)
Out[2]: repeat('X', 3)

In [3]: list(_)
Out[3]: ['X', 'X', 'X']

In [4]: itertools.permutations('ABC')   # 排列
Out[4]: <itertools.permutations at 0x47bec30>

In [5]: list(_)
Out[5]:
[('A', 'B', 'C'),
 ('A', 'C', 'B'),
 ('B', 'A', 'C'),
 ('B', 'C', 'A'),
 ('C', 'A', 'B'),
 ('C', 'B', 'A')]

In [7]: itertools.combinations('ABC', 2)    # 组合
Out[7]: <itertools.combinations at 0x4cba930>

In [8]: list(_)
Out[8]: [('A', 'B'), ('A', 'C'), ('B', 'C')]
```
