---
layout: post
title: flask-uwsgi-gevent
categories: python
tags: flask python uwsgi gevent
author: nsf
---

* content
{:toc}
uwsgi下线程和协程使用，gevent和asyncio介绍




# 1.安装步骤

## uwsgi安装

> $ pip install uwsgi

配置

```
[uwsgi]
http-socket = 0.0.0.0:8040
chdir=/home/xxx/
wsgi-file =/home/xxx/run.py
plugins = python3
callable=app
processes = 4
# threads = 3000
pidfile = aiops_flask.pid
gevent = 3000
```

## gevent安装

> $ pip install gevent

配置

```
from gevent import monkey
monkey.patch_all()
from flask import Flask
from app import api

app = Flask(__name__)
api.init_app(app)

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=8040)
```

# 2.测试uwsgi协程和线程并发

使用Apache Bench做1w并发的压力测试，注意在并发量较小时看不出二者区别

测试命令(注意windows下去掉application/json两边的单引号)：

```
ab -n 1000 -c 1000 -p /home/data.txt -T 'application/json' http://127.0.0.1:8040/user_manage/user/login
```

data.txt内容：

```
{"a": "1", "b": "2", "c": 200}
```

接口(代码有所简化)：

```
# -*- coding: utf-8 -*-
import time

from flask_restplus import Resource

from app.function_interface.user_manage import api
from flask import jsonify

parser = api.parser()


@api.route('/user_manage/user/login')
class UserLogin(Resource):
    @api.expect(parser)
    @api.response(200, description=DescriptionExample.USERLOGIN, model='object')
    def post(self):
        args = parser.parse_args()
        time.sleep(0.2)
        return jsonify({})
```

## 线程

run.py记得去掉最前面两行

```
[uwsgi]
http-socket = 0.0.0.0:8040
chdir=/home/xxx/
wsgi-file =/home/xxx/run.py
plugins = python3
callable=app
processes = 4
threads = 3000
pidfile = aiops_flask.pid
```

结果：

![](https://cdn.jsdelivr.net/gh/nsf-github/tdxlj.github.io@master/_posts/image/2021-04-13-flask-uwsgi-gevent-demo2.png)

## 协程

```
[uwsgi]
http-socket = 0.0.0.0:8040
chdir=/home/xxx/
wsgi-file =/home/xxx/run.py
plugins = python3
callable=app
processes = 4
pidfile = aiops_flask.pid
gevent = 3000
```

结果：

![](https://cdn.jsdelivr.net/gh/nsf-github/tdxlj.github.io@master/_posts/image/2021-04-13-flask-uwsgi-gevent-demo1.png)

## 注意

使用gevent可能会出现意想不到的错误，原因在于gevent本身的一些限制以及monkey.patch_all()会对后续代码进行动态替换，常用的一些库可能没问题，有些库可能会出问题，代码需要进行修改。

# 3.gevent和asyncio

性能差距很小，gevent相当于自动挡，asyncio相当于手动挡。

```
# 协程 通过 async/await 语法进行声明
# 如果一个对象可以在 await 语句中使用，那么它就是 可等待 对象
# 可等待 对象有三种主要类型: 协程, 任务 和 Future
from gevent import monkey

monkey.patch_all()
import asyncio
import time
from aiohttp_requests import requests
import requests as requests1
import gevent

url = 'http://127.0.0.1:8040/user_manage/user/login'
headers = {'token': '18fed9fb16ea5b27ed8c54fd86d205c4'}
data_json = {"user_id": "1", "user_password": "daxigua", "expires_time": 604800}


async def nested():
    """
    "协程" 可用来表示两个紧密关联的概念:
    协程函数: 定义形式为 async def 的函数;
    协程对象: 调用 协程函数 所返回的对象。
    asyncio 也支持旧式的 基于生成器的 协程。
    @return:
    """
    resp = await requests.post(url, headers=headers, json=data_json)
    r = await resp.text()
    print(r)


async def main():
    task1 = asyncio.create_task(nested())
    task2 = asyncio.create_task(nested())
    await task2
    await task1


def f1():
    resp = requests1.post(url, headers=headers, json=data_json)
    print(resp.text)


def f(n):
    gevent.joinall([gevent.spawn(f1) for _ in range(n)])


t = time.time()
asyncio.run(main(), debug=True)
print(time.time() - t)
f(300)
print(time.time() - t)
```

