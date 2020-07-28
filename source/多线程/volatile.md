# volatile

**不保证原子性**

### 保证可见性

一个线程修改了volatile修饰的变量，当修改写回主内存时，另外一个线程立即看到最新的值。

volatile关键字修饰的共享变量在转换成汇编语言时，会加上一个以lock为前缀的指令，当CPU发现这个指令时，立即做两件事：

1. 将当前内核高速缓存行的数据立刻回写到内存
2. 使在其他内核里缓存了该内存地址的数据无效
    - MESI协议，当CPU写数据时，如果发现操作的变量是共享变量，即在其他CPU中也存在该变量的副本，那么他会发出信号通知其他CPU将该变量的缓存行设置为无效状态。写回操作时要经过总线传播数据，而其他处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置为无效状态，当处理器要对这个值进行修改的时候，会强制重新从系统内存里把数据读到处理器缓存。

### 禁止指令重排

编译器会在生成指令系列时在适当的位置会插入`内存屏障`指令来禁止特定类型的处理器重排序。

- LoadLoad屏障：对于 Load1; LoadLoad; Load2， 在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。 

- StoreStore屏障：对于 Store1; StoreStore; Store2， 在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。 

- LoadStore屏障：对于 Load1; LoadStore; Store2， 在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。

- StoreLoad屏障：对于 Store1; StoreLoad; Load2， 在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。



## volatile和synchronized的区别

- synchronized 可以保证原子性，volatile 不能保证原子性
- volatile只能修饰变量，synchronized只能修饰方法和语句块
- 都可以保证可见性，但实现原理不同，volatile对变量加了lock，synchronized使用monitorEnter和monitorExit