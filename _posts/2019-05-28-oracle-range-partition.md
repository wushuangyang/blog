---
layout: post
title: Oracle表范围分区
category: Oracle
---

## Oracle表范围分区（range partition）

分区表就是通过使用分区技术，将一张大表，拆分成多个表分区（独立的segment），从而提升数据访问的性能，以及日常的可维护性。
分区表中，每个分区的逻辑结构必须相同。如：列名、数据类型;每个分区的物理存储参数可以不同。如：各个分区所在的表空间。
对于应用而言完全透明，分区前后没有变化，不需要进行修改。

- 在维护性方面，可以在分区级别，针对单独的分区，进行索引的维护、数据的加载以及备份恢复等操作。大大降低了维护时长。
- 在可用性方面，由于各个分区相对独立，当一个分区处于维护或者出现故障时，不会影响到其他分区的正常使用。
- 在性能方面，oracle对于用户的请求，只检索需要的分区，从而提升性能。
- 在其他方面，由于分区表对于用户是透明的，因此，不需要在分区后，对代码进行修改。


### 范围分区特点：
- 范围分区主要依据分区键定义时给出的键值范围，根据实际的取值，进行分区的选择，进而在相应分区中存储数据。
- 范围分区比较合适存在以数字为导向，方便进行数字范围划分的数据列。如：员工表的雇佣日期列、工资列等。
- 范围分区的数据分布可能不均匀。

### 范围分区定义规则：
- 在定义范围分区时，每个分区定义必须使用 values less than（value）子句。其中（value）表示该分区的上限值。
- 在定义范围分区时，最后一个分区可以是values less than（maxvalue）。其中（maxvalue）表示该分区存储高于其他分区上限值的数据行。


### 测试例子

创建分区表（范围分区（range partition），按月分区）:
```sql
CREATE TABLE part_test1
(
CNSMRSRLNO VARCHAR2(50) NOT NULL,
SUBAREATIME DATE,
STEPID VARCHAR2(60),
BUSINESSID VARCHAR2(32),
PRIMARY KEY (CNSMRSRLNO)
)
PARTITION BY RANGE (SUBAREATIME)
(
PARTITION p201905 VALUES LESS THAN (TO_DATE('2019-06-01', 'yyyy-mm-dd')) ,
PARTITION p201906 VALUES LESS THAN (TO_DATE('2019-07-01', 'yyyy-mm-dd')) ,
PARTITION p201907 VALUES LESS THAN (TO_DATE('2019-08-01', 'yyyy-mm-dd')) ,
PARTITION p201908 VALUES LESS THAN (TO_DATE('2019-09-01', 'yyyy-mm-dd')) ,
PARTITION p201909 VALUES LESS THAN (TO_DATE('2019-10-01', 'yyyy-mm-dd')) ,
PARTITION p201910 VALUES LESS THAN (TO_DATE('2019-11-01', 'yyyy-mm-dd')) ,
PARTITION p201911 VALUES LESS THAN (TO_DATE('2019-12-01', 'yyyy-mm-dd')) ,
PARTITION p201912 VALUES LESS THAN (TO_DATE('2020-01-01', 'yyyy-mm-dd')) 
);
```
创建本地索引:

```sql
CREATE INDEX
    IDX_part_test1
ON
    part_test1
(
        SUBAREATIME
)
    LOCAL 
(
PARTITION p201905 ,
PARTITION p201906 ,
PARTITION p201907 ,
PARTITION p201908 ,
PARTITION p201909 ,
PARTITION p201910 ,
PARTITION p201911 ,
PARTITION p201912
);
```


删除分区：
```sql
ALTER TABLE   part_test1 drop PARTITION p201905;
```

增加分区:
```sql
ALTER TABLE part_test1 ADD PARTITION p202001 VALUES LESS THAN (TO_DATE('2020-02-01', 'yyyy-mm-dd')) ;
```