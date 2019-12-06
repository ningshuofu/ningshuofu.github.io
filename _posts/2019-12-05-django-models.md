---
title: django-models
categories: django
date: 2019-12-05 11:03:56
---
# django-models

## 1.objects
> 初始化自定义model时会自动创建这个对象，通过此对象才能进行数据库操作，ModelBase的构造函数__new__中调用new_class._prepare()来进行初始化，代码如下：
```
manager = Manager()
manager.auto_created = True
cls.add_to_class('objects', manager)
```

> 从上面可以看出来objects就是一个Manager对象，代码如下：
```
class Manager(BaseManager.from_queryset(QuerySet)):
    pass
```

## 2.F,Q,annotate,aggregate
#### Q
> Q()中填入表达式，需要注意的是可以用或，包含等语法
```
Q:obj = models.Book.objects.filter(Q(id=3))
```
#### F
> F:F()中填入列名，可实现比较两个列名
```
models.Book.objects.all().update(price=F("price")+20)
```
#### annotate
> 返回的是Queryset，除了应该返回的字段外，会另加上annotate中自定义的字段，字段值为annotate()中表达式得出的值。可以通过计算查询结果中每一个对象所关联的对象集合，从而得出总计值(也可以是平均值或总和)，即为查询集的每一项生成聚合。
```
annotate:obj = models.Book.objects.values("publisher__name").annotate(Sum("price"))

<QuerySet [
{'publisher__name': '广东工业出版社', 'price__sum': Decimal('190')},
{'publisher__name': '电子出版社', 'price__sum': Decimal('88')}
]
```
#### aggregate
> aggregate的中文意思是聚合, 源于SQL的聚合函数。Django的aggregate()方法作用是对一组值(比如queryset的某个字段)进行统计计算，并以字典(Dict)格式返回统计计算结果。django的aggregate方法支持的聚合操作有AVG / COUNT / MAX / MIN /SUM 等。

```
Student.objects.all().aggregate(Avg('age'))
{ 'age__avg': 12 }
```

> 聚合函数：在Django中，聚合函数都是在django.db.models模块下的，具体的聚合函数有Avg、Count、Max、Min、Sum，现在我们一一介绍这些函数的作用：

```
Avg：计算平均值，使用于与数值相关的字段，如果使用aggregate方法来执行这个函数，那么会得到一个字典，默认情况下，字典的键为field__avg，值为执行这个聚合函数所得到的值，示例代码如下
Count：计算数量，基本用法与Avg相同，在使用这个聚合函数的时候可以传递一个distinct参数用来去重
Max和Min：计算某个字段的最大值和最小值，用法与Avg一样
Sum：计算总和，用法与Avg一样
```

> Django的aggregate和annotate方法属于高级查询方法，主要用于组合查询，可以大大提升数据库查询效率。当你需要对查询集(queryset)的某些字段进行聚合操作时(比如Sum, Avg, Max)，请使用aggregate方法。如果你想要对数据集先进行分组(Group By)然后再进行某些聚合操作或排序时，请使用annotate方法。

## 3.外键
```
class Hotel(models.Model):
    name = models.CharField(max_length=20)
class HotelBranch(models.Model):
    hotel = models.ForeignKey('Hotel', models.CASCADE,'branches')
hotel = Hotel.objects.get(name='xigua')
```
> hotel有branches这个属性，通过这个属性可以查到对应的HotelBranch数据，即外键反查的一种方式。

## 4.select_related 和 prefetch_related
> 简单来说两者都是为了避免懒加载造成的多次查询
#### select_related
> select_related()当执行它的查询时它沿着外键关系查询关联的对象数据。它会生成一个复杂的查询并引起性能的消耗，但是在以后使用外键关系时将不需要数据库查询。

> select_related通过创建SQL连接并在SELECT语句中包括相关对象的字段来工作。因此，select_related在同一数据库查询中获取相关对象。然而，为了避免由于跨越“多个'关系而导致的大得多的结果集，select_related限于一对多和一对一关系。

```
select_related() 接受可变长参数，每个参数是需要获取的外键（父表的内容）的字段名，以及外键的外键的字段名、外键的外键的外键…。若要选择外键的外键需要使用两个下划线“__”来连接。
我们举个例子：
>>> persons = Person.objects.select_related("living__province").get(pk=1)
>>> persons.living.province
# 输出结果： <Province: BeiJing>

接下来看出发的SQL语句：
(0.000) SELECT `apps_person`.`id`, `apps_person`.`firstname`, `apps_person`.`lastname`, `apps_person`.`hometown_id`, `apps_person`.`living_id`, `apps_city`.`id`, `apps_city`.`name`, `apps_city`.`province_id`, `apps_province`.`id`, `apps_province`.`name` FROM `apps_person` 
INNER JOIN `apps_city` ON (`apps_person`.`living_id` = `apps_city`.`id`) 
INNER JOIN `apps_province` ON (`apps_city`.`province_id` = `apps_province`.`id`) 
WHERE `apps_person`.`id` = 1; args=(1,)
```
> persons.living和persons.living.province不会造成额外查询，而persons.hometown.province会造成额外查询，因为select_related(）中没有写hometown这个外键

#### prefetch_related
> prefetch_related()返回的也是QuerySet，它将在单个批处理中自动检索每个指定查找的对象。这具有与select_related类似的目的，两者都被设计为阻止由访问相关对象而导致的数据库查询的泛滥，但是策略是完全不同的。

> prefetch_related，另一方面，为每个关系单独查找，并在Python中“加入”。这允许它预取多对多和多对一对象。

> 下面我来们操作一下：

```
>>> persons = Person.objects.prefetch_related("visitation__province").get(pk=1)

查询SQL语句：
(0.007) SELECT (`apps_person_visitation`.`person_id`) AS `_prefetch_related_val_person_id`, `apps_city`.`id`, `apps_city`.`name`, `apps_city`.`province_id` FROM `apps_city` 
INNER JOIN `apps_person_visitation` ON (`apps_city`.`id` = `apps_person_visitation`.`city_id`) 
WHERE `apps_person_visitation`.`person_id` IN (1); args=(1,)
>>> persons.visitation
<django.db.models.fields.related_descriptors.ManyRelatedManager object at 0x0000000003FCAAC8>
```
> prefetch_related（）的操作和 select_related()大概是相同的，只是prefetch_related（）是操作多对多的表格。

> 有一个表article，有一个多对多外键tags，如果我们获取tags对象时只希望获取以字母P开头的tag对象怎么办呢？我们可以使用Prefetch方法给prefect_related方法添加条件和属性。

```
# 获取文章列表及每篇文章相关的名字以P开头的tags对象信息
Article.objects.all().prefetch_related(
    Prefetch('tags', queryset=Tag.objects.filter(name__startswith="P"))
)

# 文章列表及每篇文章的名字以P开头的tags对象信息, 放在article_p_tag列表
Article.objects.all().prefetch_related(
    Prefetch('tags', queryset=Tag.objects.filter(name__startswith="P")),
to_attr='article_p_tag'
)
```

> 总结：
> 对与单对单或单对多外键ForeignKey字段，使用select_related方法
> 对于多对多字段和反向外键关系，使用prefetch_related方法
> 两种方法均支持双下划线指定需要查询的关联对象的字段名
> 使用Prefetch方法可以给prefetch_related方法额外添加额外条件和属性。