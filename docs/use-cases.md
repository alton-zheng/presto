# 1.1 用户场景

这一章将会细致的分析一下Presto，从而可以让管理员和终端用户深入的了解什么是Presto。

## Presto不是什么

虽然Presto一直被一些个人或者团体称为 *数据库* ，但是Presto并不是数据库。

千万不要以为Presto可以解析SQL，那么Presto就是一个标准的数据库。Presto并不是传统意义上的数据库。Presto并不是MySQL、PostgreSQL或者Oracle的代替品。Presto并不能用来处理在线事务。其实很多其他的数据库产品也是被用来设计为数据仓库或者数据分析工具，但是也不能处理在线事务。

## Presto是什么

Presto通过使用分布式查询，可以快速高效的完成海量数据的查询。如果你需要处理TB或者PB级别的数据，那么你可能更希望借助于Hadoop和HDFS来完成这些数据的处理。作为Hive和Pig（Hive和Pig都是通过MapReduce的管道流来完成HDFS数据的查询）的替代者，Presto不仅可以访问HDFS，也可以操作不同的数据源，包括：RDBMS和其他的数据源（例如：Cassandra）。

Presto被设计为数据仓库和数据分析产品：数据分析、大规模数据聚集和生成报表。这些工作经常通常被认为是线上分析处理操作。

## 谁使用Presto？

Presto是FaceBook开源的一个开源项目。Presto在FaceBook诞生，并且由FaceBook内部工程师和开源社区的工程师公共维护和改进。