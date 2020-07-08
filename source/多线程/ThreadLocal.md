# ThreadLocal

## 作用

ThreadLocal类是修饰变量的，重点是在控制变量的作用域，在多线程环境下，可以保证各个线程之间的变量互相隔离、相互独立，实现每一个线程都有自己的共享变量

## 结构

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggjlx8h1xej30me0n9402.jpg)

- 每个Thread线程内部都有一个Map。

- Map里面存储线程本地对象（key）和线程的变量副本（value）

- Thread内部的Map是由ThreadLocal维护的，由ThreadLocal负责向map获取和设置线程的变量值。所以对于不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离，互不干扰。



## 使用场景

数据库连接管理，线程会话管理等场景，只适用于**独立变量副本**的情况。

## 内存泄露

由于ThreadLocalMap的key是弱引用，而Value是强引用。这就导致了一个问题，ThreadLocal在没有外部对象强引用时，发生GC时弱引用Key会被回收，而Value不会回收，如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。

### 如何避免泄漏
Key是弱引用，就在调用ThreadLocal的get( )、set( )方法时完成后再调用remove方法，将Entry节点和Map的引用关系移除，这样整个Entry对象在GC Roots分析后就变成不可达了，下次GC的时候就可以被回收。

如果使用ThreadLocal的set方法之后，没有显示的调用remove方法，就有可能发生内存泄露，所以使用完ThreadLocal之后，记得调用remove方法。



## 父子进程

当子线程要获取父进程中的ThreadLocal值，得到的确是null，原因是子线程ThreadLocal调用get方法时的当前线程是子线程，所以取不到父线程的值。

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

### InheritableThreadLocal

```java
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
   
    protected T childValue(T parentValue) {
        return parentValue;
    }

    /**
     * 重写Threadlocal类中的getMap方法，在原Threadlocal中是返回
     *t.theadLocals，而在这么却是返回了inheritableThreadLocals，因为
     * Thread类中也有一个要保存父子传递的变量
     */
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    /**
     * 同理，在创建ThreadLocalMap的时候不是给t.threadlocal赋值
     *而是给inheritableThreadLocals变量赋值
     * 
     */
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
```

如果使用InheritableThreadLocal,那么保存的所有东西都已经不在原来的t.thradLocals里面，而是在一个新的t.inheritableThreadLocals变量中了。

父线程的所有的值都`copy`到子线程中，使得子线程可以获取父线程的内容。在copy过程中是`浅拷贝`，key和value都是原来的引用地址

过程：

1. 在创建InheritableThreadLocal对象的时候赋值给线程的t.inheritableThreadLocals变量
2. 在创建新线程的时候会check父线程中t.inheritableThreadLocals变量是否为null，如果不为null则copy一份ThradLocalMap到子线程的t.inheritableThreadLocals成员变量中去
3. 因为复写了getMap(Thread)和CreateMap()方法,所以get值得时候，就可以在getMap(t)的时候就会从t.inheritableThreadLocals中拿到map对象，从而实现了可以拿到父线程ThreadLocal中的值



## 与Synchronized的对比

- ThreadLocal使用场合主要解决多线程中数据因并发产生不一致问题。

- ThreadLocal和Synchonized都用于解决多线程并发访问。但是ThreadLocal与synchronized有本质的区别:

- synchronized是利用锁的机制，使变量或代码块在某一时该只能被一个线程访问。而**ThreadLocal为每一个线程都提供了变量的副本，使得每个线程在某一时间访问到的并不是同一个对象，这样就隔离了多个线程对数据的数据共享**。而**Synchronized却正好相反，它用于在多个线程间通信时能够获得数据共享**。

- Synchronized用于线程间的数据共享，而ThreadLocal则用于线程间的数据隔离。当然ThreadLocal并不能代替synchronized,它们处理不同的问题域。Synchronized用于实现同步机制，比ThreadLocal更加复杂。



## 参考

https://www.jianshu.com/p/98b68c97df9b

http://www.threadlocal.cn

https://blog.csdn.net/a837199685/article/details/52712547