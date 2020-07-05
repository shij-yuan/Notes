# order by

分为两类：需要回表与不需要回表

```sql
select city,name,age from t where city='杭州' order by name limit 1000  ;
```

## 全字段排序（additional_fields / 不回表）

使用空间换时间。MySQL会给每个线程分配一块内存用于排序（sort_buffer）。

### 流程

1.  初始化sort_buffer，确定放入name、city、age这三个字段；
2. 从索引city找到第一个满足 city='杭州’ 条件的主键id；
3. 到主键id索引取出整行，取name、city、age三个字段的值，存入sort_buffer中；
4. 从索引city取下一个记录的主键id；
5. 重复步骤3、4直到city的值不满足查询条件为止；
6. 对sort_buffer中的数据按照字段name做快速排序；
7. 按照排序结果取前1000行返回。

如果要排序的数据量小于sort_buffer_size，排序就在内存中完成；若大于，就需要使用外部排序，外部排序一般使用归并排序算法。**MySQL将需要排序的数据分成12份，每一份单独排序后存在这些临时文件中。然后把这12个有序文件再合并成一个有序的大文件。**



## rowid排序（回表）

若**排序的单行长度**大于`max_length_for_sort_data`，则Mysql会认为sort_buffer里面要放的字段数太多，内存里能够同时放下的行数很少，要分成多个临时文件，降低排序的性能。

### 流程

1. 初始化sort_buffer，确定放入**两个字段，即 name 和 id** ；
2. 从索引city找到第一个满足city='杭州’条件的主键id；
3. 到主键id索引取出整行，取name、id这两个字段，存入sort_buffer中；
4. 从索引city取下一个记录的主键id；
5. 重复步骤3、4直到不满足city='杭州’条件为止；
6. 对sort_buffer中的数据按照字段name进行排序；
7. 遍历排序结果，取前1000行，**并按照id的值回表查询取出city、name和age三个字段**返回。



## 覆盖索引（无需排序）

Using index

