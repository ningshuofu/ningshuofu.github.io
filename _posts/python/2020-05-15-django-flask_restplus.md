---
layout: post
title: flask_restplus
categories: python
tags: flask python 框架
author: nsf
---

* content
{:toc}

# flask_restplus



[官方文档]: https://flask-restplus.readthedocs.io/

以下内容为本人未在官方文档找到的一些内容补充

## 1.form验证部分

简单介绍

```
parser = RequestParser() //分模块路由的话通过Namespace.parser()获取
parser.add_argument('foo', type=int, required=True) //字段验证
parser.parse_args() //获取验证后的请求参数
```

请求加入@api.expect(parser）装饰器实现验证

### 1.1.自定义验证字段类型

上面的例子字段为'foo'，类型为int，如果要支持复杂的字段（一般为list和dict），list和dict只能验证参数类型是否匹配，无法验证深层次参数，所以需要自己实现一个验证函数（做法和django类似），以下是一个demo

```
parser.add_argument('xigua', required=True, type=xigua_d)
def xigua_d(value, name):
    if not isinstance(value, dict):
        raise ValueError(name + '参数类型错误，必须是dict类型')
    return {'apple': value.get('apple')}
```



