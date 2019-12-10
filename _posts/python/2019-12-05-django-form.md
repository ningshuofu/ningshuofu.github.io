---
layout: post
title: django-form
categories: python
tags: django python 框架
author: nsf
---

* content
{:toc}

# django form

## 工作流程

```
1.若使用装饰器装饰接口，则先进入装饰器逻辑部分
2.若有form验证,需要注意验证细节
    一般验证代码类似form = form_class(form),form_class为自定义的form
    form为一般为传入的参数
3.forms下的class都继承Field，所以写自定义的Field应该参考源码，比如forms.CharField
```



