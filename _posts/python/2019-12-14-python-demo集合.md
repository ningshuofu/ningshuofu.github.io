---
layout: post
title: python
categories: python
tags: python
author: nsf
---

* content
{:toc}
![](https://cdn.jsdelivr.net/gh/nsf-github/tdxlj.github.io@master/_posts/python/pro.gif)




# 1.输出彩色进度条

```
import random
from time import sleep


def color(code=''):
    return f'\033[{code}m'


def pro():
    for i in range(1, 101):
        left = i * 30 // 100
        right = 30 - left
        print(color(), '\r[', color('32;1;0'), '#' * left, ' ' * right, color(), '] ', color('33;1;0'), str(i), '%',
              sep='', end='', flush=True)
        sleep(0.01)
        
        
pro()
```

![](https://cdn.jsdelivr.net/gh/nsf-github/tdxlj.github.io@master/_posts/python/pro.gif)

## 2.json格式化

```
import json


def format_json():
    my_mapping = {'a': 23, 'b': 42, 'c': 0xc0ffee}
    print(json.dumps(my_mapping, indent=4, sort_keys=True))


format_json()
```

> {
> ​    "a": 23,
> ​    "b": 42,
> ​    "c": 12648430
> }	