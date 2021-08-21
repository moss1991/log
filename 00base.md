# 通俗解释

## 思想类
* “没有银弹”：没有银色子弹的口语版本，两层意思，一是不同场景需要不同的方案和技术，二是没有万能的技术
* “少即是多”：懒是人性的本质，为了提高用户体验，系统要多做事和自动，用户操作要少和简单
* “UCD”：user center design = 用户为中心的设计思想，项目模型分为系统模型、业务模型、用户心理模型，设计的出发点是用户，这样的产品更容易推广使用
* “DDD”：domain driver design = 领域驱动设计，设计和实施系统前先了解、调研领域，归集领域知识，根据领域知识来一步步驱动软件设计


## 技术类
* “读写分离”：吃饭上厕所不能搁一块，不光使不出劲儿，而且自己恶心，别人看着也恶心
* “微服务”：相似功能、完成一件事的一些接口，打包成一个微服务，不同的微服务可以看成：政务中心、银行、法院、派出所、医院、饭店
* “中间件”：特定功能的组件，是个二道贩子，例：医院的黄牛、中英语翻译
* “技术中台”：团队的研发力量体现
* “数据中台”：提供数据的入存管出的功能，负责让系统更加智慧
* “业务中台”：规范化、标准化做事的过程，提供标准流程和最佳实践，负责让系统更敏捷
* “云大物移智”：云计算、物联网、移动互联网、大数据、智慧城市
* “云计算”：云计算是基础，提供计算能力，是芯片操作系统、应用软件到服务产业链的垂直整合
* “物联网”：传感器构成传感器网络，能够收集并分析物联网终端的传感器生成的数据
* “大数据”：沙中淘金
* “移动互联网”：为用户提供各类信息服务，探索新的生产关系
* “智慧城市”：新型城市发展方式

## 行业术语
* “talk is cheat,show me the code”：程序员之间的battle, 等于“就你写的那玩意儿，可别吹了”
* “屎山”：英文是shit mountain，指的是长期缺乏管理的代码库，程序员各写各的，导致别人维护起来的感觉就是在“shit mountain”中恶心
* “冒烟儿跑”：低质量的程序，就像一台已经着火冒着黑烟跑的列车，一旦停下来，没人知道会不会爆炸，没人敢保证停下来还能再启动起来
* “在吗？”：又要改需求了
* “回去再确认一下”：别说了，老子没想到
* “那个bug没问题啊，你再试试”：我刚偷偷改完这个bug

<br><br><br><br>

# 数据分析平台的对比

| | 传统数据分析平台 | 大数据分析平台|
|---|---|---|
| 关注点 | 描述性分析 <br> 诊断性分析| 预测性分析 |
| 数据集 | 有限的数据集 <br> 干净的数据集 <br> 简单方法 | 大规模数据集 <br> 多类型原始数据 <br> 复杂数据模型 |
| 分析结果 | 事件及其原因 | 新的规律和知识|

<br><br><br><br>

# HBase与RDBMS对比
| | HBase | RDBMS |
|---|---|---|
| 数据类型 | 只有字符串 | 丰富的数据类型 |
| 硬件环境 | 和Hadoop一样，可部署在大量廉价的PC Server上 | 昂贵的企业集群系统 |
| 数据操作 | 简单的增删改查 | 各种各样的函数，表连接 |
| 错误容忍 | 单个节点的错误基本不影响整体性能 | 高可靠性，宕机恢复成本高 |
| 存储模式 | 基于列存储 | 基于表格结构和行存储 |
| 数据保护 | 更新后旧版本仍然会保留 | 替换 |
| 可伸缩性 | 轻易的进行增加节点，可扩展性高 | 需要中间层 |

<br><br><br><br>

# Hive和HBase协作关系

数据源（OLTP） - 数据存储（HDFS） - 数据计算（HIVE） - 海量数据查询（HBASE）/ 其他任何一种数据库 - 数据应用