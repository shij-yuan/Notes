# Mybatis

## 一级缓存

共享范围是一个SqlSession内部。底层是一个没有容量限定的HashMap。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/bb851700.png)



## 二级缓存

**基于 mapper文件的 namespace**，也就是说多个sqlSession可以共享一个mapper中的二级缓存区域，并且如果两个mapper的namespace相同，即使是两个mapper，那么这两个mapper中执行sql查询到的数据也将存在相同的二级缓存区域中。

二级缓存开启后，同一个namespace下的所有操作语句，都影响着同一个Cache，即二级缓存被多个SqlSession共享，是一个全局的变量。

当开启缓存后，数据的查询执行的流程就是 二级缓存 -> 一级缓存 -> 数据库。

![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/28399eba.png)



## ${} 与 #{}

`#{}`是**预编译处理**，`${}`是字符串替换。

- 处理`#{}`时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值。使用#{}可以有效的防止SQL注入，提高系统安全性。

    预编译：**预编译完成之后，SQL的结构已经固定，即便用户输入非法参数，也不会对SQL的结构产生影响，从而避免了潜在的安全风险。**

- 处理`${}`时，就是把${}替换成变量的值。

