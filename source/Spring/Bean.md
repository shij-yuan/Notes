# Bean

## 作用域

### singleton

Spring IoC容器中只会存在一个**共享的 bean 实例**，并且所有对 bean 的请求，只要 id 与该 bean 定义相匹配，则只会返回bean的同一实例。

### prototype

**一个 bean 定义对应多个对象实例。** **prototype 作用域的 bean 会导致在每次对该 bean 请求**（将其注入到另一个 bean 中，或者以程序的方式调用容器的 getBean( ) 方法）时都会创建一个新的 bean 实例。

### request

只适用于Web程序，每一次 HTTP 请求都会产生一个新的bean，同时该bean**仅在当前HTTP request内有效**，当请求结束后，该对象的生命周期即告结束。

### session

只适用于Web程序，session 作用域表示该针对每一次 HTTP 请求都会产生一个新的 bean，同时该 bean 仅在当前 HTTP session 内有效。



## 初始化

1. 从缓存中获取对象（如果有则说明已初始化，直接返回）
2. 获得 `beanDefinition`，初始化依赖的 Bean
3. 再次从缓存中检查是否已经初始化
4. 实例化 Bean（赋值）
    1. 加载 Bean class
    2. 调用类的构造方法生成对象
    3. 将依赖注入的信息注入对象
    4. 向缓存中加入缓存，解决循环引用
5. 初始化 Bean （分配内存空间，引用对象指向该地址）



## 生命周期

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gizdootjx4j30us0cin1m.jpg)

1. 实例化 bean 对象，类似于 new XXObject( )
2. 将配置文件中配置的属性填充到刚刚创建的 bean 对象中。
3. 检查 bean 对象是否实现了 Aware 一类的接口，如果实现了则把相应的依赖设置到 bean 对象中。比如如果 bean 实现了 BeanFactoryAware 接口，Spring 容器在实例化bean的过程中，会将 BeanFactory 容器注入到 bean 中。
4. 调用 BeanPostProcessor 前置处理方法，即 postProcessBeforeInitialization(Object bean, String beanName)。
5. 检查 bean 对象是否实现了 InitializingBean 接口，如果实现，则调用 afterPropertiesSet 方法。或者检查配置文件中是否配置了 init-method 属性，如果配置了，则去调用 init-method 属性配置的方法。
6. 调用 BeanPostProcessor 后置处理方法，即 postProcessAfterInitialization(Object bean, String beanName)。我们所熟知的 AOP 就是在这里将 Adivce 逻辑织入到 bean 中的。
7. 注册 Destruction 相关回调方法。
8. bean 对象处于就绪状态，可以使用。
9. 应用上下文被销毁，调用注册的 Destruction 相关方法。如果 bean 实现了 DispostbleBean 接口，Spring 容器会调用 destroy 方法。如果在配置文件中配置了 destroy 属性，Spring 容器则会调用 destroy 属性对应的方法。



## 依赖

Spring 单例对象：对象实例化 -> 对象属性填充 -> 对象初始化

### 循环依赖

**构造器注入**和**prototype**类型的属性注入都会初始化Bean失败。**单例**的属性注入可以成功。

#### 缓存

- 一级缓存：保存所有的 singletonBean 的实例
- 二级缓存：保存所有早期创建的Bean对象，这个Bean还没有完成依赖注入
- 三级缓存：singletonBean的生产工厂

getSingleton()过程：Spring首先从一级缓存singletonObjects中获取。如果获取不到，并且对象正在创建中，就再从二级缓存earlySingletonObjects中获取。如果还是获取不到且允许singletonFactories通过getObject()获取，就从三级缓存singletonFactory.getObject() 获取，如果获取到了则从三级缓存移动到二级缓存。

### 解决

1. A首先完成了初始化的第一步，并且将自己提前曝光到singletonFactories中
2. A发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被创建，所以走 create B 流程
3. B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存singletonObjects(没有，A还没初始化完全)，尝试二级缓存earlySingletonObjects（没有），尝试三级缓存singletonFactories，由于A通过ObjectFactory将自己提前曝光了，所以B能够通过ObjectFactory.getObject 拿到A对象，B拿到A对象后顺利完成了初始化阶段，完全初始化之后将自己放入到一级缓存singletonObjects中。
4. A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，进去了一级缓存singletonObjects 中
5. B 获得 A 的引用，也完成初始化



## BeanFactory 与 FactoryBean

- BeanFactory是个Factory，也就是IOC容器或对象工厂；所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的。**工厂模式**
- FactoryBean是个Bean，是能生产或者修饰对象生成的工厂Bean。**装饰器模式**