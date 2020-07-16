# cookie和session



## cookie

Cookie是客户端保存用户信息的一种机制，服务器使用 response 向客户端浏览器颁发一个Cookie。Cookie是服务器在**本地机器**上存储的一小段文本，（该头部包含了sessionId）并随着每次请求发送到服务器。

```java
@Override
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    UserInfoTo userInfoTo = threadLocal.get();
    if(!userInfoTo.isTempUser()){
        Cookie cookie = new Cookie(CartConstant.TEMP_USER_COOKIE_NAME, userInfoTo.getUserKey());
        cookie.setDomain("emall.com");
        cookie.setMaxAge(CartConstant.TEMP_USER_COOKIE_TIMEOUT);
        response.addCookie(cookie);
    }
}
```



## session

session是依赖Cookie实现的。服务器使用session把用户的信息临时保存在了服务器上。服务器根据 sessionid 获取出会话中存储的信息，然后确定会话的身份信息。

```java
MemberRespVo data;
session.setAttribute(AuthConstant.LOGIN_USER, data);
```

### 取出信息

```java
HttpSession session = request.getSession();
MemberRespVo member = (MemberRespVo) session.getAttribute(AuthConstant.LOGIN_USER);
```



## 流程

1. 客户端会发送一个http请求到服务器端
2. 服务器端接受客户端请求后，发送一个http响应到客户端，这个响应头，其中就包含Set-Cookie头部
3. 客户端保存cookie
4. 客户端发起的第二次请求，在请求头中携带保存的cookie

## token

### 流程

1. 用户登录，服务器返回Token给客户端。
2. 客户端收到 token 后保存在客户端
3. 客户端再次访问服务器，携带 token
4. 服务器端采用filter过滤器校验。校验成功则返回请求数据，校验失败则返回错误码

## 区别

token可以防止**跨站请求伪造**。

在 HTTP 请求中以参数的形式加入一个随机产生的 token，并在服务器端建立一个拦截器来验证这个 token。

token 可以在用户登陆后产生并放于 session 之中，然后在每次请求时把 token 从 session 中拿出，与请求中的 token 进行比对



## 分布式下session共享问题

- 哈希一致性（ip hash）

使每次请求落在同一服务器

- 统一存储 session （redis + springSession）

### 子域名

自定义 session

```java
@Bean
    public CookieSerializer cookieSerializer(){
        DefaultCookieSerializer cookieSerializer = new DefaultCookieSerializer();
        cookieSerializer.setDomainName("127.0.0.1");
        cookieSerializer.setCookieName("EMALLSESSION");
        return cookieSerializer;
    }
```

### spingSession

装饰者模式