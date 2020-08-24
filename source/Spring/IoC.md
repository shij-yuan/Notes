# IoC

**依赖倒置原则**：高层决定底层，底层依赖高层。高层不能依赖底层，如果底层需要修改，则都要修改。

**控制反转**（IOC）：是依赖倒置原则的思路。

**依赖注入**：实现了控制反转。把底层类作为参数传入上层类，实现上层类对下层类的“控制”。setter 注入和构造函数注入。

对于spring框架来说，就是由spring来负责控制对象的生命周期和对象间的关系。

# AOP

- 横切关注点：重复且类似的代码，如权限认证、日志、事务处理

- 核心关注点：主要的业务。

AOP的作用在于分离系统中的各种关注点，减少系统的重复代码，降低模块间的耦合度。

```java
// 环绕通知
@Around("pointcut()")
public Object around(ProceedingJoinPoint pjp) throws Throwable {
    Object result;
    System.out.println("---------------@Around前----------------");
    result = pjp.proceed();
    System.out.println("---------------@Around后----------------");
    return result;
}
```

