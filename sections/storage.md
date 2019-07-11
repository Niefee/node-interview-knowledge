# [存储](/sections/storage.md)

* [`[Point]` Mysql](/sections/storage.md#mysql)
* [`[Point]` Mongodb](/sections/storage.md#mongodb)
* [`[Point]` Replication](/sections/storage.md#replication)
* [`[Point]` 数据一致性](/sections/storage.md#数据一致性)
* [`[Point]` 缓存](/sections/storage.md#缓存)

## mysql

结构化查询语言(Structured Query Language)简称`SQL`。
MySQL是一个关系型数据库管理系统，使用`SQL`语言是访问数据库。

> MySQL基础入门知识：https://note.niefee.com/%E5%90%8E%E7%AB%AF%E5%BC%80%E5%8F%91/MySQL%E7%AC%94%E8%AE%B0%E5%A4%A7%E5%85%A8.html

## mongodb

MongoDB是一个介于关系数据库和非关系数据库之间的产品。支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。

> 基础使用：https://note.niefee.com/%E5%90%8E%E7%AB%AF%E5%BC%80%E5%8F%91/MongoDB.html

> 官方文档：https://docs.mongodb.com/manual/

## replication

建议阅读：https://elemefe.github.io/node-interview/#/sections/zh-cn/storage?id=replication

## 数据一致性

建议阅读：https://elemefe.github.io/node-interview/#/sections/zh-cn/storage?id=%e6%95%b0%e6%8d%ae%e4%b8%80%e8%87%b4%e6%80%a7

## 缓存

引入缓存可以提高性能，但是数据会存在两份，一份在数据库中，一份在缓存中，如果更新其中任何一份会引起数据的不一致，数据的完整性被破坏了，因此，同步数据库和缓存的这两份数据就非常重要。

> 简单介绍：https://www.cnblogs.com/hadley/p/9557596.html

常用redis/memcache作数据库的缓存，相关分析可参考：https://www.zhihu.com/question/27738066