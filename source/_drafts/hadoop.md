---
title: hadoop
tags:
---

# hadoop
## HDFS
* NameNode：保存整个文件系统的目录信息、文件信息及分块信息，统筹角色，和dataNode交互
* DataNode：分布在廉价的计算机上，用于存储Block块文件
<!-- more -->

# hive
* 提供了类SQL的语法，封装了底层的MapReduce过程，让有SQL基础的业务人员，也可以直接利用Hadoop进行大数据的操作
* Hive不是一个完整的数据库.Hadoop以及HDFS的设计本身约束和局限性地限制了Hive所能胜任的工作.其中最大的限制就是Hive不支持记录级别的更新\/插入\/删除操作.但是用户可以通过查询生成新表或者将查询结果导入到文件中.

# Sqoop
* Apache Sqoop（SQL-to-Hadoop） 项目旨在协助 RDBMS 与 Hadoop 之间进行高效的数据交流


# HBASE
* 大：一个表可以有上亿行，上百万列
* 面向列：面向列(族)的存储和权限控制，列(族)独立检索。

