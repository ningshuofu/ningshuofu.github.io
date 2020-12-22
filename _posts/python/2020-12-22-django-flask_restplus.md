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



### 1.2.参数介绍

```
class Argument(object):
    '''
    :param name: Either a name or a list of option strings, e.g. foo or -f, --foo.
    :param default: The value produced if the argument is absent from the request.
    :param dest: The name of the attribute to be added to the object
        returned by :meth:`~reqparse.RequestParser.parse_args()`.
    :param bool required: Whether or not the argument may be omitted (optionals only).
    :param string action: The basic type of action to be taken when this argument
        is encountered in the request. Valid options are "store" and "append".
    :param bool ignore: Whether to ignore cases where the argument fails type conversion
    :param type: The type to which the request argument should be converted.
        If a type raises an exception, the message in the error will be returned in the response.
        Defaults to :class:`unicode` in python2 and :class:`str` in python3.
    :param location: The attributes of the :class:`flask.Request` object
        to source the arguments from (ex: headers, args, etc.), can be an
        iterator. The last item listed takes precedence in the result set.
    :param choices: A container of the allowable values for the argument.
    :param help: A brief description of the argument, returned in the
        response when the argument is invalid. May optionally contain
        an "{error_msg}" interpolation token, which will be replaced with
        the text of the error raised by the type converter.
    :param bool case_sensitive: Whether argument values in the request are
        case sensitive or not (this will convert all values to lowercase)
    :param bool store_missing: Whether the arguments default value should
        be stored if the argument is missing from the request.
    :param bool trim: If enabled, trims whitespace around the argument.
    :param bool nullable: If enabled, allows null value in argument.
    '''
```

其中需要注意的是default参数，在官方文档未找到提及该参数的地方，导致一些业务在必填参数时习惯性使用required=True，细想不太妥当，比如一个有翻页功能的接口，需要每页数据数量这个参数，或许使用required=False,default=10，这种方式可能比较合适。