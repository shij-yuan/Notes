# ConcurrentHashMap



## JDK 1.8

使用 CAS + Synchronized

### 变量 sizeCtl

- 未初始化：
    - sizeCtl = 0：表示没有指定初始容量。
    - sizeCtl > 0：表示初始容量。

- 初始化中：
    - sizeCt l= -1,标记作用，告知其他线程，正在初始化。当前线程放弃 CPU 时间片，自旋
- 正常状态：
    - sizeCtl = 0.75 n ,扩容阈值
- 扩容中:
    - sizeCtl < 0 : 表示有其他线程正在执行扩容
    - sizeCtl = (resizeStamp(n) << RESIZE_STAMP_SHIFT) + 2 :表示此时只有一个线程在执行扩容

### ForwardingNode节点

1. 标记作用，表示其他线程正在扩容，并且此节点已经扩容完毕
2. 关联了nextTable,扩容期间可以通过find方法，访问已经迁移到了nextTable中的数据

### nextTable

扩容期间，将table数组中的元素迁移到 nextTable



### 初始化（CAS）

1. sizeCtl = -1 说明有程序正在初始化，则当前线程放弃 CPU 时间片，自旋
   
    - 若不为-1，则通过 CAS 把 sizeCtl 的值设置为-1，表明当前线程正在进行表的初始化，其它失败的线程就会自旋
2. 如果 sizeCtl 大于0，则初始化容量为 sizeCtl，否则返回默认容量 16



### put

1. 计算出桶的位置，若当前桶为空，则通过 CAS 原子操作，把新节点插入到此位置

    - 若所在桶不为空，则判断节点的 hash 值是否为 MOVED（值是-1），若为-1，说明当前数组正在进行扩容，则需要当前线程帮忙迁移数据
2. 开始put：使用**synchronized**给桶中第一个节点对象加锁
    - 从头结点开始遍历，如果找到了和当前 key 相同的节点，则用新值替换旧值。若遍历到了尾结点，则把新节点尾插进去
3. 如果节点个数大于等于 8，则转化为红黑树。
4. addCount：给元素个数加 1，并有可能会触发**扩容**



### **扩容**

#### 扩容条件

1. 当前容量超过阈值
2. 当链表中元素个数超过默认8个，但数组的大小还未超过64的时候，进行数组的扩容

**扩容顺序**：先插入数据，再判断容量



#### 扩容过程：transfer

1. 线程执行put操作，addCount + 1 后发现容量已经达到扩容阈值，进行扩容操作
2. 扩容线程A 最少负责扩容16个桶，以 CAS 的方式修改transferIndex（32 - 16），然后按照降序迁移table[31]--table[16]这个区间的hash桶
3. 迁移hash桶时，会将桶内的链表或者红黑树，拆分成2份，将其插入nextTable[ i ]和nextTable[ i + oldcap ]
4. 迁移完毕的hash桶,会被设置成ForwardingNode节点，以此告知访问此桶的其他线程，此节点已经迁移完毕。
5. 线程B访问到了ForwardingNode节点，如果线程2执行写操作，那么就会先帮其扩容。如果读方法，则会调用ForwardingNode的 find 方法，去 nextTable 里面查找相关元素。



### addCount() ：增加一个元素后数值+1

让这些竞争的线程，分散到不同的对象里边，单独操作它自己的数据(计数变量)，用这样的方式尽量降低竞争。到最后需要统计 size 的时候，再把所有对象里边的计数相加

当需要修改元素数量时，线程会先去 CAS 修改 baseCount 加1，若成功即返回。若失败，则线程被分配到某个 CounterCell ，然后操作 value 加1。若成功，则返回。否则，给当前线程重新分配一个 CounterCell，再尝试给 value 加1

### transfer() 帮助迁移数据

- 所有线程都遵循从后向前进行迁移
- 当前线程迁移时会确定一个范围，限定它此次迁移的数据范围



## JDK 1.7

使用 reentrantLock

### put

1. 通过哈希算法计算出当前 key 的 hash 值
2. 通过这个 hash 值找到它所对应的 Segment 数组的下标
3. 再通过 hash 值计算出它在对应 Segment 的 HashEntry数组 的下标
4. 找到合适的位置插入元素

通过tryLock尝试加锁，如果加锁成功，返回null，否则执行 scanAndLockForPut 方法