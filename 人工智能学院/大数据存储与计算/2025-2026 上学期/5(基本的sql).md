 ![image-20251206201346036](C:\Users\赵兴理\AppData\Roaming\Typora\typora-user-images\image-20251206201346036.png)



### order by和sort by

| 特性         | **ORDER BY**               | **SORT BY**                                |
| :----------- | :------------------------- | :----------------------------------------- |
| **排序范围** | **全局排序**（整个数据集） | **分区内排序**（每个Reduce任务内）         |
| **结果数量** | 最终只有 **1个** 输出文件  | 有 **多个** 输出文件（每个Reduce任务一个） |
| **性能**     | 性能差，容易内存溢出       | 性能好，适合大数据量                       |
| **使用场景** | 需要全局有序的小数据量     | 大数据量的预处理或分区内排序               |

**sort by 的方向固定为升序（ASC）**，不能变。

#### distribute by

```
 DISTRIBUTE BY关键字控制Mapper的输出如何分发到Reducer中，将具有相同哈希值的记录被分发到同一个Reducer中。
         Hive 会根据指定的列计算哈希值，相同哈希值的数据会被发送到同一个 reducer，每个 reducer 处理自己接收到的数据分区。
         DISTRIBUTE BY关键字不具备排序的作用。
典型使用场景
预处理数据‌：为后续的 JOIN 或聚合操作优化数据分布。
避免数据倾斜‌：通过合理分配减少某些 reducer 过载。
与 SORT BY 结合‌：实现分区内排序（DISTRIBUTE BY col1 SORT BY col2）。

```

#### cluster by

```
 DISTRIBUTE BY关键字控制Mapper的输出如何分发到Reducer中，将具有相同哈希值的记录被分发到同一个Reducer中。
         Hive 会根据指定的列计算哈希值，相同哈希值的数据会被发送到同一个 reducer，每个 reducer 处理自己接收到的数据分区。
         DISTRIBUTE BY关键字不具备排序的作用。
典型使用场景
预处理数据‌：为后续的 JOIN 或聚合操作优化数据分布。
避免数据倾斜‌：通过合理分配减少某些 reducer 过载。
与 SORT BY 结合‌：实现分区内排序（DISTRIBUTE BY col1 SORT BY col2）。

```





### 创建外部表

```sql
create external table student_for_join(sno varchar(10),sname varchar(10))
row format delimited fields terminated by ',’ 
stored as textfile location '/data/student_for_join';
```

```ba
vi /usr/hivetestdata/student_for_join
hdfs dfs -put /usr/hivetestdata/student_for_join	 /data/student_for_join
```

```sql
select * from school.student_for_join;
```

### 连接查询

sno,sname                    sno,cno,grade

![image-20251130221031614](C:\Users\赵兴理\AppData\Roaming\Typora\typora-user-images\image-20251130221031614.png)![image-20251130221047271](C:\Users\赵兴理\AppData\Roaming\Typora\typora-user-images\image-20251130221047271.png)

#### 内连接

只返回两个表中**连接条件匹配**的记录。

```sql
select student_for_join.sno, student_for_join.sname, sc_for_join.cno, sc_for_join.grade from student_for_join inner join sc_for_join on student_for_join.sno=sc_for_join.sno where student_for_join.sno=1;
```

 ![image-20251130221157969](C:\Users\赵兴理\AppData\Roaming\Typora\typora-user-images\image-20251130221157969.png)

#### 左外连接

返回**左表的所有记录** + 右表匹配的记录（不匹配的显示NULL）

```sql
select * from student_for_join a left outer join sc_for_join b on a.sno=b.sno;
```

 ![image-20251130221318998](C:\Users\赵兴理\AppData\Roaming\Typora\typora-user-images\image-20251130221318998.png)

#### 右外连接

返回**右表的所有记录** + 左表匹配的记录（不匹配的显示NULL）

```sql
select * from student_for_join a right outer join sc_for_join b on a.sno=b.sno;
```

 ![image-20251130221353041](C:\Users\赵兴理\AppData\Roaming\Typora\typora-user-images\image-20251130221353041.png)

#### 全外连接

返回左右两表的**所有记录**，匹配的显示数据，不匹配的显示NULL

```sql
select * from student_for_join a full outer join sc_for_join b on a.sno=b.sno;
```

 ![image-20251130221452648](C:\Users\赵兴理\AppData\Roaming\Typora\typora-user-images\image-20251130221452648.png)



