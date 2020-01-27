---
layout: post
title: python-demo集合
categories: python
tags: python
author: nsf
---

* content
{:toc}
![](https://cdn.jsdelivr.net/gh/nsf-github/tdxlj.github.io@master/_posts/image/2019-12-25-python-demo集合.gif)




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

![](https://cdn.jsdelivr.net/gh/nsf-github/tdxlj.github.io@master/_posts/image/2019-12-25-python-demo集合.gif)

# 2.json格式化

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

# 3.图片转字符

```
# -*- coding: UTF-8 -*-
from PIL import Image
from PIL import ImageDraw
from PIL import ImageFont
import numpy as np
import time


def demo(source_file, output_file=None, scale=2, sample_step=3, n=6):
    start_time = int(time.time())

    old_img = Image.open(source_file)
    pix = old_img.load()
    width = old_img.size[0]
    height = old_img.size[1]
    print("width:%d,height:%d" % (width, height))

    canvas = np.ndarray((height * scale, width * scale, 3), np.uint8)
    canvas[:, :, :] = 255
    new_image = Image.fromarray(canvas)
    draw = ImageDraw.Draw(new_image)

    font = ImageFont.truetype('msyhbd.ttc',12)
    char_table = list(u'喵主人')

    pix_count = 0
    table_len = len(char_table)
    for y in range(0, height, n):
        for x in range(0, width, n):
            if x % sample_step == 0 and y % sample_step == 0:
                draw.text((x * scale, y * scale), char_table[pix_count % table_len], pix[x, y], font)
                pix_count += 1

    if output_file is not None:
        new_image.save(output_file)

    print("used time : %d second,pix_count : %d" % ((int(time.time()) - start_time), pix_count))
    print(pix_count)


demo("1.jpg", "output.jpg")
```

原图和结果如下图：

![](https://cdn.jsdelivr.net/gh/nsf-github/tdxlj.github.io@master/_posts/image/2020-01-27-python-demo集合1.jpg)

![](https://cdn.jsdelivr.net/gh/nsf-github/tdxlj.github.io@master/_posts/image/2020-01-27-python-demo集合2.jpg)

除此之外，学习了Windows下字体方面的知识，字体位置为’C:\Windows\Fonts‘，代码中如果要替换字体，可以点击具体字体查看详情，找到对应的字体文件进行替换即可。