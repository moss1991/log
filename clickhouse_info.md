# OLTP 与 OLAP

## 概念

|  | OLTP | OLAP |
| --- | --- | --- |
| 对应系统 | 业务系统、合同、订单 | 决策支持系统 |
| 定位 | 产生数据 | 使用数据 |
| 工作负载 | 增、删、改 | 查询 |
| 系统衡量指标 | 事务吞吐量 | 查询响应速度qps |
| 数据库设计 | 3NF 或 BCNF | 星型、雪花模型 |  

OLAP分析分为关系型联机分析处理（ROLAP）、多维联机分析处理（MOLAP）两种

* ROLAP: 即关系型 OLAP，通过对原始明细数据实时聚合计算的方式来进行查询。比如 Presto，Impala 之类的 OLAP 引擎。

* MOLAP: 即多维型 OLAP，通过摄入时对原始明细数据进行预聚合加工处理，然后通过预聚合数据来进行查询。比如 Kylin，Druid 之类的 OLAP 引擎。

Hybird OLAP: 混合类型 OLAP。

## 关于ClickHouse的特性

ClickHouse 擅长:

* SQL 支持: 支持大部分 SQL 功能。
* 列式存储，数据压缩: 列式存储能够更加有利于 OLAP 聚合查询，同时也能大大提高数据压缩率。
* 多核(垂直扩展)，分布式处理(水平扩展): 使用多线程和多分片并行处理。
* 实时数据摄入: 数据可以实时批量摄入立即被查询
* 主键索引，二级索引: ClickHouse 主要采用了稀疏索引的方式做主键索引，minmax，set，ngrambf/tokenbf 等 Bloom Filter 去做二级索引

ClickHouse 不擅长:

* 没有高速，低延迟的更新和删除方法 
* 稀疏索引使得点查性能不佳
* 不支持事务

##  ClickHouse 为何会那么快

## ClickHouse 应用场景

* 实时数仓
* 实时日志分析，监控分析
* 用户行为分析，精细化运营分析: 日活，留存率分析，路径分析，有序漏斗转化率分析，Session 分析等


## ClickHouse的原理

* 当一个 SQL 传入服务器后，ClickHouse 将SQL 解析成 AST 语法树。

* 通过 Interpreter，将 AST 语法树转化成 QueryPlan，构建 Pipeline。这里 Interpreter 做的事情通常在别的数据库中，比如 Presto，会分成语义解析(元数据校验)，执行计划，执行计划优化等阶段。而在 ClickHouse 中这块代码耦合度相对比较高，个人觉得也是可以重构下的。

* 执行 Pipeline。 

## ClickHouse的技术poc清单

以下部分是确定可以实现的：

* 用户权限设置，比如对用户增删改查限制，直接对user赋予权限，角色赋予权限
* 库表的创建，调整，**TinyLog存储类型不支持add column操作**
* 从mysql同步数据到clickhouse
* clickhouse 抽取kafka
* oracle同步数据需要etl工具支持，或者使用flume

## 总结

* clichhouse是个olap数据库，擅长查询，不擅长读写
* ch保存的都是经过kv计算预处理过的**元数据**

## 可以落地的计划

* ClickHouse 容器化部署

容器化部署更高拥有更好的弹性伸缩能力，也能和其它的服务进行混合部署来节省成本

有些业务的导入数据量还是非常巨大的。但是其实查询量并不大，因为读写不分离，这时候导入数据量反而决定了集群的规模

将读写进行分离，写入部分通过 k8s 容器化技术临时构建集群来完成

* 在 ClickHouse 平台化

业务方易用性，多租户隔离，限流，熔断，监控报警，业务治理等方面进行更多地投入，特别是前期业务方还不多的时候，否则未来无论是平台侧还是业务方改造都是非常大的

* ClickHouse 更多的推广，更多业务的接入

用户行为分析，实时数仓，实时报表等业务， BI 平台支持 Presto 离线明细类报表和 Druid 实时预聚合报表，ClickHouse 实时明细类报表的引入，能够使得用户更加实时，方便地进行报表分析。

* 核心业务双链路

OLAP 的核心业务使用 Druid / doris + ClickHouse 的双链路去实现。

Druid 还要再调研，据说有问题
