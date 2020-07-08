# GC日志分析

打开 GC 日志

```
-verbose : gc
-xx:+PrintGC
-XX:+PrintGCDetails
```

`[PSYoungGen: 5986K->696K(8704K)] 5986K-> 704K (9216K)`

中括号内: GC 回收前年轻代大小 -> 回收后大小，( 年轻代总大小)

括号外: GC 回收前年轻代和老年代大小 -> 回收后大小，( 年轻代和老年代总大小)

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gexufzhnpbj31zg0ncno3.jpg)

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gexugf4bvej327a0kktwg.jpg)





### 堆空间分析

JDK 7 ：触发GC 

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggjltfoydcj31by0l2qbv.jpg)

JDK 8 : **4MB直接进入老年代**

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gexuqmfaqpj31100kaq9o.jpg)

