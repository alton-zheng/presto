# 1.2 Presto概念

- [总览](#概览)
- [服务器类型](#服务器类型)
- [数据源](#数据源)
- [查询执行模型](#查询执行模型)



## 总览

要了解Presto，你必须首先了解 `Presto` 文档中使用的术语和概念。

尽管很容易理解语句和查询，但作为最终用户，你应该熟悉 `State` 和 `Split` 等概念，以充分利用Presto来执行有效查询。作为Presto管理员或Presto贡献者，你应该了解Presto的 `State` 概念如何映射到 `Task` ，以及 `Task` 如何包含一组处理数据的 `Driver`。

本节为贯穿 Presto中引用的核心概念提供了一个可靠的定义，这些部分从最一般的概念到最具体的概念进行了排序。



## 服务器类型

Presto服务器有两种类型：`Coordinator` 和 `Worker`。以下部分说明了两者之间的区别。



### Coordinator

`Presto Coordinator` 是负责解析语句，查询计划和管理Presto `Worker` 节点的服务器。它是Presto安装的“brain”，也是 `Client` 连接以提交要执行的语句的节点。每个Presto安装必须在一个或多个Presto `Worker` 旁边配备一个Presto `Coordinator` 。出于开发或测试目的，可以将Presto的单个实例配置为同时执行这两个角色。

`Coordinator` 跟踪每个 `Worker` 上的活动并协调查询请求的执行。`Coordinator` 创建涉及一系列 `Stage` 的查询逻辑模型，然后将其转换为在Presto `Worker` 群集上运行的一系列关联 `task`。

`Coordinator` 使用 `REST API` 与 `Worker` 和 `Client` 进行通信。



### Worker

`Presto worker` 是Presto安装中的服务器，负责执行 `Task` 和处理数据。辅助节点从 `Connector` 获取数据并相互交换中间数据。`Coordinator` 负责从 `Worker` 那里获取结果，并将最终结果返回给 `Client` 。

当 `Presto worker` 进程启动时，它将自己通知给 `Coordinator` 中的发现服务器，这使Presto `Coordinator` 可以使用它来执行任务。

`Worker` 使用  `REST API` 与其他 `Worker` 和Presto `Coordinator` 进行通信。



## 数据源

在整个文档中，你将阅读诸如 `Connector` ， `Catalog` ， `Schema` 和 `Table` 之类的术语。这些基本概念涵盖了Presto特定数据源的模型，并将在下面部分一一介绍。

### Connector

 `Connector` 使Presto 适配数据源，例如Hive或关系数据库。你可以像考虑数据库 `driver` 一样来考虑 `Connector` 。它是Presto [SPI](connector/spi-overview.md) 的一种实现， 它允许Presto使用标准API与资源进行交互。

Presto包含几个内置 `Connector` ：用于 [JMX](connector/jmx.md)的  `Connector` ，可访问内置系统 `Table` 的[System](connector/system.md) `Connector` ，[Hive](connector/hive.md) `Connector` 和旨在提供TPC-H基准数据的 [TPCH](connector/tpch.md) `Connector` 。许多第三方开发人员(社区贡献者)都贡献了 `Connector` ，因此Presto可以访问各种数据源中的数据。

每个 `Catalog` 都与特定的 `Connector` 关联。如果检查 `Catalog` 配置文件，你将看到每个 `Catalog` 都包含一个必填属性`connector.name`， `Catalog` 管理器将其用于为给定 `Catalog` 创建 `Connector` 。可能有多个 `Catalog` 使用同一 `Connector` 来访问相似数据库的两个不同实例。例如，如果你有两个Hive群集，则可以在单个Presto群集中配置两个都使用Hive `Connector` 的 `Catalog` ，从而允许你从两个Hive群集中查询数据（即使在同一SQL查询中）。



### Catalog

Presto `Catalog` 包含 `Schema` ，并通过 `Connector` 引用数据源。例如，你可以配置JMX `Catalog` 以通过JMX `Connector` 提供对JMX信息的访问。在Presto中运行SQL语句时，将针对一个或多个 `Catalog` 运行它。 `Catalog` 的其他示例包括连接到Hive数据源的Hive `Catalog` 。

在Presto中寻址 `Table` 时，标准 `Table` 名始终植根于 `Catalog` 中。例如，一个 `fully-qualified` 的 `Table` 名称`hive.test_data.test`将引用hive `Catalog`中的 test_data `Schema` 中的 test `表`（ `catalogName.schemaName.tableName`）

 `Catalog` 是在存储在 Presto 配置目录中属性文件中定义的。



### Schema

`Schema` 是组织 `Table` 的一种方式(与 `Oracle` 数据库中的 `Schema` 定义类似)。 `Catalog` 和 `Schema` 共同定义了一组可以查询的 `Table` 。当使用Presto访问Hive或关系数据库（例如MySQL）时， `Schema` 会转换为目标数据库中的相同概念。其他类型的 `Connector` 可能选择以对基础数据源有意义的方式将 `Table` 组织到 `Schema` 中。

### Table

 `Table` 是一组无序的行，这些行被组织成具有类型的命名列。这与任何关系数据库中的相同。从源数据到 `Table` 的映射由 `Connector` 定义。



## 查询执行模型

Presto执行SQL语句，并将这些语句转换为在 `Coordinator` 和 `Worker` 的分布式群集中执行的查询。

### Statement

Presto执行与ANSI兼容的SQL语句。当Presto文档引用一个语句时，它就是指ANSI SQL标准中定义的语句，该语句由子句， 表达式和谓词组成。

一些读者可能会对为什么本节列出 `Statement` 和 `Query` 的单独概念感到好奇。这是必需的，因为在Presto中，语句仅引用SQL语句的文本表示形式。执行语句后，Presto会创建一个查询以及一个查询计划，然后将其分配给一系列`Presto Worker`。

### Query

当Presto解析一条语句时，它将转换为查询并创建一个分布式查询计划，然后将其实现为在Presto worker上运行的一系列相互关联的 `Stage`。当你在Presto中检索有关查询的信息时，你将收到与响应语句而产生结果集有关的每个组件的快照。

语句和查询之间的区别很简单。可以将语句视为传递给Presto的SQL文本，而查询则引用实例化以执行该语句的配置和组件。查询包含 `Stage`，`Task`，`Split`， `Connector` 以及其他组件和数据源，它们协同工作以产生结果。

### Stage

当Presto执行查询时，它通过将执行分为多个 `Stage` 层次结构来执行。例如，如果Presto需要需要聚合存储在Hive中的10亿行数据，它可以通过创建一个根`Stage` 来聚合其他几个 `Stage` 的输出来实现，所有这些 `Stage` 都被设计用来实现分布式查询计划的不同部分。

组成查询的 `Stage` 层次结构类似于树。每个查询都有一个根 `Stage`，负责聚合其他 `Stage` 的输出。 `Stage` 是 `Coordinator` 用来对分布式查询计划建模的部分，但是 `Stage` 本身并不运行在 `Presto workers` 上。

### Task

如前一节所述，`Stage` 对分布式查询计划的特定部分进行建模，但是 `Stage` 本身不会在 `Presto worker` 上执行。要了解阶段是如何执行的，你需要了解 `Stage` 是作为在 `Presto worker` 网络上分布的一系列任务实现的。

`Task` 是 `Presto` 体系结构中的“work horse”，因为分布式查询计划被分解为一系列 `Stage`，然后转变为 `Task`，然后对这些任务进行执行或过程拆分。Presto任务具有输入和输出，就像一个 `Stage` 可以通过一系列任务并行执行一样，`Task` 也可以由一系列 `Driver` 并行执行。

### Split

任务按 `Split` 操作，拆分是较大数据集的一部分。分布式查询计划的最低级别的阶段通过 `Connector` 的 `Split` 来检索数据，而分布式查询计划的较高级别的中间 `Stage` 则从其他 `Stage` 检索数据。

当Presto计划查询时， `Coordinator` 将查询 `Connector` 以获取 `Table` 中所有可用 Split 的列表。 `Coordinator` 跟踪哪些机器正在运行哪些任务，以及哪些任务正在处理哪些 `Split`。



### Driver

`Task` 包含一个或多个并行 `Driver`。`Driver` 根据数据进行操作，并组合 `operator` 以产生 `output`，然后将 `output` 汇总到一个任务中，然后在另一个 `stage` 将其交付给另一个 `task`。 `Driver` 是一系列 `Operator` 实例，或者你可以将 `Driver` 视为内存中一组物理 `Operator`。它是Presto `Schema` 中最低的并行度。`Driver` 具有一个 `Input` 和一个 `Output` 。



### Operator

`Operator` 使用，转换和产生数据。例如， `Table` 扫描从 `Connector` 获取数据并生成可被其他 `Operator` 使用的数据，而 `filter operator` 通过在输入数据上应用谓词(where)来消费数据并生成子集。



### Exchange

在Presto节点之间交换查询不同 `Stage` 的传输数据。任务将数据产生到 `output buffer`  中，并使用 `exchange client` 使用其他 `Task` 中的数据。