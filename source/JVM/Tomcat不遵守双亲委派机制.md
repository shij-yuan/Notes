# Tomcat 中的隔离与双亲委派机制

## Tomcat的隔离

一个WEB容器可以部署多个应用。如果应用各自使用的包相同，但是版本不同，就必须做到应用之间隔离，否则他们的全限定类名如果一样，使用双亲委派机制会有问题。

## 解决

不完全违背双亲委派机制。Tomcat中各个web应用自己的类加载器 (WebAppClassLoader) 会优先加载，如果加载不到，则交给 CommonClassLoader 继续双亲委派机制。

## 双亲委派机制

![](http://upload-images.jianshu.io/upload_images/4236553-c65e628b05bddb2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Tomcat 的双亲委派机制

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ggjlu4c9lyj30cu0i73yn.jpg)