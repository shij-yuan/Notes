# OAuth2

## 流程

1. 第三方应用请求用户授权。

2. 用户同意授权（登录），并返回一个凭证（code）

3. 第三方应用通过第二步的凭证（code）向授权服务器请求授权

4. 授权服务器验证凭证（code）通过后，同意授权，并返回一个资源访问的凭证（Access Token）。

5. 第三方应用通过第四步的凭证（Access Token）向资源服务器请求相关资源。

6. 资源服务器验证凭证（Access Token）通过后，将第三方应用请求的资源返回。


![社交登录](https://tva1.sinaimg.cn/large/007S8ZIlly1ggl2689o09j31ej0p1ae7.jpg)