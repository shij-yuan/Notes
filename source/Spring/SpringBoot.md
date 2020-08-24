# SpringBoot

## @SpringBootApplication

包含三个注解

### @SpringBootConfiguration

来源于 @Configuration，功能都是将当前类标注为配置类，并将当前类里以 @Bean 注解标记的方法的实例注入到srping容器中，实例名即为方法名。

### @EnableAutoConfiguration

启用自动配置，其可以帮助 SpringBoot 应用将所有符合条件的 @Configuration 配置都加载到当前 IoC 容器之中。

1. 从 ClassPath下扫描所有的 META-INF/spring.factories 配置文件
2. 通过反射机制实例化 spring.factories 文件中的 @EnableAutoConfiguration 对应的配置项，变成对应标注了 @Configuration 的形式的IoC容器配置类，然后注入IoC容器。

### @ComponentScan

将一些标注了特定注解的bean定义（@Controller，@Component）批量采集注册到Spring的IoC容器之中。



## SpringApplication 实例的初始化

1. 推断应用的类型：判断是 REACTIVE应用、SERVLET应用、NONE 三种中的哪一种
2. 使用 `SpringFactoriesLoader`查找并加载 classpath下 `META-INF/spring.factories`文件中所有可用的 `ApplicationContextInitializer`
3. 使用 `SpringFactoriesLoader`查找并加载 classpath下 `META-INF/spring.factories`文件中的所有可用的 `ApplicationListener`
4. 推断并设置 main方法的定义类



## SpringApplication 的 run() 方法

### 主要流程

- 第一部分进行SpringApplication的初始化模块，配置一些基本的环境变量、资源、构造器、监听器；
- 第二部分实现了应用具体的启动方案，包括启动流程的监听模块、加载配置环境模块、及核心的创建上下文环境模块；
- 第三部分是自动化配置模块
    1. 获取传入的配置工厂类名（`xxxAutoConfiguration`）、类加载器
    2. 通过类加载器获取指定的 spring.factories 文件
    3. 获取文件中工厂类全类名
    4. 反射实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器，得到工厂类 class 对象、构造方法
    5. 生成工厂类实例

### 具体步骤

1. 通过 `SpringFactoriesLoader` 加载 `META-INF/spring.factories` 文件，获取并创建应用的监听器`SpringApplicationRunListener` 对象
2. 然后由 `SpringApplicationRunListener` 来发出 starting 消息
3. 创建参数，并配置当前 SpringBoot 应用将要使用的环境 Environment
4. 完成之后，依然由 `SpringApplicationRunListener` 来发出 environmentPrepared 消息
5. 创建 `ApplicationContext`：应用上下文类，其主要继承了beanFactory
6. 初始化 `ApplicationContext`，并设置 Environment，加载相关配置（将listeners、environment、applicationArguments、banner等重要组件与上下文对象关联）
7. 由 `SpringApplicationRunListener` 来发出 `contextPrepared` 消息，告知SpringBoot 应用使用的 `ApplicationContext` 已准备好
8. 将各种 beans 装载入 `ApplicationContext`，继续由 `SpringApplicationRunListener` 来发出 contextLoaded 消息，告知 SpringBoot 应用使用的 `ApplicationContext` 已装填好
9. refresh ApplicationContext，完成IoC容器可用的最后一步
10. 由 `SpringApplicationRunListener` 来发出 started 消息
11. 完成最终的程序的启动
12. 由 `SpringApplicationRunListener` 来发出 running 消息，告知程序已运行起来了

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gh2fqho1nbj30qo0qpdnc.jpg)

### 自动配置流程

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gh2g48g7lgj30tn0qodjv.jpg)





@EnableAutoConfiguration 图示

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gh2ffr3zb2j30f30lldg8.jpg)

