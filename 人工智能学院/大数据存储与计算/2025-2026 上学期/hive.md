```sql
-- 核心必配（保证动态分区生效）
SET hive.exec.dynamic.partition = true;全局开启动态分区功能	
SET hive.exec.dynamic.partition.mode = nonstrict;允许 “全动态分区”（所有分区字段都自动生成）
SET hive.mapred.mode = nonstrict;解除 Hive 全局严格模式的限制	

-- 非必需（按需调整，避免分区数超限）
SET hive.exec.max.dynamic.partitions.pernode = 1000;限制单个 NodeManager 节点允许创建的最大动态分区数	
-- 可选：限制全局最大动态分区数（避免全集群过载）
SET hive.exec.max.dynamic.partitions = 10000;
```

```sql
set hive.strict.checks.bucketing = false; 关闭 Hive 对分桶表的严格校验规则	
set hive.mapred.mode = nonstrict; 全局解除 Hive 严格模式的限制	
set hive.enforce.bucketing = true; 启用 Hive 自动分桶机制	
set mapreduce.job.reduces = -1; 让 Reduce 数量由 Hive 自动计算	
```



wc –l 显示行数 –w 显示单词数；

###  UDF（User Defined Function）UDAF（User Defined Aggregate Function）UDTF（User Defined Table-Generating Function）

| 类型     | 输入 | 输出      | 是否聚合 | 典型用途               |
| -------- | ---- | --------- | -------- | ---------------------- |
| **UDF**  | 一行 | 一行      | 否       | 字符串处理、数值计算   |
| **UDAF** | 多行 | 一行      | 是       | sum、avg、count 等聚合 |
| **UDTF** | 一行 | 多行/多列 | 否       | explode、拆字段        |

### 视图

- 视图是基于数据库的基本表创建的一种伪表，因此数据库只存储视图的定义，不存储数据项，数据项仍然存在基本表中。视图可作为抽象层，将数据发布给下游用户。    
- 视图只能用于查询，但不能进行数据的插入和修改，提高了数据的安全性。在创建视图时，视图就已经固定，因此对基本表的后续更改（如添加列等操作）将不会反映在视图中。        
- 视图允许从多个表中抽取字段。使用视图可以降低查询的复杂度，达到优化查询的目的。
- 视图是基于基本表创建的，并没有将真实数据存储在Hive中，若删除视图关联的基本表，则查询视图内容时将会报错。

### hive优化

- Fetch抓取是指在Hive中对某些数据的查询可以不必使用MapReduce计算，而是读取存储目录下的文件，再输出查询结果到控制台，如全局查询、字段查询和使用LIMIT语句查询。         在特殊的场景下配置Fetch抓取，可以提高查询的效率。
- 调整Map任务数       并不是map任务数越多，其执行效率就越高。       假设一个任务存在多个小文件，每一个小文件都会启动一个map任务，当一个map任务启动和初始化的时间远远大于逻辑处理的时间，会造成很大的资源浪费，并且可执行的map任务数是有限的。       因此，可以设置在map任务执行前合并小文件以达到减少map任务数的目的。
- 其实在Hive作业的众多阶段中，并非所有的阶段都是完全相互依赖的，因此存在某些阶段是允许并行执行的。通过配置并行执行，可以使得整个Hive作业的执行时间缩短。
- 在Hive中，尽量先多使用子查询和使用WHERE语句降低表数据的复杂度，再使用JOIN连接查询。如果先进行表连接，那么查询将先进行全表扫描，最后才使用WHERE语句筛选，执行效率将会降低。如果先使用子查询，那么可利用WHERE语句过滤不相关字段，不但能增加map任务数，还能减小数据量。

### 正则表达式

^ 匹配一行的开头，^a       匹配"anA"，⽽不匹配"Ana"

匹配一行的结尾   a$（美金符号）      匹配"Ana"，⽽不匹配"anA"

匹配前面元字符0次或多次，ba*       将匹配 b,ba, baa, baaa

匹配前面元字符1次或多次，ba+      将匹配ba, baa, baaa

? 匹配前面元字符0次或1次，ba?        将匹配b, ba

. 换行符以外的任意字符

\d 数字                  \w  数字，字母，下划线 

x|y 匹配x或y

{n} 精确匹配n次

{n,}匹配n次以上

{n,m} 匹配n-m次

[xyz] 字符集(character set)，匹配这个集合中的任一个字符 

[^xyz] 不匹配这个集合中的任何一个字符

#### 语法格式：a regexp b

- 说明：a是需要匹配的字符串，b是正则表达式字符串

- 返回值：true/false

#### 语法: regexp_extract(string subject, string pattern, int index)

- 返回值: string
- 说明： 将字符串subject按照pattern正则表达式的规则拆分，返回index指定的字符串。
- 第一参数： 要处理的字段
- 第二参数: 需要匹配的正则表达式
- 第三个参数:
- - 0：返回与之匹配的整个字符串       
  - 1：返回第1个括号里面的  
  - 2：返回第2个括号里面的字段...