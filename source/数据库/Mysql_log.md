# Log



## bin log


binlog记录了数据库表结构和表数据变更，比如`update/delete/insert/truncate/create`

作用：**复制和恢复数据**

- MySQL往往都是**一主多从**结构的，从服务器需要与主服务器的数据保持一致
- 数据库的数据出问题，可以通过`binlog`来对数据进行恢复



## redo log （持久性）

记载着这次**在某个页上做了什么修改**

修改的时候，写完内存，但数据还没真正写到磁盘的时候，数据库挂了，就可以根据`redo log`来对数据进行恢复。因为`redo log`是顺序IO，所以**写入的速度很快**，并且`redo log`记载的是物理变化（xxxx页做了xxx修改），文件的体积很小，**恢复速度很快**。



## 区别

`binlog`记载的是`update/delete/insert`这样的SQL语句，而`redo log`记载的是物理修改的内容（xxxx页修改了xxx）。

`redo log` 记录的是数据的**物理变化**，`binlog` 记录的是数据的**逻辑变化**

`redo log`**事务开始**的时候，就开始记录每次的变更信息，而`binlog`是在**事务提交**的时候才记录。



## undo log

作用：回滚和多版本控制(MVCC)

记录相反的操作