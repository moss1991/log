# mysql

## 单库模式

* 占绝大多数情况
* 单独表最多1亿条数据可以运行
* 建议表行项目在千万条以内
* 缺点:可用性低
* 优点:简单

## 主从模式

* 主库、从库、分片中间件的模式
* 主库写、从库读
* 分片中间件：mycat、shardingshere
* 优点：
  * 适合读多，写少场景
  * 配合MHA中间件实现高可用
  * 主从数据同步
* 缺点：单独使用读写分离，超过10亿条数据单机无法支撑

## 分库分表（分片）

* 多个数据库，分片中间件进行组合
* 按照序号分片
* 几十亿数据到极限
* 范围法：1亿-2亿、2亿-3亿、3亿-4亿
  * 优点：
  * 缺点：
* hash法：
  * 优点：
  * 缺点： 