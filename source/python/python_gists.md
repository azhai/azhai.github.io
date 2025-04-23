---
date:       2016-03-24
title:      Gist上一些Python的算法题
slug:       python-gists
author:     ryan
tags:       [python, gist]
weight:     660324
---

## 判断是否合法的罗马数字

```python
# -*- coding: utf-8 -*-
# is_roman_number.py
"""
VIII  8
XDIX  99
"""

numerals = 'IVXLCDM'
numdict = dict(zip(numerals, range(len(numerals))))

def is_roman_number(number):
    m = ''
    repeat = 0
    discard = False
    ridge = 6
    for n in number:
        if n not in numerals:
            return False
        if m==n:
            repeat += 1
            if repeat > 2:
                return False
            elif numdict[n] % 2 == 1:
                return False
        else:
            repeat = 0
        if m:
            near = numdict[n] - numdict[m]
            if near > 2:
                return False
            elif near > 0:
                if numdict[n] > ridge:
                    return False
                else:
                    ridge = numdict[n]
        m = n
    return True

if __name__ == '__main__':
    number = raw_input()
    print is_roman_number(number)
```

## 给一组未排序的自然数，求其所有升序子序列的数量

```python
# -*- coding: utf-8 -*-
# count_sublists.py
"""
给一组未排序的自然数，求其所有升序子序列的数量
Ex1:
给定5个数 3 1 2 5 4
结果为 8
[3],[1],[1,2],[1,2,5],[2],[2,5],[5],[4]
Ex2:
给定20个数 1 5 4 3 7 8 9 10 2 19 17 11 12 13 14 15 20 16 6 18
结果为 48
[1],[1,5],[1,5,4,3,7],[5],[4],[3],[3,7],[3,7,8],[3,7,8,9],[3,7,8,9,10],
[7],[7,8],[7,8,9],[7,8,9,10],[8],[8,9],[8,9,10],[9],[9,10],[10],
[2],[19],[17],[11],[11,12],[11,12,13],[11,12,13,14],[11,12,13,14,15],[11,12,13,14,15,20],[12],
[12,13],[12,13,14],[12,13,14,15],[12,13,14,15,20],[13],[13,14],[13,14,15],[13,14,15,20],[14],[14,15],
[14,15,20],[15],[15,20],[20],[16],[6],[6,18],[18]
"""

import math

#求组合的值，小于2时返回0
comb2 = lambda n: math.factorial(n)/math.factorial(n-2)/2 if n >= 2 else 0
#计算满足条件子序列数量
count_sub = lambda sub: comb2(len(sub))
cacl_subs = lambda subs: sum(map(count_sub, subs.values()))
#得到排序后的原索引
sort_key = lambda x: int(x[1])
sort_indexes = lambda arr: [i for i,_ in sorted(enumerate(arr), key=sort_key)]

#获得（索引）升序子序列
def get_subs(idxes):
    pointers = {}
    subs = {}
    for i in idxes:
        if pointers.has_key(i-1):
            p = pointers[i] = pointers[i - 1]
            subs[p].append(i)
        else:
            p = pointers[i] = i
            subs[p] = [i]
    return subs


if __name__ == '__main__':
    #> 20
    #> 1 5 4 3 7 8 9 10 2 19 17 11 12 13 14 15 20 16 6 18
    n = input()
    arr = raw_input().split()
    subs = get_subs(sort_indexes(arr))
    print  n + cacl_subs(subs)
```
