---
layout: post
title: cassandra
categories: 数据库
tags: cassandra
author: nsf
---

* content
{:toc}

cassandra备忘

```
官方文档见: https://cassandra.apache.org/doc/latest/
```



# 1.cassandra操作

## 1.1 关键字

| 操作名          | 操作含义           |      | 类别              | 类别含义                             |
| --------------- | ------------------ | ---- | ----------------- | ------------------------------------ |
| create          | 创建               |      | keyspace          | 命名空间                             |
| alter           | 修改（非数据粒度） |      | table             | 表（默认不区分大小写，加双引号区分） |
| drop            | 删除               |      | primary key       | 主键                                 |
| select(json)?   | 选择               |      | index             | 索引                                 |
| truncate        | 清空               |      | replication       | 复制策略                             |
| batch           | 批处理             |      | durable_writes    | 是否使用提交日志来更新此键空间       |
| update          | 更新               |      | materialized view | 视图                                 |
| insert          | 插入               |      | cluster           | 集群                                 |
| delete          | 删除（数据）       |      | timestamp         | 设置操作的时间戳                     |
| describe        | 查看               |      | ttl               | 为插入的数据设置过期时间             |
| alter filtering | 允许过滤           |      |                   |                                      |

## 1.2 数据类型

| 数据类型      | 常量              | 描述                       |
| ------------- | ----------------- | -------------------------- |
| ascii         | strings           | 表示ASCII字符串            |
| bigint        | bigint            | 表示64位有符号长           |
| **blob**      | blobs             | 表示任意字节               |
| Boolean       | booleans          | 表示true或false            |
| **counter**   | integers          | 表示计数器列               |
| decimal       | integers, floats  | 表示变量精度十进制         |
| double        | integers          | 表示64位IEEE-754浮点       |
| float         | integers, floats  | 表示32位IEEE-754浮点       |
| inet          | strings           | 表示一个IP地址，IPv4或IPv6 |
| int           | integers          | 表示32位有符号整数         |
| text          | strings           | 表示UTF8编码的字符串       |
| **timestamp** | integers, strings | 表示时间戳                 |
| **timeuuid**  | uuids             | 表示类型1 UUID             |
| **uuid**      | uuids             | 表示任何类型UUID           |
| varchar       | strings           | 表示uTF8编码的字符串       |
| varint        | integers          | 表示任意精度整数           |

## 1.3 集合类型

| 集合 | 描述                             |
| ---- | -------------------------------- |
| list | 列表是一个或多个有序元素的集合。 |
| map  | 字典是键值对的集合。             |
| set  | 集合是一个或多个元素的集合。     |

## 1.4 批量操作

```
from cassandra.query import SimpleStatement
query = "SELECT * FROM users"  # users contains 100 rows
statement = SimpleStatement(query, fetch_size=10)
datas = session.execute(statement)
while datas.has_more_pages:
    datas.fetch_next_page()
    print(len(datas.current_rows))
```

## 1.5 分页

```
from cassandra.query import SimpleStatement
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider

auth_provider = PlainTextAuthProvider(username='cassandra', password='cassandra')
db = Cluster(["x.x.x.x"], auth_provider=auth_provider, port='9042')
db_name = 'xxx'
query = "SELECT * FROM xxx"
statement = SimpleStatement(query, fetch_size=10)
a = db.connect(db_name).execute(statement)
for user_row in a.fetch_next_page():
    print(user_row)
```

