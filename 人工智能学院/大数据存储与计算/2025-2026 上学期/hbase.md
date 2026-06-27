![image-20251209010622892](C:\Users\赵兴理\AppData\Roaming\Typora\typora-user-images\image-20251209010622892.png)



HBase依赖于Hadoop集群,同时依赖ZooKeeper来管理Master和region地址,因此需要先安装ZooKeeper,再安装HBase.对yarn（资源调度）不要求，可以不启动。

#### Hbase是Hadoop平台的一个Map数据库 响应时间在毫秒级

如何理解HBase是一个用Map映射实现的数据库?

引用BigTable中的原文定义: sparse, distributed, persistent，multidimensional sorted map      

-  稀疏: 这是HBase的优势,如果某一列为空,不需要像关系数据库那样写一个NULL, HBase不需要填定任何标识或数据,节省大量的空间。       
- 分布式：数据不是存储在一个服务器，而是分布在整个集群中。　　
- 多维: key（不是row key） 是一个复合的数据结构，由rowkey, column family, qualifier，type 类型（插入，修改等）及timestamp组成。       
- 排序：这个key-value结构完全按照key的字典序进行排序。

## **① RowKey（行键）—— 第一维**

- 每行数据的唯一标识
- 全局字典序排序
- 查询时最重要的字段（只能通过 rowkey 定位）

## **② Column Family（列族）—— 第二维**

- HBase 设计的基本存储单元
- 在建表时静态定义
- 一个 column family 对应磁盘上的一个独立文件目录

## **③ Qualifier（列修饰符）—— 第三维（动态列）**

- 列族下面的“列名”
- 可以无限动态扩展（不需要提前建表）

## **④ Timestamp（时间戳）—— 第四维（版本）**

- 自动生成（毫秒级）或用户指定
  - 记录该 cell 的多个版本

#### HBase的优点：

- （1）容量大，单表可有千亿行，百万列；
- （2）易扩展，HBase 集群扩展非常容易；
- （3）稀疏性，不存储空值，对于稀疏表，可节省大量存储空间；
- （4）高性能，采用多种机制，查询响应能在毫秒级；
- （5）多版本，每个数据可以保存多个版本；
- （6）数据类型单一，只有字符串类型，没有其它类型；
- （7）支持过期操作，可以设置TTL (Time To Live生存期) ，过期数据会被自动清理。

   #### Region 是 HBase 中分布式存储和负载均衡的最小单元，不同的 Region 分布到不同的 RegionServer 上。

Region 虽然是分布式分布式存储的最小单元，但并不是存储的最小单元， Store 是存储的最小单元。Region 由一个或者多个 Store 组成，每个 Store 会保存一个 Column Family ；每个 Store 由一个 MemStore 和 0 至多个 Hfile 组成；MemStore 存储在内存中， HFile 存储在 HDFS 中。

#### Zookeeper安装最好为奇数台（半数以上就能运行）。

为什么需要过半同意？‌       

- 这能确保集群中‌有且只有一个‌多数派，从而避免出现“脑裂”（即一个集群分裂成两个或多个都认为自己是主集群的小集群）的情况，保证数据的一致性。

奇数节点如何带来优势？‌

通过对比来看奇数节点和偶数节点的容错能力。‌        

- 3节点集群 (奇数)‌：总节点数3，半数以上是2。这意味着集群要正常工作，至少需要2个节点存活。它的‌容错能力‌是允许挂掉1个节点 (3-2=1)。‌     
-  4节点集群 (偶数)‌：总节点数4，半数以上是3。集群要正常工作，至少需要3个节点存活。它的‌容错能力‌也是允许挂掉1个节点 (4-3=1)。

 通过对比可以看到，一个4节点的集群，虽然比3节点多了一台机器，成本更高，但其容忍节点失效的能力却和3节点集群一样。从资源利用率的角度看，3节点集群是更经济的选择。

### HBase的体系结构中主要包括以下组件：HBase客户端，ZooKeeper, Master, RegionServer, HDFS.

#### 1  HBase客户端

 	主要包括Shell命令行接口和Java API编程接口。HBase客户端主要通过RPC（Remote Procedure Call）机制与HMaster及HRegionServer通信，完成数据的增删改查操作(DML)和表的日常维护操作(DDL)。        

​	客户端在访问数据之前首先通过元数据表定位数据所在的RegionSever，之后才会发送信息给RegionServer。同时这些元数据会缓存在客户端，以方便下次使用。如果发生了负载均衡调整，或相应的RegionServer宕机，使得数据分片发生迁移，则需要重新请求元数据并缓存在本地。

#### 2  ZooKeeper功能

（1）实现HMaster的高可用；

（2）保存集群的元数据，完成Region寻址；

（3）参与RegionServer的宕[dàng]机处理， ZooKeeper可以感知RegionServer是否宕机，如果宕机，会通知HMaster处理。

（4）实现分布式表锁    当需要对整个表进行管理操作时，需要对整个表加锁，而表是分布存储的，ZooKeeper提供了对整个分布式表加锁的功能。

#### 3   HMaster

（1）处理用户的各种管理操作，如创建表，修改表，权限管理，Compaction等操作；

​    （2）管理集群中所有的RegionServer, 如负载均衡，宕机恢复，Region迁移等。  

  （3）清理过期日志文件Hlog及HFile文件，是否过期，是否已被删除。    

#### 4 RegionServer

##### RegionServer主要以下组件：     

   (1)WAL（Write ahead log）,也叫（Hlog）， 早期版本中,整个RegionServer对应一个WAL. 1.0后的版本支持多个WAL并行写, 所有写操作必须先保证写入到WAL，才会再写入到MemStore,再写入到HFile.       

(2)BlockCache       HBase的读缓存,提高系统的读性能.       

(3)Region       一个Region是表的一个水平切片.它由多个Store组成,每个Store由一个MemStore和0个或多个HFile组成.      



### hbase操作

![image-20251206210945914](C:\Users\赵兴理\AppData\Roaming\Typora\typora-user-images\image-20251206210945914.png)

### 行键相关过滤器（核心索引级过滤，性能最优）

| 过滤器类型            | 核心作用                                                     | 性能 | Shell 语法示例                                               |
| --------------------- | ------------------------------------------------------------ | ---- | ------------------------------------------------------------ |
| `PrefixFilter`        | 匹配行键**前缀**（如行键以`ORD202512`开头）                  | 极高 | `scan 'order', {FILTER => "PrefixFilter('ORD202512')"}`      |
| `RowFilter`           | 通用行键过滤，支持`=`/`>`/`<`/`>=`/`<=`/`!=`/`contains`等运算符 + 比较器 | 高   | # 行键大于 ROW100`scan 'user', {FILTER => "RowFilter(>, 'binary:ROW100')"}`# 行键包含 "UID"（正则）`scan 'user', {FILTER => "RowFilter(=, 'regexstring:.*UID.*')"}` |
| `InclusiveStopFilter` | 配合`scan`的`STARTROW/STOPROW`，让`STOPROW`包含在结果中（默认不包含） | 极高 | `scan 'user', {STARTROW => 'UID100', STOPROW => 'UID200', FILTER => "InclusiveStopFilter()"}` |

### 列族 / 列名相关过滤器（过滤列结构，不涉及值）

| 过滤器类型                   | 核心作用                                       | 性能 | Shell 语法示例                                               |
| ---------------------------- | ---------------------------------------------- | ---- | ------------------------------------------------------------ |
| `FamilyFilter`               | 按列族名过滤（支持运算符），筛选指定列族的列   | 中   | # 只保留列族为 base 的列`scan 'user', {FILTER => "FamilyFilter(=, 'binary:base')"}` |
| `QualifierFilter`            | 通用列名（Qualifier）过滤，支持运算符 + 比较器 | 中   | # 列名大于 age20`scan 'user', {FILTER => "QualifierFilter(>, 'binary:age20')"}` |
| `ColumnPrefixFilter`         | 匹配列名**前缀**（如列名以 phone 开头）        | 中高 | # 列名以 phone 开头的列`scan 'user', {FILTER => "ColumnPrefixFilter('phone')"}` |
| `MultipleColumnPrefixFilter` | 匹配多个列名前缀（批量筛选）                   | 中高 | # 列名以 phone 或 email 开头`scan 'user', {FILTER => "MultipleColumnPrefixFilter('phone','e` |

### 列值相关过滤器（过滤列值，业务场景最广）	

| 过滤器类型           | 核心作用                                                     | 性能 | Shell 语法示例                                               |
| -------------------- | ------------------------------------------------------------ | ---- | ------------------------------------------------------------ |
| `FirstKeyOnlyFilter` | 仅返回每行的**第一个列键值对**（行键 + 第一个列 + 值），不返回其他列 | 极高 | `scan 'user', {FILTER => "FirstKeyOnlyFilter()"}`            |
| `ValueFilter`        | 通用列值过滤，遍历所有列值匹配条件（支持运算符 + 比较器）    | 低   | # 任意列值包含 "北京"`scan 'user', {FILTER => "ValueFilter(=, 'substring:北京')"}`# 列值大于 1000`scan 'order', {FILTER => "ValueFilter(>, 'binary:1000')"}` |
