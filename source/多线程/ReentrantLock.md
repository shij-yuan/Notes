# ReentrantLock

## 公平锁

1. 如果当前锁状态`state`为 0，说明锁处于闲置状态可以被获取，

2. **首先调用`hasQueuedPredecessors`方法判断当前线程是否还有前节点 (prev node) 在等待获取锁**。

    如果有，则直接返回 false；如果没有，通过调用`compareAndSetState`（CAS）修改 state 值来标记自己已经拿到锁。

    CAS 执行成功后调用`setExclusiveOwnerThread`设置锁的持有者为当前线程。程序执行到现在说明锁获取成功，返回 true；

3. 如果当前锁状态`state`不为 0，但当前线程已经持有锁（`current == getExclusiveOwnerThread()`），由于锁是可重入（多次获取）的，则更新重入后的锁状态`state += acquires` 。锁获取成功 返回 true。

## 非公平锁

1. 调用 compareAndSetState 方法抢占式加锁，加锁成功则将自己设为持锁线程，并返回
2. 若加锁失败，则调用 acquire 方法，将线程置于同步队列尾部进行等待
3. 线程在同步队列中成功获取锁，则将自己设为持锁线程后返回
4. 若同步状态不为0，且当前线程为持锁线程，则执行重入逻辑

## 条件

### await( )

1.  首先判断线程是否被中断，如果是，直接抛出`InterruptedException`，否则进入下一步；
2.  添加当前线程到条件队列中，然后释放全部资源 / 锁;
3.  如果当前节点不在等待队列中，调用`LockSupport.park()`阻塞当前线程，直到被`unpark`或被中断。这里先简单说一下`signal`方法，在线程接收到 signal 信号后，unpark 当前线程，并把当前线程转移到等待队列中（sync queue）。所以，在当前方法中，如果线程被解除阻塞（unpark），也就是说当前线程被转移到等待队列中，就会跳出`while`循环，进入下一步；
4.  线程进入等待队列后，调用`acquireQueued`方法获取锁；
5.  调用`unlinkCancelledWaiters`方法检查条件队列中已经取消的节点，并解除它们的链接（这些取消的节点在随后的垃圾收集中被回收掉）；
6.  逻辑处理结束，最后处理中断（抛出`InterruptedException`或把忽略的中断补上）。

### signal( )

1.  获取条件队列的首节点，解除首节点的链接（`first.nextWaiter = null;`）；
2.  调用`transferForSignal`把条件队列的首节点转移到等待队列的尾部。在`transferForSignal`中，转移节点后，转移的节点没有前继节点，说明当前最后一个等待线程，直接调用`unpark()`唤醒当前线程。



# CountDownLatch

## await( )

使线程进入等待状态，直到计数器减至0，或者线程被中断。当计数器为0时，调用此方法将会立即返回，不会被阻塞住。

## countDown

将计数器进行自减操作，当计数器为0时，唤醒正在同步队列中等待的线程(调用 AQS 中的 releaseShared 方法)



# CyclicBarrier

线程运行到此处的线程都会被屏障挡住，并进入等待状态。直到最后一个线程达到屏障时，所以被阻塞的线程才能继续执行。

## 

