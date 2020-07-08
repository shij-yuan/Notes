# String 的细节

JDK 8 以前内部定义了 `final char[] value`储存字符数据

JDK 9 使用 `byte[]`储存

### 不可变性

String:代表不可变的字符序列。简称:不可变性。

- 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。

- 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值, 不能使用原有的value进行赋值。

- 当调用String的replace ()方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值



```java
String s1 = "abc";
String s2 = "abc";

sout(s1 == s2);
// true
// 字面量定义方式：字符储存在堆的常量池中，s1, s2 指向同一个地址

s2 += "def";
// s1: abc   s2 : abcdef
```



## String 的基本特性

字符串常量池中是不会存储相同内容的字符串的。

- String的String Pool是一个固定大小的Hashtable，默认值大小长度是1009。**如果放进String Pool的String非常多， 就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接导致调用 String. intern 时性能会大幅下降。**

- 使用-XX: StringTableSi ze可设置StringTable的长度

- 在jdk6中StringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。StringTablesize设置没有 要求

- 在jdk7中，StringTable的长 度默认值是60013，1009是 可设置的最小值。



## String 拼接

拼接有变量时，会使用 `StringBuilder`操作：`append`

**sb.append** 效率高于字符串的拼接，后者每次拼接会新建 sb ， 前者只有一个  sb；

sb.append 存在数组扩容，拷贝原来的数据进入新的数组，可以**初始化时直接大容量数组**



## intern()

从字符串常量池中查询字符串是否存在，若不存在则将字符串放进常量池，返回地址。

```java
String s1 = new String("ab");
// 共 2 个对象：
// 1. new 关键字在堆空间创建的 2. 字符串常量池中的对象，字节码： ldc
 0 new #2 <java/lang/String>
 3 dup
 4 ldc #3 <ab>
 6 invokespecial #4 <java/lang/String.<init>>
 9 astore_1
10 return
```

```java
String s2 = new String("a") + new String("b");
// 共 6 对象
1 new StringBuilder()
2 new String("a")
3 常量池中的 a
4 new String("b")
5 常量池中的 b
6 sb 中的 new string("ab") (没有在字符串常量池中创建"ab")

```



### 题目

```java
		String s1 = new String("1");
        s1.intern(); // 字符串常量池已经存在"1"
        String s2 = "1";
        System.out.println(s1 == s2);

        String s3 = new String("1") + new String("1"); 
		// s3变量的地址是 new String("11")
		// 此时字符串常量池中 没有 "11"
        s3.intern();
		// 在字符串常量池中创建字符"11"
		// JDK 6 : 新的对象， 有新的地址
		// JDK 7 : 常量并未创建新对象， 而是创建指向堆空间中  new String("11") 的地址
        String s4 = "11";
        System.out.println(s3 == s4);
```

**JDK 6 : false false**

![](https://tva1.sinaimg.cn/large/007S8ZIlly1geua4j9wbmj316k0loagg.jpg)

**JDK 7/8 : false true**

![](https://tva1.sinaimg.cn/large/007S8ZIlly1geua3mieebj317q0go43c.jpg)





## G1  的 String去重

- 当垃圾收集器工作的时候，会访问堆上存活的对象。对每个访问的对象都会检查是否是候选的要去重的String对象。
- 如果是，把这个对象的一个引用插入到队列中等待后续的处理。-一个去重的线程在后台运行，处理这个队列。处理队列的一个元素意味着从队列删除这个元素，然后尝试去重它引用的String对象。
- 使用一个hashtable来记录所有的被String对象使用的不重复的char数组。
- 当去重的时候，会查这个hashtable，来看堆上是否已经存在一个一模一样的char数组。
- 如果存在，string对象会被调整引用那个数组，释放对原来的数组的引用，最终会被垃圾收集器回收掉
- 如果查找失败，char数组会被插入到hashtable,这样以后的时候就可以共享这个数组了。