---
layout: post
title: django-debug-toolbar
categories: python
tags: django python 框架 工具
author: nsf
---
[TOC]

## django-debug-toolbar

一个非常方便的工具，它可以深入了解您的代码正在做什么以及花费了多少时间。特别是它可以显示您的页面生成的所有SQL查询，以及每个查询所用的时间。
第三方面板也可用于工具栏，可以（例如）报告缓存性能和模板呈现时间。

### 安装步骤

官方文档：https://django-debug-toolbar.readthedocs.io/en/latest/installation.html#getting-the-code

步骤如下：

版本：2.0

安装
> $ pip install django-debug-toolbar

配置settings
```
INSTALLED_APPS = [
    # ...
    'django.contrib.staticfiles',
    # ...
    'debug_toolbar',
]
STATIC_URL = '/static/'

配置中间件
MIDDLEWARE = [
    # ...
    'debug_toolbar.middleware.DebugToolbarMiddleware',
    # ...
]

配置ip
INTERNAL_IPS = [
    # ...
    '127.0.0.1',
    # ...
]
```

配置urls
```
from django.conf import settings
from django.conf.urls import include, url  # For django versions before 2.0
from django.urls import include, path  # For django versions from 2.0 and up

if settings.DEBUG:
    import debug_toolbar
    urlpatterns = [
        path('__debug__/', include(debug_toolbar.urls)),

        # For django versions before 2.0:
        # url(r'^__debug__/', include(debug_toolbar.urls)),

    ] + urlpatterns
```

不支持返回json的接口，测试接口时不方便使用，可以安装 django-debug-panel插件

## django-debug-panel

版本：1.11

官方文档：https://github.com/recamshak/django-debug-panel

步骤如下：

#### Install and configure 
`Django Debug Toolbar <https://github.com/django-debug-toolbar/django-debug-toolbar>`_

#### Install Django Debug Panel:

    pip install django-debug-panel

#### Add ``debug_panel`` to your ``INSTALLED_APPS`` setting:

    INSTALLED_APPS = (
        # ...
        'debug_panel',
    )

#### Replace the Django Debug Toolbar middleware with the Django Debug Panel one. Replace:

    MIDDLEWARE_CLASSES = (
        ...
        'debug_toolbar.middleware.DebugToolbarMiddleware',
        ...
    )

   with:

    MIDDLEWARE_CLASSES = (
        ...
        'debug_panel.middleware.DebugPanelMiddleware',
        ...
    )


#### (Optional) Configure your cache.
   All the debug data of a request are stored into the cache backend ``debug-panel``
   if available. Otherwise, the ``default`` backend is used, and finally if no caches are
   defined it will fallback to a local memory cache.
   You might want to configure the ``debug-panel`` cache in your ``settings``:

    CACHES = {
        'default': {
            'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
            'LOCATION': '127.0.0.1:11211',
        },
    
        # this cache backend will be used by django-debug-panel
        'debug-panel': {
            'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
            'LOCATION': '/var/tmp/debug-panel-cache',
            'OPTIONS': {
                'MAX_ENTRIES': 200
            }
        }
    }

#### Install the Chrome extension `Django Debug Panel <https://chrome.google.com/webstore/detail/django-debug-panel/nbiajhhibgfgkjegbnflpdccejocmbbn>`_

注意有时候会出错误：No module named 'django.core.urlresolvers'，这是因为django-debug-panel使用的是django1.x版本没有及时更新，将源代码中django.core.urlresolvers 包 更改为了 django.urls即可

django版本：2.2.3