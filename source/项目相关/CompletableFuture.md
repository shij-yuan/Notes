# CompletableFuture

CompletableFuture实现了CompletionStage和Future接口。

Future获取结果时，只能通过`get()`方法**阻塞**线程或者通过轮询`isDone()`的方式来获取任务结果，消耗CPU资源，而且还不能及时的获取任务结果。而CompletableFuture则不会，它可以通过触发**异步**方法来获取结果。



## 使用

1. 编写异步任务

当一个线程依赖另一个线程时，可以使用 thenApply 方法来把这两个线程串行化

- `runAsync` 方法不支持返回值
- `supplyAsync` 可以支持返回值
- `thenAcceptAsync` 依赖`supplyAsync` 的返回结果

```java
@Autowired
ThreadPoolExecutor executor;

public xxx xxx {
    CompletableFuture<SkuInfoEntity> infoFuture = CompletableFuture.supplyAsync(() -> {
        // 必须先得到的数据
    }, executor);

    CompletableFuture<Void> saleAttrFuture = infoFuture.thenAcceptAsync((res) -> {
        // thenAcceptAsync() 依赖 infoFuture 获得的数据 
    }, executor);
    
    CompletableFuture<Void> imageFuture = CompletableFuture.runAsync(() -> {
        // imageFuture 不依赖上面的数据，执行自己的任务
    }, executor);
}
```

2. 阻塞并等待任务完成

```java
CompletableFuture.anyOf(saleAttrFuture, descFuture, baseAttrFuture, imageFuture).get();
```



## 过程

1. 主线程调用CompletableFuture的supplyAsync()方法，传入Supplier和Executor。在supplyAsync()中又继续调用CompletableFuture的`asyncSupplyStage(Executor, Supplier)`方法。然后调用指定的线程池，并执行`execute(new AsyncSupply(d,f))`，这里d就是“源任务”。AsyncSupply实现了Runnable的`run()`方法，先判断`d.result == null`，判断该任务是否已经完成，防止并发情况下其他线程完成此任务了
2. 源任务未完成时，main线程会在`asyncSupplyStage()`方法中返回“依赖任务”（阻塞中），接下来main线程会继续执行CompletableFuture的`thenAccept(Comsumer<? super T> action)`方法，然后调用`CompletableFuture的uniAcceptStage()`方法。
3. 在uniAcceptStage()方法中，会将“依赖任务”、“源任务”、线程池以及Comsumer报装程一个UniAccept对象，然后调用push()压入stack的栈顶中。随后调用UniAccept的tryFire()方法。
4. CompletableFuture的`uniAccept()`方法会判断 a.result 是否为空（a就是“源任务”），完成后将结果存入 result 。
5. 最后调用`postComplete()`，触发其他依赖任务。