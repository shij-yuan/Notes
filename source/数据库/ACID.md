# ACID



## 原子性 ： undo log

undo log属于逻辑日志，它记录的是sql执行相关的信息。当发生回滚时，InnoDB会根据undo log的内容做与之前相反的工作：对于每个insert，回滚时会执行delete；对于每个delete，回滚时会执行insert；对于每个update，回滚时会执行一个相反的update，把数据改回去



## 持久性：redo log

当数据修改时，除了修改Buffer Pool中的数据，还会在redo log记录这次操作；当事务提交时，会调用fsync接口对redo log进行刷盘。如果MySQL宕机，重启时可以读取redo log中的数据，对数据库进行恢复。redo log采用的是WAL（Write-ahead logging，预写式日志），所有修改先写入日志，再更新到Buffer Pool，保证了数据不会因MySQL宕机而丢失，从而满足了持久性要求

写 redo log 比直接将Buffer Pool中修改的数据写入磁盘(即刷脏)要快：

- 刷脏是随机IO，因为每次修改的数据位置随机，但写redo log是追加操作，属于顺序IO。
- 刷脏是以数据页（Page）为单位的，MySQL默认页大小是16KB，一个Page上一个小修改都要整页写入；而redo log中只包含真正需要写入的部分，无效IO大大减少。



## 隔离性（读：锁； 写：MVCC）

即并发情形下事务之间互不干扰。

- 写操作的影响：锁机制保证隔离性
- 读操作的影响：MVCC保证隔离性



## 一致性 （目标）

- 保证原子性、持久性和隔离性，如果这些特性无法保证，事务的一致性也无法保证
- 数据库本身提供保障：安全方面，例如不允许向整形列插入字符串值、字符串长度不能超过列的限制等
- 应用层面进行保障：逻辑上的