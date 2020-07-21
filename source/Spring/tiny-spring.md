# tiny-spring

## IoC

### BeanDefinition

在spring中，每个bean都是以BeanDefinition的形式存在的。解决 `Bean` 的具体定义问题：**如何在 IoC 容器中定义一个 `Bean`，使得 IoC 容器可以根据这个定义来生成实例**。

包括 `Bean` 的 **名字** `String beanClassName`、**类型**`Class beanClass`、**属性** `PropertyValues propertyValues`。根据其 **类型** 可以生成一个类实例，然后可以把 **属性** 注入进去。`propertyValues` 里面包含了一个个 `PropertyValue` 条目，每个条目都是键值对 `String` - `Object`，分别对应要生成实例的属性的**名字**与**类型**。如果类型是 `BeanReference` 类型，则说明其是一个引用，其中保存了引用的名字，需要用先进行解析，转化为对应的实际 `Object`。

### BeanFactory

解决 IoC 容器在 **已经获取 `Bean` 的定义的情况下，如何装配、获取 `Bean` 实例** 。

IoC 容器的结构：`AbstractBeanFactory` 维护一个 `beanDefinitionMap` 哈希表用于保存类的定义信息（`BeanDefinition`）。获取 `Bean` 时，如果 `Bean` 已经存在于容器中，则返回之，否则则调用 `doCreateBean`方法装配一个 `Bean`。（所谓存在于容器中，是指容器可以通过 `beanDefinitionMap` 获取 `BeanDefinition` 进而通过其 `getBean()` 方法获取 `Bean`。）

### AutowireCapableBeanFactory

可以实现自动装配的 `BeanFactory`。在这个工厂中，实现了 `doCreateBean` 方法，该方法分三步：1，通过 `BeanDefinition` 中保存的类信息实例化一个对象；2，把对象保存在 `BeanDefinition` 中，以备下次获取；3，为其装配属性。装配属性时，通过 `BeanDefinition` 中维护的 `PropertyValues` 集合类，把 `String` - `Value` 键值对注入到 `Bean` 的属性中去。如果 `Value` 的类型是 `BeanReference`则说明其是一个引用（对应于 XML 中的 `ref`），通过 `getBean` 对其进行获取，然后注入到属性中。

AbstractBeanFactory的 registerBeanDefinition 方法是为了将 XmlBeanDefinitionReader 读取到的beanDefinition 注册进来。

#### getBean: 生成 `Bean`并初始化

1. `doCreateBean` ：实例化 `Bean`。 
    - `createInstance` ：生成一个新的实例。 
    -  `applyProperties` ：注入属性，包括依赖注入的过程。在依赖注入的过程中，如果 `Bean` 实现了 `BeanFactoryAware` 接口，则将容器的引用传入到 `Bean` 中去，这样，`Bean` 将获取对容器操作的权限，也就允许了 **编写扩展 IoC 容器的功能的 `Bean`**。
2. `initializeBean(bean)` ： 初始化 `Bean`。 
    - 从 `BeanPostProcessor` 列表中，依次取出 `BeanPostProcessor` 执行 `bean = postProcessBe foreInitialization(bean,beanName)` 。
    - 初始化方法（tiny-spring 未实现对初始化方法的支持）。 
    - 从 `BeanPostProcessor` 列表中， 依次取出 `BeanPostProcessor` 执行其 `bean = postProcessAfterInitialization(bean,beanName)`。

1. 从 map 中根据 bean 的 name 获取 BeanDefinition
2. 如果 BeanDefinition 不存在就抛出异常
3. 获取 beanDefinition 的成员变量 bean
4. 如果 bean 不是 null 直接返回，如果是 null 就调用子类 AutowireCapableBeanFactory 创建 bean

### ApplicationContext

对`Resouce` 、 `BeanFactory`、`BeanDefinition`进行了功能的封装，解决 **根据地址获取 IoC 容器并使用**。

#### 流程

1. `ApplicationContext` 完成了类定义的读取和加载，并注册到 `BeanFactory` 中去。 
2. `ApplicationContext` 从 `BeanFactory` 中寻找 `BeanPostProcessor`，注册到 `BeanFactory` 
    维护的 `BeanPostProcessor` 列表中去。 
3. `ApplicationContext` 以单例的模式，通过主动调用 `getBean` 实例化、注入属性、然后初始化 `BeanFactory` 中所有的 `Bean`。由于所有的 `BeanPostProcessor` 都已经在第 2 步中完成实例化了，因此接下来实例化的是普通 `Bean`，因此普通 `Bean` 的初始化过程可以正常执行。 
4. 调用 `getBean` 时，委托给 `BeanFactory`，此时只是简单的返回每个 `Bean` 单例，因为所有的 `Bean` 实例在第三步都已经生成了。

#### AbstractApplicationContext

`ApplicationContext` 的抽象实现，内部包含一个 `BeanFactory` 类。主要方法有 `getBean()` 和 `refresh()` 方法。`getBean()` 直接调用了内置 `BeanFactory` 的 `getBean()` 方法，`refresh()` 则用于实现 `BeanFactory` 的刷新，也就是告诉 `BeanFactory` 该使用哪个资源（`Resource`）加载类定义（`BeanDefinition`）信息，该方法留给子类实现，用以实现 **从不同来源的不同类型的资源加载类定义** 的效果。

#### ApplicationContext

继承了 `BeanFactory`。通常，要实现一个 IoC 容器时，需要先通过 `ResourceLoader` 获取一个 `Resource`，其中包括了容器的配置、`Bean` 的定义信息。接着，使用 `BeanDefinitionReader` 读取该 `Resource` 中的 `BeanDefinition` 信息。最后，把 `BeanDefinition` 保存在 `BeanFactory` 中，容器配置完毕可以使用。注意到 `BeanFactory` 只实现了 `Bean` 的 **装配、获取**，并未说明 `Bean` 的 **来源** （`BeanDefinition` ）是如何 **加载** 的。

#### refresh方法：从资源文件加载类定义、扩展容器

1. `loadBeanDefinitions(BeanFactory)` ：加载类定义，并注入到内置的 `BeanFactory` 中，这里的可扩展性在于，**未对加载方法进行要求，也就是可以从不同来源的不同类型的资源进行加载**。
2. `registerBeanPostProcessors(BeanFactory)` ：获取所有的 `BeanPostProcessor`，并注册到 `BeanFactory` 维护的 `BeanPostProcessor` 列表去。
3. `onRefresh` ： 
    - `preInstantiateSingletons` ：以单例的方式，初始化所有 `Bean`。tiny-spring 只支持 `singleton` 模式。





## AOP

### 在 `Bean` 初始化过程中完成 AOP 的植入（AutoProxyCreator）

1. `BeanPostProcessor` ：在 `postProcessorAfterInitialization` 方法中，使用动态代理的方式，返回一个对象的代理对象。解决了 **在 IoC 容器的何处植入 AOP** 的问题。
2. `BeanFactoryAware` ：这个接口提供了对 `BeanFactory` 的感知，这样，尽管它是容器中的一个 `Bean`，却可以获取容器的引用，进而获取容器中所有的切点对象，决定对哪些对象的哪些方法进行代理。解决了 **为哪些对象提供 AOP 的植入** 的问题。

### 实现

#### PointcutAdvisor

切点通知器，用于提供 **对哪个对象的哪个方法进行什么样的拦截** 的具体内容。通过它可以获取一个切点对象 `Pointcut` 和一个通知器对象 `Advisor`。

#### Pointcut

切点对象可以获取一个 `ClassFilter` 对象和一个 `MethodMatcher` 对象。前者用于判断是否对某个对象进行拦截（用于 **筛选要代理的目标对象**），后者用于判断是否对某个方法进行拦截（用于 **在代理对象中对不同的方法进行不同的操作**）。

#### Advisor

通知器对象可以获取一个通知对象 `Advice` 。就是用于实现 **具体的方法拦截**，需要使用者编写，也就对应了 Spring 中的前置通知、后置通知、环切通知等。

#### 步骤

1. `AutoProxyCreator`（实现了 `BeanPostProcessor` 接口）在实例化所有的 `Bean` 前，最先被实例化。
2. 其他普通 `Bean` 被实例化、初始化。在初始化的过程中，`AutoProxyCreator` 加载 `BeanFactory` 中所有的 `PointcutAdvisor`（这也保证了 `PointcutAdvisor` 的实例化顺序优于普通 `Bean`），然后依次使用 `PointcutAdvisor` 内置的 `ClassFilter`，判断当前对象是不是要拦截的类。
3. 如果是，则生成一个 `TargetSource`（要拦截的对象和其类型），并取出 `AutoProxyCreator` 的 `MethodMatcher`（对哪些方法进行拦截）、`Advice`（拦截的具体操作），再，交给 `AopProxy` 去生成代理对象。
4. `AopProxy` 生成一个 `InvocationHandler`，在它的 `invoke` 函数中，首先使用 `MethodMatcher` 判断是不是要拦截的方法，如果是则交给 `Advice` 来执行（`Advice` 由用户来编写，其中也要手动/自动调用原始对象的方法），如果不是，则直接交给 `TargetSource` 的原始对象来执行。