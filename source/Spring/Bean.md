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



## 生命周期

![](https://blog-pictures.oss-cn-shanghai.aliyuncs.com/bean实例化过程.png)

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

如果某个 bean 对象依赖另一个 bean 对象，此时就不能直接设置了。Spring 容器首先要先去实例化 bean 依赖的对象，实例化好后才能设置到当前 bean 中。

### 循环依赖

- 如果依赖靠**构造器方式注入**，则无法处理，Spring 直接会报循环依赖异常。