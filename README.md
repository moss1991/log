# 堡垒机参数

```
https://blj.cofco.com
10.6.128.189
sysadm  
6uUTmZFN$rOG
```

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

## 架构设计

MySQL -> 预计算 KV 系统 -> ClickHouse


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

OLAP 的核心业务使用 Druid + ClickHouse 的双链路去实现。

Druid 还要再调研，据说有问题

---




















# 视图和物化视图
---




















# ch教程
## 安装

### 安装server端 和 client 端
```
// 创建服务端
docker run -d --name demo-clickhouse-server -p 8123:8123 yandex/clickhouse-server
// 创建后删除客户端
docker run -it --rm --link demo-clickhouse-server:clickhouse-server yandex/clickhouse-client --host clickhouse-server
// 附加 映射方式

docker run -d --name=clickhouse-server \
-p 8123:8123 -p 9009:9009 -p 9090:9000 \
--ulimit nofile=262144:262144 \
-v /app/cloud/clickhouse/data:/var/lib/clickhouse:rw \
-v /app/cloud/clickhouse/conf/config.xml:/etc/clickhouse-server/config.xml \
-v /app/cloud/clickhouse/conf/users.xml:/etc/clickhouse-server/users.xml \
-v /app/cloud/clickhouse/log:/var/log/clickhouse-server:rw \
yandex/clickhouse-server:20.3.5.21
```

### 开放外网访问

开放外网访问，VIM CONFIG.XML 找到 LISTEN_HOST 标签，修改为以下

```
vim : /listen_host 找到代码位置
<listen_host>0.0.0.0</listen_host>
```

### 重启ch使配置生效

```
service clickhouse-server start
service clickhouse-server stop
service clickhouse-server restart
```

### 使用client连接
```
clickhouse-client -u dba -h 172.17.0.2 --password rufGA1j+  
```

### GUI连接

可以使用 `DataGrip`

---

## 用户管理

### 创建用户

* 通过 `sql` 方式

```
//在config.xml中添加access_control_path 配置项,通过 SQL 形式创建的用户、角色等信息将以文件的形式被保存在这个目录
<access_control_path>/var/lib/clickhouse/access/</access_control_path>

//在user.xml中为默认用户default添加access_management配置项,0代表disabled，1代表enabled。默认为0
<access_management>1</access_management>

// 创建
CREATE USER user_01 IDENTIFIED WITH PLAINTEXT_PASSWORD BY '123'
// 修改密码
ALTER USER user_01 IDENTIFIED WITH PLAINTEXT_PASSWORD BY '111111'
```

* 通过 `xml` 文件方式

```
// 进入容器
docker exec -it f2a607310a84 /bin/bash

// 安装vim
apt-get update
apt-get install vim

// 修改 user 配置文件，增加dba用户
vim /etc/clickhouse-server/users.xml

// 生成密码
PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'

rufGA1j+
84f8737b89665783e77d5d16bba0d9a2049a8142c81fb5b3c0b275912063d28a

// vim增加dba用户相关信息
<dba>      
  <password_sha256_hex>84f8737b89665783e77d5d16bba0d9a2049a8142c81fb5b3c0b275912063d28a</password_sha256_hex>
  // Network网络限制            
  <networks incl="networks" replace="replace">          
    <ip>::/0</ip>       
  </networks>      
  <profile>default</profile>     
  <quota>default</quota>       
  <allow_databases>           
    <database>test</database>    
  </allow_databases>
</dba>

```

### 用户的network配置

* ip
```
<!-设置具体的ip地址来限制登录用户-->
<ip>10.0.0.1/8</ip>

<!--为所有客户端打开权限-->
<ip>::/0</ip>

<!--仅允许本地登陆-->
<ip>::1</ip>
<ip>127.0.0.1</ip>
```
* host
```
<host>example1.host.com</host>
```
* host_regexp
```
<host_regexp>^example\d\d-\d\d-\d\.host\.ru$</host_regexp>
```

### 补充说明

* ip设置，代表允许哪些client可以连接server
* profile设置，表示该用户拥有该权限
* quota,表示对用户的限制
* database设置，设置用户可以访问的数据库，以及数据库下表的访问权限


### 用户授权 = Profile设置

* 通过 `xml` 文件方式

在user.xml配置文件的profile选项组下constraints选项组里定义对设置的约束，并禁止用户使用SET查询更改某些设置。


profile中有约束条件，从而限制其中的参数值被任意修改，约束条件有三种规则：
```
Min：最小值约束，对应参数取值不能小于该值.
Max：最大值约束，对应参数取值不能大雨该值.
Readonly：只读约束，对应参数禁止修改.
```
关于readonly 读权限、写权限和设置权限，由此标签控制，它有三种取值
```
0 默认值，可读可写
1 读权限，只能执行SELECT、EXISTS、SHOW和DESCRIBE
```
allow_ddl：DDL权限由此标签控制，它有两种取值
```
0，不允许DDL查询
1，允许DDL查询（默认值）
```

**示例，对用户增删改查限制**

```
//只读，不能DDL
<normal> 
  <readonly>1</readonly>
  <allow_ddl>0</allow_ddl>
</normal>

//读且能set，不能DDL
<normal_1> 
  <readonly>2</readonly>
  <allow_ddl>0</allow_ddl>
</normal_1>

//只读，即使DDL允许
<normal_2>
  <readonly>1</readonly>
  <allow_ddl>1</allow_ddl>
</normal_2>

//读写，能DDL
<normal_3>
  <readonly>0</readonly>
  <allow_ddl>1</allow_ddl>
</normal_3>
```

* 通过 `sql` 方式

方式一，直接对user赋予权限

```
// 添加
grant select on db_test.* to user_01 ;
Grant insert on db_test.* to user_01;
Grant delete on db_test.* to user_01;
Grant update on db_test.* to user_01;

// 去掉
revoke delete on db_test.* from user_01 ;
```

方式二，角色赋予权限
```
//创建角色
Creat role manager；
//对角色授权
GRANT SELECT ON *.* TO manager；
GRANT insert ON *.* TO manager；
//对用户赋予角色信息
GRANT manager to user_01;
//创建角色赋予默认的角色
Create USER test_01 default role manage；
```

## 用户限制 = quotas设置

* 通过 `xml` 文件方式

每个参数代表的含义：
```
1.Duration  累计时间周期，单位秒，到达周期时间清除所有收集值，然后重新计算。
2.Queries  在周期内允许查询次数限制
3.Errors  在周期内允许引发异常的次数限制
4.Result_rows  在周期内允许的查询返回结果行数
5.Read_rows  在周期内，允许远程节点读取的数据行数
6.Execution_time  允许查询的总执行时间，单位为秒
```

* 通过 `sql` 方式

```
//创建,限制最大的查询次数
CREATE QUOTA qA FOR INTERVAL 15 MONTH MAX QUERIES 123 TO user_01

//删除一个配额信息
DROP QUOTA [IF EXISTS] name [,...] [ON CLUSTER cluster_name]

//查询配额信息
SHOW CREATE QUOTA [name | CURRENT]
```

## 创建库和表

* 创建库
```
CREATE DATABASE IF NOT EXISTS mdp
```

* 创建表
```
CREATE TABLE t1(
  id UInt16,
  name String
) ENGINE=TinyLog
```

* 复制某个表的结构
```
create table t2 as t1 
engine=MergeTree 
order by id

//可以对其指定不同的表引擎声明
//如果没有表引擎声明，则创建的表将与db2.name2使用相同的表引擎
```

* 复制某个表的结构和数据

```
create table as 
select * from t1
t3engine=TinyLog
```

* 插入数据

```
insert into t1 (id,name,address) values(1,'aa','addr1'),(2,'bb','addr2')
```

```
insert into t2 select * from t1
```

## 调整表

* 给表增加列

```
//增加age列类型为Int8
alter table t1 add column age Int8
```

**TinyLog存储类型不支持add column操作**

这是因为clickhouse修改表是通过alter实现的，而TinyLog不支持alter操作，若要使用alter语法，可在建表时使用**MergeTree引擎**

查看表引擎

```
select engine from system.parts where table=table_name
```

* 修改表的列

```
//修改age列类型为String
alter table t1 modify column age String
```

* 删除列

```
//删除age列
alter table t1 drop column age
```

* 查询表

```
SHOW TABLES
```

* 查看表结构
```
desc t1
```

* 删除表
```
drop table t2
```

## 操作数据

* 替换表中某列的某个字符串

```
alter table demo.t1 update address = replace(address,'addr','no_where’)where address is not null
```


## 从mysql同步数据到clickhouse

```
// 进入容器
docker exec -it 155cf538f4f4 /bin/bash
// 尝试查询，测试连通性
select * from mysql('172.17.0.5:3306', 'flm_haiguan', 'result_2020', 'root', 'rc123456') 
// 基于查询结果创建表，这种创建方式可以把数据同步过来
CREATE TABLE tb_haiguan \
ENGINE = MergeTree \
ORDER BY id AS \
SELECT * \
FROM mysql('172.17.0.5:3306', 'flm_haiguan', 'result_2020', 'root', 'rc123456')
// 测试查询速度
select trade_country_name, product_name ,count(*) as count from test1  group by trade_country_name,product_name order by count desc limit 0,20
```

## ClickHouse实时同步MySQL数据

```
cat metainfo.conf 
```

## clickhouse抽取oracle数据

```
JDBC('jdbc:mysql://localhost:3306/?user=root&password=root', 'test', 'test')
```

```
jdbc:oracle:thin:@//172.17.0.7:8003/helwin/?user=system&password=system
```


## clickhouse 抽取kafka

### kafka中创建消息
```
from kafka import KafkaProducer
import json
import datetime

topic='ch4'
producer = KafkaProducer(
    bootstrap_servers='10.6.128.189:8007',
    value_serializer=lambda m:json.dumps(m).encode("utf-8"))  # 连接kafka

for i in range(10):
    # data={"cofco_number":i,"ts":datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")}
    data = {
        "cofco_number" : i,
        # "test":"teetsttst"
        "ts" : datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    }
    producer.send(topic,data)

producer.close()

# 参数bootstrap_servers：指定kafka连接地址
# 参数value_serializer：指定序列化的方式，我们定义json来序列化数据，当字典传入kafka时自动转换成bytes
# 用户密码登入参数
# security_protocol="SASL_PLAINTEXT"
# sasl_mechanism="PLAIN"
# sasl_plain_username="maple"
# sasl_plain_password="maple"
```

### kafka中消费消息

```
#消费者
from kafka import KafkaConsumer
import time

topic = 'ch4'
consumer = KafkaConsumer(
    topic, 
    bootstrap_servers=['10.6.128.189:8007'], 
    group_id="test", auto_offset_reset="earliest")

for msg in consumer:
    recv = "%s:%d:%d: key=%s value=%s" % (msg.topic, msg.partition, msg.offset, msg.key, msg.value)
    print(recv)
    # print(msg.value)
    # time.sleep(1)

# 参数bootstrap_servers：指定kafka连接地址
# 参数group_id：如果2个程序的topic和group_id相同，那么他们读取的数据不会重复，2个程序的topic相同，group_id不同，那么他们各自消费相同的数据，互不影响
# 参数auto_offset_reset：默认为latest表示offset设置为当前程序启动时的数据位置，earliest表示offset设置为0，在你的group_id第一次运行时，还没有offset的时候，给你设定初始offset。一旦group_id有了offset，那么此参数就不起作用了

```



### ch中创建表
```
CREATE TABLE queue1 ( \
    cofco_number Int, \
    ts String \
) ENGINE = Kafka SETTINGS kafka_broker_list = '10.6.128.189:8007', \
    kafka_topic_list = 'ch4', \
    kafka_group_name = 'test_group_name', \
    kafka_format = 'JSONEachRow'

```

### ch中查询数据

注意 注意 注意

这种做法，相当于ch消费了指定的topic，每当查询之后，相当于消费掉了

所以会出现，python产生消息，ch中查询，可以有结果，但是这个select相当于消费掉了，再次在ch中查询，就会发现没有数据

## 3.5 容器里数据存放位置

```
/var/lib/clickhouse
```




















---
# oracle教程

## oracle 循环插入数据
```
INSERT INTO "SMTBI_DATAIN_ZLB"."IM_OPERATION_T_ACTUAL_YEAR_CP" ("PROFESSION_COMPANY", "ENTITY_ID", "YEARS", "MONTHS", "SCENE", "CURRENCY", "VIEWPOINT", "PROJECT_ID", "MATERIAL_ID", "VALUE", "ITEM_ID", "ID1", "ID2", "ID3", "AMOUNT", "CREATOR", "CREATE_TIME", "UPDATOR", "UPDATE_TIME", "ROWID") VALUES ('1020', 'A1020001', '2021', '01', 'S02', 'CNY', 'V02', 'Z01999', '9999999', 'VA01', '71000000', NULL, NULL, NULL, '1000', 'test', TO_DATE('2021-04-28 09:47:20', 'SYYYY-MM-DD HH24:MI:SS'), 'test', TO_DATE('2021-04-28 09:47:25', 'SYYYY-MM-DD HH24:MI:SS'), 'AABEH8AAJAAA64MAAA');

INSERT INTO "SMTBI_DATAIN_ZLB"."IM_OPERATION_T_ACTUAL_YEAR_CP" ("PROFESSION_COMPANY", "ENTITY_ID", "YEARS", "MONTHS", "SCENE", "CURRENCY", "VIEWPOINT", "PROJECT_ID", "MATERIAL_ID", "VALUE", "ITEM_ID", "ID1", "ID2", "ID3", "AMOUNT", "CREATOR", "CREATE_TIME", "UPDATOR", "UPDATE_TIME") VALUES ('1020', 'A1020001', '2021', '01', 'S02', 'CNY', 'V02', 'Z01999', '999499', 'VA01', '71000000', NULL, NULL, NULL, '1000', 'test', TO_DATE('2021-04-28 09:47:20', 'SYYYY-MM-DD HH24:MI:SS'), 'test', TO_DATE('2021-04-28 09:47:25', 'SYYYY-MM-DD HH24:MI:SS'));


begin 
  for i in 1 .. 50000 loop
     INSERT INTO "SMTBI_DATAIN_ZLB"."IM_OPERATION_T_ACTUAL_YEAR_CP" ("PROFESSION_COMPANY", "ENTITY_ID", "YEARS", "MONTHS", "SCENE", "CURRENCY", "VIEWPOINT", "PROJECT_ID", "MATERIAL_ID", "VALUE", "ITEM_ID", "ID1", "ID2", "ID3", "AMOUNT", "CREATOR", "CREATE_TIME", "UPDATOR", "UPDATE_TIME") VALUES ('1020', 'A1020001', '2021', '01', 'S02', 'CNY', 'V02', 'Z01999', '999' || i , 'VA01', '71000000', NULL, NULL, NULL, '1000', 'test', TO_DATE('2021-04-28 09:47:20', 'SYYYY-MM-DD HH24:MI:SS'), 'test', TO_DATE('2021-04-28 09:47:25', 'SYYYY-MM-DD HH24:MI:SS'));
  end loop;
end;
```

## docker镜像快速安装oracle

```
docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g

docker run -d -p 8003:1521 --name oracle11g registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g

docker exec -it oracle11g bash
```

更换成root用户
```
su root 
helowin
```

设置环境变量
```
vi /etc/profile

export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
export ORACLE_SID=helowin
export PATH=$ORACLE_HOME/bin:$PATH
```
创建软连接
```
ln -s $ORACLE_HOME/bin/sqlplus /usr/bin
```


```
1) cd /var/tmp
2) chown oracle:dba .oracle
```

切换到oracle 用户，刷新权限

```
sqlplus /nolog

conn /as sysdba

alter user system identified by system;

alter user sys identified by sys;
```


















# mysql教程

代码中的声明
```
class Data(db.Model):
	__tablename__ = "datas"
	id = db.Column(db.Integer, primary_key=True)
	smallInteger = db.Column(db.SmallInteger)
	bigInteger = db.Column(db.BigInteger)
	floatData = db.Column(db.Float(10))
	numericData = db.Column(db.Numeric(10))
	stringData = db.Column(db.String(250))
	textData = db.Column(db.Text(200))
	mediumText = db.Column(db.Text(65536))
	longText = db.Column(db.Text(16777216))
	largeBinary = db.Column(db.LargeBinary(300))
	mediumBlob = db.Column(db.LargeBinary(65536))
	longBlob = db.Column(db.LargeBinary(16777216))
	pickle = db.Column(db.PickleType)
	mediumPickle = db.Column(db.PickleType(65536))
	longPickle = db.Column(db.PickleType(16777216))
	unicodeData = db.Column(db.Unicode(10))
	unicodeText = db.Column(db.UnicodeText)
	booleanData = db.Column(db.Boolean(0))
	dateData = db.Column(db.Date)
	timeData = db.Column(db.Time)
	dateTime = db.Column(db.DateTime)
	interval = db.Column(db.Interval)
	enumData = db.Column(db.Enum('father', 'mother'))
	def __repr__(self):
		return "Data {}".format(self.id)
```
对应的表结构
```
+--------------+-------------------------+------+-----+---------+----------------+
| Field        | Type                    | Null | Key | Default | Extra          |
+--------------+-------------------------+------+-----+---------+----------------+
| id           | int(11)                 | NO   | PRI | NULL    | auto_increment |
| smallInteger | smallint(6)             | YES  |     | NULL    |                |
| bigInteger   | bigint(20)              | YES  |     | NULL    |                |
| floatData     |  float                   | YES  |     | NULL    |                |
| numericData  | decimal(10,0)           | YES  |     | NULL    |                |
| stringData   | varchar(250)            | YES  |     | NULL    |                |
| textData     | tinytext                | YES  |     | NULL    |                |
| mediumText   | mediumtext              | YES  |     | NULL    |                |
| longText     | longtext                | YES  |     | NULL    |                |
| largeBinary  | blob                    | YES  |     | NULL    |                |
| mediumBlob   | mediumblob              | YES  |     | NULL    |                |
| longBlob     | longblob                | YES  |     | NULL    |                |
| pickle       | blob                    | YES  |     | NULL    |                |
| mediumPickle | blob                    | YES  |     | NULL    |                |
| longPickle   | blob                    | YES  |     | NULL    |                |
| unicodeData  | varchar(10)             | YES  |     | NULL    |                |
| unicodeText  | text                    | YES  |     | NULL    |                |
| booleanData  | tinyint(1)              | YES  |     | NULL    |                |
| dateData     | date                    | YES  |     | NULL    |                |
| timeData     | time                    | YES  |     | NULL    |                |
| dateTime     | datetime                | YES  |     | NULL    |                |
| interval     | datetime                | YES  |     | NULL    |                |
| enumData     | enum('father','mother') | YES  |     | NULL    |                |
+--------------+-------------------------+------+-----+---------+----------------+
```






















---
# portainer

```
docker run -p 8080:9000 -p 8000:8000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /mydata/portainer/data:/data -d portainer/portainer
```


















## kafka

### 查看版本

`cd /opt/kafka/libs ` 查看文件版本

### 拉镜像
```
docker pull wurstmeister/zookeeper
docker pull wurstmeister/kafka
```

### 创建zookeeper
```
docker run -d --name zookeeper -p 8006:2181 -t wurstmeister/zookeeper
```

### 创建kafka
```
docker run -d --name kafka \
-p 8007:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=172.17.0.5:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.6.128.189:8007 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka
```

### 进入容器

```
docker exec -it ${CONTAINER ID} /bin/bash
```

### 测试消息
```
/opt/kafka/bin
# 创建主题
./kafka-topics.sh --create --zookeeper 172.17.0.5:2181 --replication-factor 1 --partitions 1 --topic mykafka

#运行一个生产者
./kafka-console-producer.sh --broker-list 10.6.128.189:8007 --topic mykafka

#运行一个消费者
./kafka-console-consumer.sh --bootstrap-server 10.6.128.189:8007 --topic mykafka --from-beginning

```

### 操作

主题相关的
```
//查看所有topic
./bin/zookeeper-client

```

### 管理平台
docker run -d -p 8008:9000 -e ZK_HOSTS="172.17.0.5:2181" -e APPLICATION_SECRET=letmein sheepkiller/kafka-manager



















## pytorh

```
docker run -it -p 8005:22 -p 8006:6006 -p 8007:8888 -v /user/project:/home/user/disk2/menfer/project --name menfer pytorch/pytorch
```
