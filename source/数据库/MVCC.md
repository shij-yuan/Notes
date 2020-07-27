# MVCC

每一行记录都有两个隐藏列：`DATA_TRX_ID`、`DATA_ROLL_PTR`。

1. 获得事务 ID
2. 获得 read view
3. 查询到数据，与 read view 中的事务 ID 进行比较
    - 符合要求则返回
    - 不符合要求则从 undo log 链中寻找

## DATA_TRX_ID

记录最近更新这条行记录的`事务 ID`，大小为 `6` 个字节

## **DATA_ROLL_PTR**

表示指向该行回滚段`（rollback segment）`的指针，大小为 `7` 个字节，`InnoDB` 便是通过这个指针找到之前版本的数据。该行记录上所有旧版本，在 `undo` 中都通过链表的形式组织。

## **DB_ROW_ID**

行标识（隐藏单调自增 `ID`），大小为 `6` 字节，如果表**没有主键**，`InnoDB` 会自动生成一个隐藏主键，因此会出现这个列。

## undo log 链

Update 操作中：

1. 对行记录加排他锁
2. 把该行原本的值拷贝到 `undo log` 中，`DB_TRX_ID` 和 `DB_ROLL_PTR` 都不变
3. 修改该行的值这时产生一个新版本，更新 `DATA_TRX_ID` 为修改记录的事务 `ID`
4. 将 `DATA_ROLL_PTR` 指向刚刚拷贝到 `undo log` 链中的旧版本记录。如果对同一行记录执行连续的 `UPDATE`，`Undo Log` 会组成一个链表，遍历这个链表可以看到这条记录的变迁
5. 记录 `redo log`，包括 `undo log` 中的修改



## read view

在 `RR` 隔离级别下，每个事务 `touch first read` 时（本质上就是事务开始后执行的第一个 `SELECT` 语句时，后续所有的 `SELECT` 都是复用这个 `ReadView`，其它 `update`, `delete`, `insert` 语句和一致性读 `snapshot` 的建立没有关系），会**将当前系统中的所有的活跃事务拷贝到一个列表生成`ReadView`**。

在 `RC` 隔离级别下，每个 `SELECT` 语句开始时，都会重新将当前系统中的所有的活跃事务拷贝到一个列表生成 `ReadView`。二者的区别就在于生成 `ReadView` 的时间点不同，一个是事务之后第一个 `SELECT` 语句开始、一个是事务中每条 `SELECT` 语句开始。

## 判断可见性

`ReadView` 中是当前活跃的事务 `ID` 列表，称之为 `m_ids`，其中最小值为 `up_limit_id`，最大值为 `low_limit_id`，事务 `ID` 是事务开启时 `InnoDB` 分配的，其大小决定了事务开启的先后顺序，因此我们可以通过 `ID` 的大小关系来决定版本记录的可见性。

- 如果被访问版本的 `trx_id` **小于** 最小值 `up_limit_id`，说明生成该版本的事务在 `ReadView` **生成前就已经提交了**，所以该版本可以被当前事务访问。
- 如果被访问版本的 `trx_id` **大于** 最大值 `low_limit_id`，说明生成该版本的事务在生成 `ReadView` 后才生成，所以该版本不可以被当前事务访问。需要根据 `Undo Log` 链找到前一个版本，然后根据该版本的 DB_TRX_ID **重新判断**可见性。
- 如果被访问版本的 `trx_id` 在最大值和最小值之间（包含），那就需要判断一下 `trx_id` 的值是不是在 `m_ids` 列表中。
    - 如果在，说明创建 `ReadView` 时生成该版本所属事务还是**活跃**的，因此该版本**不可以被访问**，需要查找 Undo Log 链得到上一个版本，然后根据该版本的 `DB_TRX_ID` 再从头计算一次可见性；
    - 如果不在，说明创建 `ReadView`时生成该版本的事务**已经被提交**，该版本**可以被访问**。