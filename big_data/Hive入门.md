### Hive 入门
### 1，数据仓库
- 数据仓库的目的是构建**面向分析**的集成化数据环境，为企业提供**决策支持**。它出于分析性报告和决策支持目的而创建。

#### 1.1 数仓的主要特征
- 面向主题：
  - 确认要分析什么，什么就是主题；
- 集成性
  - 数仓本身既不生产数据也不消费数据，其数据主要来源于外部业务系统；
  - 
- 非易失性（不可更新性）
  - 数据仓库的数据反映的是一段相当长的时间内历史数据的内容（离线仓库）；
  - 数据仓库中一般有大量的查询操作，很少有更新或删除的操作；
- 时变性
  - 数据仓库的目的是通过分析企业过去一段时间业务的经营状态，挖掘其中隐藏的模式；

#### 1.2 数据仓库与数据库的区别
- OLTP(On-Line Transaction Processing)，操作型处理，本身既生产数据也消费数据；
- OLAP(On-Line Analytical Processing)，分析型处理，一般针对某些主题的历史数据进行分析，支持管理决策；
- 数据仓库的出现，并不是要取代数据库：
  - 数据库是面向事务的设计，支持增删改查操作；数据仓库是面向主题设计的，主要是查询操作，很少有更新与删除；
  - 数据库一般存储业务数据，数据仓库存储的一般是历史数据；
  - 数据库设计是尽量避免冗余，一般是针对某一业务应用进行设计，比如一张简单的 User 表，记录用户名、密码等简单数据即可，符合业务应用，但是不符合分析。数据仓库在设计是有意引入冗余，依照分析需求，分析维度、分析指标进行设计；
  - 数据库是为捕获数据而设计，数据仓库是为分析数据而设计；

### 2，Hive（离线数仓）
- Hive 是一个 Hadoop 客户端，用于将 HQL(Hive SQL) 转化成 MapReduce 程序；
  - Hive 中每张表的数据存储在 HDFS；
  - Hive 分析数据底层的视线是 MapReduce（也可以配置为 Spark 或者 Tez）；
  - 执行程序运行在 Yarn 上；

#### 2.1 Hive 架构原理

![Hive 架构](Hive架构.png)

Hive 的架构可以分为以下几个组件：
- 用户接口：包括 CLI、JDBC/ODBC、webUI；
- 元数据(Metastore)：用以存储 Hive 中的库、表、列、注释、分区、表属性等信息；默认存储在 Hive 自带的 Derby 数据库中，企业中一般会通过配置将其存储到 MySQL 中；
- Driver 驱动程序：
  - 解释器(SQL Parser)：将 SQL 字符串转换成抽象语法树(AST)；
  - 编译器：将 AST 语法树转换为逻辑执行计划（SQL 语句到底有哪些，如何执行）；
  - 优化器：每个 SQL 语句都有自己的执行计划；
  - 执行器：将逻辑执行计划转换为物理执行计划（转换为 MR 或 Spark 实现最终的计算）；
- 执行引擎：默认的执行引擎是 MapReduce，可以通过配置将执行引擎替换为 Spark、Tez 等；

#### 2.2 数据表操作
- 数据库相关操作：

```sql
-- 创建数据库
create database db_bigdata;

-- 显示所有数据库
show databases;

-- 查看某个数据库详细信息
desc database db_bigdata;

-- 删除数据库
drop database db_bigdata;

-- 如果删除的数据库不为空，需要使用 cascade 强制删除，默认是 restrict，数据库不为空删除会报错；
drop database test cascade;
```

- 数据表相关操作

<details>
<summary>数据表相关操作</summary>

```sql
-- 查看表的详细信息
desc formatted db_hive;

-- 建表语句
create table t_hot_hero_skin_price (
    id int,
    name string,
    win_rate int,
    skin_price map<string, int>
)
row format delimited
fields terminated by ','
collection items terminated by '-'
map keys terminated by ':';

-- 存储的数据： 
-- fields terminated by: 列分隔符；
-- collection items terminated by：map、struct 和 array 中每个元素之间的分隔符；

-- map 数据结构
-- 1,孙悟空,53,西部大镖客:288-大圣娶亲:888-全息碎片:0-至尊宝:888-地狱火:1688
-- 2,鲁班七号,54,木偶奇遇记:288-大圣娶亲:888-全息碎片:0-至尊宝:888-地狱火:1688

-- JSON 数据
{
    "name": "张三",
    "friends": [
        "李四",
        "王五"
    ],
    "students": {
        "小郭": 18,
        "老于": 20
    },
    "address": {
        "street": "梁山",
        "city": "山东",
        "postal_code": 10010
    }
}

-- 建表语句
create table t_shuihu
(
    name     string,
    friends  array<string>,
    students map<string, int>,
    address  struct<city:string, street:string, postal_code:int>
)
row format serde 'org.apache.hadoop.hive.serde2.JsonSerDe';
```
</details>

#### 2.3 SerDe
- `SerDe`：Serializer and Deserializer 的缩写
- Hive uses SerDe (and FileFormat) to read and write table rows.
- HDFS files --> InputFileFormat --> <key, value> --> Deserializer --> Row object;
- Row object --> Serializer --> <key, value> --> OutputFileFormat --> HDFS files;

#### 2.4 加载表和导出表
```hql
-- 导入本地数据  local
load data local inpath 'filepath' into table t_bigdata;

-- 导入 HDFS 数据
load data inpath 'filepath' into table t_hdfs_bigdata;

-- insert + select 加载原表中的数据到新表

-- insert + directory 导出数据
insert overwrite local directory 'filepath'
row format delimited
fields terminated by ','
select * from t_student;
```

#### 2.5 Hive 表的类型
- 内部表(Managed_TABLE)：Hive 完全管理表（元数据和数据）的生命周期；
- 外部表(External_TABLE)：只管理表元数据的生命周期；删除外部表，只会删除元数据，而不会删除实际数据；
- 分区表(partition by)：
  - `partiton by(col_name)`：表示根据哪个字段进行分区；
  - 分文件夹存储，分区字段不能是表中已经存在的字段；
  - 分区字段可以是时间、地域、种类等具有标识意义的字段；
- 分桶表：
  - `cluster by(col_name)`：表示根据哪个字段进行分桶；
  - `into n buckets`：表示分为几个桶；
  - 分桶的字段必须是表中已经存在的字段；

### 3. Hive 调优
- MapReduce 运行模式：
  - 本地模式：`set mapreduce.framework.name=local`；
  - YARN 模式：`set mapreduce.framework.name=yarn`；

#### 3.1 Explain 执行计划
- Explain 执行计划由一系列具有依赖关系的 Stage 组成，每个 Stage 对应一个 MapReduce Job，或者一个文件系统操作等；
- 若某个 Stage 对应一个 MapReduce Job，其 Map 端和 Reduce 端的计算逻辑分别由 Map Operator Tree 和 Reduce Operator Tree 进行描述，Operator Tree 由一系列的 Operator 组成，一个 Operator 代表在 Map 或 Reduce 阶段的一个单一的逻辑操作，例如 TableScan Operator, Select Operator, Join Operator 等；

<details>
<summary>执行计划</summary>

```json
STAGE DEPENDENCIES:
  Stage-1 is a root stage
  Stage-0 depends on stages: Stage-1

STAGE PLANS:
  Stage: Stage-1
    Map Reduce
      Map Operator Tree:
          TableScan
            alias: test1
            Statistics: Num rows: 6 Data size: 75 Basic stats: COMPLETE Column stats: NONE
            Select Operator
              expressions: id (type: int)
              outputColumnNames: id
              Statistics: Num rows: 6 Data size: 75 Basic stats: COMPLETE Column stats: NONE
              Group By Operator
                aggregations: sum(id)
                mode: hash
                outputColumnNames: _col0
                Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
                Reduce Output Operator
                  sort order:
                  Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
                  value expressions: _col0 (type: bigint)
      Reduce Operator Tree:
        Group By Operator
          aggregations: sum(VALUE._col0)
          mode: mergepartial
          outputColumnNames: _col0
          Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
          File Output Operator
            compressed: false
            Statistics: Num rows: 1 Data size: 8 Basic stats: COMPLETE Column stats: NONE
            table:
                input format: org.apache.hadoop.mapred.SequenceFileInputFormat
                output format: org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat
                serde: org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe

  Stage: Stage-0
    Fetch Operator
      limit: -1
      Processor Tree:
        ListSink
```
</details>

#### 3.2 Hive 调优
- 分组聚合优化
- Join 优化
- 数据倾斜
- 任务并行度
- 小文件合并

- 分组聚合优化
  - 未经优化的分组聚合，是通过一个 MapReduce Job 实现的。Map 端负责读取数据，并按照分组字段区分，通过 Shuffle 将数据发往 Reduce 端，各组数据在 Reduce 端完成最终的聚合运算；
  - 分组聚合优化，主要围绕着减少 Shuffle 数量进行，具体做法是 map-side 聚合。所谓 map-side 聚合，就是在 map 端维护一个 hash table，利用其完成部分的聚合，然后将部分的聚合结果，按照分组字段分区，发送至 reduce 端，完成最终的聚合。
  - map-side 聚合能有效减少 shuffle 的数据量，提高分组聚合运算的效率；

```sql
-- 启用 map-side 聚合
set hive.map.aggr=true;

-- 用于检测源表数据是否适合进行 map-side 聚合。
-- 检测的方法是：先对若干条数据进行 map-side 聚合，若聚合后的条数和聚合前的条数比值小于该值，则认为该表适合进行 map-side 聚合；否则不适合。
set hive.map.aggr.hash.min.reduction=0.5;

-- 用于检测源表是否适合 map-side 聚合的条数
set hive.groupby.mapaggr.checkinterval=100000;

-- map-sice 聚合所用的 hash table, 占用 map task 堆内存的最大比例，若超过该值，则会对 hash table 进行一次 flush
set hive.map.aggr.hash.force.flush.memory.threshold=0.9
```

- Join 优化
- Hive 拥有很多种 Join 算法，包括 Common Join, Map Join, Bucket Map Join, Sort Merge Bucket Map Join 等；
  - Common Join
  - Map Join：大表 join 小表；
  - Bucket Map Join

- 数据倾斜
  - 数据倾斜问题，通常是指参与计算的数据分布不均，即某个 key 或者某些 key 的数据量远超其他 key，导致在 shuffle 阶段，大量相同 key 的数据发往同一个 reduce，进而导致该 reduce 所需的时间远超其他 reduce，成为整个任务的瓶颈。
  - Hive 中的数据倾斜常出现在分组聚合和 join 操作的场景中；
  - 由分组聚合导致的数据倾斜：
    - 开启 map-side 聚合；
    - Skew-GroupBy 优化：Skew-GroupBy 的原理是启动两个 MR 任务，第一个 MR 按照随机数分区，将数据分散发送到 Reduce，完成部分聚合，第二个 MR 按照分组字段分区，完成最终聚合。

```sql
-- 启用分组聚合数据倾斜优化
set hive.groupby.skewindata=true
```

- 由 Join 导致的数据倾斜，三种解决方案：
  - map join：使用 map join 算法，join 操作仅在 map 端就能完成，没有 shuffle 操作，没有 reduce 阶段，自然不会产生 reduce 端的数据倾斜。该方案适用于大表 join 小表时发生数据倾斜的场景。
  - skew join：原理是，为倾斜的大 key 单独启动一个 map join 任务进行计算，其余 key 进行正常的 common join。
  - 调整 SQL 语句；

```sql
-- 启用 Map Join 自动转换
set hive.auto.convert.join=true;

-- 一个 Common Join operator 转为 Map Join operator 的判断条件
set hive.mapjoin.smalltable.filesize=250000;

-- 开启无条件转 Map Join
set hive.auto.convert.join.noconditionaltask=true;

-- 无条件转 Map Join 时的小表之和阈值
set hive.auto.convert.join.noconditionaltask.size=10000000;


-- 启用 skew join 优化
set hive.optimize.skewjoin=true;
-- 触发 skew join 的阈值，若某个 key 的行数超过该参数值，则触发
set hive.skewjoin.key=100000;
```

<br/>

**参考资料：**
- [Hive 概述](https://www.sqlboy.tech/pages/eb41c7/#hive-架构)
- [docker-compose 部署 hive](https://blog.csdn.net/SJshenjian/article/details/131069153)
- [hive 优化](https://www.bilibili.com/video/BV1g84y147sX)