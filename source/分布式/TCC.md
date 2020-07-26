# TCC

TCC事务机制相对于传统事务机制（X/Open XA Two-Phase-Commit），其特征在于它不依赖资源管理器(RM)对XA的支持，而是通过对（由业务系统提供的）业务逻辑的调度来实现分布式事务。

**1. 初步操作（Try）**

TCC事务机制以初步操作（Try）为中心的，确认操作（Confirm）和取消操作（Cancel）都是围绕初步操作（Try）而展开。因此，Try阶段中的操作，其保障性是最好的，即使失败，仍然有取消操作（Cancel）可以将其不良影响进行回撤。

**2. 确认操作（Confirm）**

确认操作（Confirm）是对初步操作（Try）的一个补充。当TCC事务管理器决定commit全局事务时，就会逐个执行初步操作（Try）指定的确认操作（Confirm），将初步操作（Try）未完成的事项最终完成。

**3. 取消操作（Cancel）**

取消操作（Cancel）是对初步操作（Try）的一个回撤。当TCC事务管理器决定rollback全局事务时，就会逐个执行初步操作（Try）指定的取消操作（Cancel），将初步操作（Try）已完成的事项全部撤回。



1. 在全局事务决定提交时，调用与try业务逻辑相对应的confirm业务逻辑
2. 在全局事务决定回滚时，调用与try业务逻辑相对应的cancel业务逻辑



## 与二阶段提交的区别

1. 2PC机制的业务阶段 == TCC机制的try业务阶段 （与全局事务处理无关）
2. 2PC机制的提交阶段（prepare & commit） ==  TCC机制的提交阶段（confirm）
3. 2PC机制的回滚阶段（rollback） ==  TCC机制的回滚阶段（cancel）

