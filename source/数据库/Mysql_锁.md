# MySQL锁



## **InnoDB锁模式**

**行锁：**

- 共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排他锁。 
- 排他锁（X）：允许获得排他锁的事务更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁。

为了允许行锁和表锁共存，实现多粒度锁机制，InnoDB 还有两种内部使用的意向锁（Intention Locks），这两种意向锁都是**表锁**：

- 意向共享锁（IS）：事务打算给数据行加行共享锁，事务在给一个数据行加共享锁前必须先取得该表的 IS 锁。 
- 意向排他锁（IX）：事务打算给数据行加行排他锁，事务在给一个数据行加排他锁前必须先取得该表的 IX 锁。





## **InnoDB 行锁实现方式：**

- InnoDB 行锁是通过给索引上的索引项加锁来实现的，只有通过索引条件检索数据，InnoDB 才使用行级锁，否则，InnoDB 将使用表锁（**真正使用索引时**）
- 多个session是访问不同行的记录， 但是如果**使用相同的索引键**， 会出现锁冲突的



## 锁算法

- Record lock 记录锁：锁定行记录

- Gap lock：间隙锁：锁定一个区间，是**键值在条件范围内，但不存在的记录**

    锁定的是索引记录之间的间隙，并发事务插入新数据前会先检测间隙中是否已被加锁，防止幻读的出现

    若执行的条件是范围过大，则InnoDB会将整个范围内所有的索引键值全部锁定，很容易对性能造成影响。

- Next-key lock：记录锁 + 间隙锁，锁定行记录 + 区间



**快照读**

- 即不加锁读，读取记录的快照版本而非最新版本，通过MVCC实现；

- 在快照读读情况下，mysql通过mvcc来避免幻读。

**当前读**

- 即加锁读，读取记录的最新版本，会加锁保证其他并发事务不能修改当前记录，直至获取锁的事务释放锁；

- 在当前读读情况下，mysql通过next-key来避免幻读。



## 例子

```sql
select * from table where id = ?
select * from table where id < ?
select * from table where id = ? lock in share mode
select * from table where id < ? lock in share mode
select * from table where id = ? for update
select * from table where id < ? for update
```

需要根据事务隔离级别、索引类型进行判断。

### RC/RU 隔离级别 + 条件非索引  ||  RC/RU 隔离级别 + （非）聚簇索引

```sql
(1)select * from table where num = 200
不加任何锁，是快照读。

(2)select * from table where num > 200
不加任何锁，是快照读。

(3)select * from table where num = 200 lock in share mode
当num = 200，有两条记录。这两条记录对应的pId=2，7，因此在pId=2，7的聚簇索引上加行级S锁，采用当前读。
(非聚簇索引时，两个索引都加上锁)

(4)select * from table where num > 200 lock in share mode
当num > 200，有一条记录。这条记录对应的pId=3，因此在pId=3的聚簇索引上加上行级S锁，采用当前读。

(5)select * from table where num = 200 for update
当num = 200，有两条记录。这两条记录对应的pId=2，7，因此在pId=2，7的聚簇索引上加行级X锁，采用当前读。

(6)select * from table where num > 200 for update
当num > 200，有一条记录。这条记录对应的pId=3，因此在pId=3的聚簇索引上加上行级X锁，采用当前读。
```



### RR/Serializable 隔离级别 + 非索引

```sql
(1)select * from table where num =(或>) 200
在RR级别下，不加任何锁，是快照读。
在Serializable级别下，在pId = 1,2,3,7（全表所有记录）的聚簇索引上加S锁。
并且在聚簇索引的所有间隙(-∞,1)(1,2)(2,3)(3,7)(7,+∞)加gap lock

(2)select * from table where num =(或>) 200 lock in share mode
在pId = 1,2,3,7（全表所有记录）的聚簇索引上加S锁。
并且在聚簇索引的所有间隙(-∞,1)(1,2)(2,3)(3,7)(7,+∞)加gap lock

(3)select * from table where num =(或>) 200 for update
在pId = 1,2,3,7（全表所有记录）的聚簇索引上加X锁。
并且在聚簇索引的所有间隙(-∞,1)(1,2)(2,3)(3,7)(7,+∞)加gap lock
```

### RR/Serializable 隔离级别 + 聚簇索引

如果`where`后的条件为精确查询(`=`的情况)，那么只存在record lock。如果为范围查询(`>`或`<`的情况)，那么存在的是record lock + gap lock。

### RR/Serializable 隔离级别 + 非聚簇索引

#### 唯一索引

与聚簇索引类似，两个索引都加锁。

#### 普通索引

通过索引进行精确查询以后，不仅存在 record lock，还存在 gap lock。