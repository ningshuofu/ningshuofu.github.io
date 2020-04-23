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
3.forms下的字段都继承Field，所以写自定义的Field应该参考源码，比如forms.CharField
4.验证步骤如下：
    调用form.is_valid()
    调用self.errors
    调用self.full_clean()
    调用self._clean_fields()、self._clean_form()、self._post_clean()
    其中_clean_fields()会遍历自定义form下所有field，然后调用field.clean(value, initial)去做field字段验证
    其中field.clean()调用value = self.to_python(value)、self.validate(value)、self.run_validators(value)来做验证，to_python一般做格式转换、validate做具体逻辑验证、run_validators依次调用self.validators中的由开发者自定义的validator（不推荐重载）
5.关键源码如下
form.is_valid()

class BaseForm:
	...
    def is_valid(self):
    	return self.is_bound and not self.errors
    ...
    @property
    def errors(self):
        if self._errors is None:
            self.full_clean()
        return self._errors
    ...
	def full_clean(self):
        ...
        self._clean_fields()
        self._clean_form()
        self._post_clean()
    ...
    def _clean_fields(self):
        for name, field in self.fields.items():
            ...
            try:
                if isinstance(field, FileField):
                    initial = self.get_initial_for_field(field, name)
                    value = field.clean(value, initial)
                else:
                    value = field.clean(value)
                self.cleaned_data[name] = value
                if hasattr(self, 'clean_%s' % name):
                    value = getattr(self, 'clean_%s' % name)()
                    self.cleaned_data[name] = value
            except ValidationError as e:
                self.add_error(name, e)

class Field:
	...
    def clean(self, value):
        value = self.to_python(value)
        self.validate(value)
        self.run_validators(value)
        return value
                
```



