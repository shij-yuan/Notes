# Feign



## 请求头丢失

在使用远程调用时，新建了一个 request 请求 ，默认没有拦截器进行处理，导致这个请求没有任何请求头。

需要自定义拦截器，把原来的请求头依次添加到新的请求中。

```java
@Configuration
public class FeignConfig {
    @Bean("requestInterceptor")
    public RequestInterceptor requestInterceptor() {
        return new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate template) {
                // 1、RequestContextHolder 获取当前的请求
                ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                if (attributes != null) {
                    // 原始请求 页面发起的老请求
                    HttpServletRequest request = attributes.getRequest();
                    if (request != null) {
                        // 获取原始请求的头数据 cookie
                        String cookie = request.getHeader("Cookie");
                        // 给feign生成的心请求设置请求头cookie
                        template.header("Cookie", cookie);
                    }
                }
            }
        };
    }
}
```



## 异步时上下文丢失

每一个异步任务，都会启动自己的拦截器。这些拦截器使用 ThreadLocal 获取上下文信息，但是异步中无法获得原始的请求信息，导致空指针异常

```java
	private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
			new NamedThreadLocal<>("Request attributes");
```



应该从主线程中获取请求信息，然后再异步线程中共享

```java
// 主线程中获取请求
RequestAttributes requestAttributes = RequestContextHolder.getRequestAttributes();
CompletableFuture<Void> getAddressTask = CompletableFuture.runAsync(() -> {
    		// 异步任务获取请求信息
            RequestContextHolder.setRequestAttributes(requestAttributes); 
			......
        }, executor);
```

