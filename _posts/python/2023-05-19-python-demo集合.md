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

![](https://cdn.jsdelivr.net/gh/nsf-github/tdxlj.github.io@master/_posts/image/2020-01-27-python-demo31.jpg)

![](https://cdn.jsdelivr.net/gh/nsf-github/tdxlj.github.io@master/_posts/image/2020-01-27-python-demo32.jpg)

除此之外，学习了Windows下字体方面的知识，字体位置为’C:\Windows\Fonts‘，代码中如果要替换字体，可以点击具体字体查看详情，找到对应的字体文件进行替换即可。

# 4.hashmap

```
class Item(object):

    def __init__(self, key, value):
        self.key = key
        self.value = value


class HashTable(object):

    def __init__(self, size):
        self.size = size
        self.table = [[] for _ in range(self.size)]

    def _hash_function(self, key):
        return key % self.size

    def set(self, key, value):
        hash_index = self._hash_function(key)
        for item in self.table[hash_index]:
            if item.key == key:
                item.value = value
                return
        self.table[hash_index].append(Item(key, value))

    def get(self, key):
        hash_index = self._hash_function(key)
        for item in self.table[hash_index]:
            if item.key == key:
                return item.value
        raise KeyError('Key not found')

    def remove(self, key):
        hash_index = self._hash_function(key)
        for index, item in enumerate(self.table[hash_index]):
            if item.key == key:
                del self.table[hash_index][index]
                return
        raise KeyError('Key not found')
```

# 5.LRU cache

```
class Node(object):
    def __init__(self, results):
        self.results = results
        self.prev = None
        self.next = None


class LinkedList(object):
    def __init__(self):
        self.head = None
        self.tail = None

    def move_to_front(self, node):
        if node is self.head:
            return
        elif node is self.tail:
            self.tail = node.prev
            self.tail.next = None
        else:
            node.prev.next = node.next
            node.next.prev = node.prev
        node.prev = None
        node.next = self.head
        self.head.prev = node
        self.head = node

    def append_to_front(self, node):
        if self.head is None:
            self.head = self.tail = node
        else:
            node.next = self.head
            self.head.prev = node
            self.head = node

    def remove_from_tail(self):
        if self.tail is None:
            return
        if self.head is self.tail:
            self.head = self.tail = None
        else:
            self.tail = self.tail.prev
            self.tail.next = None


class Cache(object):
    def __init__(self, max_size):
        self.max_size = max_size
        self.size = 0
        self.lookup = {}  # key: query, value: node
        self.linked_list = LinkedList()

    def get(self, query):
        """Get the stored query result from the cache.

        Accessing a node updates its position to the front of the LRU list.
        """
        node = self.lookup.get(query)
        if node is None:
            return None
        self.linked_list.move_to_front(node)
        return node.results

    def set(self, results, query):
        """Set the result for the given query key in the cache.

        When updating an entry, updates its position to the front of the LRU list.
        If the entry is new and the cache is at capacity, removes the oldest entry
        before the new entry is added.
        """
        node = self.lookup.get(query)
        if node is not None:
            # Key exists in cache, update the value
            node.results = results
            self.linked_list.move_to_front(node)
        else:
            # Key does not exist in cache
            if self.size == self.max_size:
                # Remove the oldest entry from the linked list and lookup
                self.lookup.pop(self.linked_list.tail.query, None)
                self.linked_list.remove_from_tail()
            else:
                self.size += 1
            # Add the new key and value
            new_node = Node(results)
            self.linked_list.append_to_front(new_node)
            self.lookup[query] = new_node
```

