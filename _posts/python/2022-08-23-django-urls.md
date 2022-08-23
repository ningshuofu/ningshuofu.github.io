---
layout: post
title: django-urls
categories: python
tags: django python 框架
author: nsf
---

* content
{:toc}
django获取所有接口api（包含url和http方法）



# django-urls

## 1.urls

用import_string载入setting中的ROOT_URLCONF可以获取所有urlpatterns，然后递归遍历urlpatterns，一直遍历到URLPattern即可获取所有url

```
from django.utils.moudle_loading import import_string
from xxx import setting

md = import_string(setting.ROOT_URLCONF)
```

## 2.url method

**缺少的api：**获取到项目所有url后，问题是没有url下支持的http方法

**思考：**解决方案是递归调用所有url下的所有http方法，获取到支持的http方法

**存在的问题：**实际项目中无法调用实际的接口方法，因为可能会对系统造成不可控影响，所以需要对所有接口进行拦截，拦截的位置比较讲究，需要在django框架**调用实际接口实现之前**，django框架**已经加载了实际接口之后**调用。

**源码：**通过查看源码，拦截的位置为rest_framework/view.py文件内APIView类中的dispatch方法，具体位置在

```
handler = self.http_method_not_allowed
```

这一行代码之后，这之前已经将实际实现的接口方法加载到handler中，下一行为

```
response = handler(request, *args, **kwargs)
```

这一行即为调用实际的接口方法，所以可以在这之前根据具体需求来拦截所有请求，大致逻辑如下：

```
# 如果已经实现了接口方法
if getattr(self, request.method.lower(), None) is not None:
    return Response({'code': 0})
# 如果没有实现接口方法
else:
    return Response({'code': 1})
```

ps：这里只说明了源码关于接口方法加载的一小段代码逻辑，如果不满足需求还得看源码其他部分