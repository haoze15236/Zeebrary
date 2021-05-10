# 一级缓存

也叫本地会话缓存，默认开启，可以通过全局配置文件setting属性来修改一级缓存作用范围：

- **localCacheScope** 

  **STATEMENT** :表示为语句级别，由于mybatis每次执行语句之后都会关闭statement，所以其实相当于关闭了一级缓存。(经测试，即使使用BATCH执行器，也无法从缓存获取结果)

  **SESSION** :默认级别，开启一个sqlSession之后，在同一个session下执行的相同sql，会通过缓存返回结果。但是存在2种可能导致一级session级别的一级缓存失效

  - 两次相同sql之间执行了增删改的操作,会导致缓存刷新。
  - 手动清楚了一级缓存`sqlSession.clearCache();`

一级缓存的默认实现类为`org.apache.ibatis.cache.impl.PerpetualCache`,可以看到其实内部是维护了一个map集合

```java
private final Map<Object, Object> cache = new HashMap();
```

debugger在getObject()方法可以看到，缓存的key如下：

```
2111334294:4464961771:mybatis.mapper.ExpReportLineMapper.selectLines:0:2147483647:SELECT
            *
        FROM
            exp_report_line l
        where l.EXP_REPORT_LINE_ID = ?:1:development
```

可以看到，key的组成包括了**statement ID**：mybatis.mapper.ExpReportLineMapper.selectLines，**执行的sql** ，**参数值** 等等，所以即使查询的返回结果一致，也有可能因为这些内容不同，无法通过缓存获取。

# [二级缓存](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#cache)

全局作用域缓存，默认是开启的，可以通过全局配置文件setting属性来修改是否启用：

- **cacheEnabled** : 默认为ture，全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。

官方描述的是**所有映射器配置文件中已配置的任何缓存**，所以我们除了开启全局配置之外，还要再映射器配置文件配置，也就是使用`cache`标签说明。

```xml
<mapper namespace="mybatis.mapper.ExpReportLineMapper">
    <cache eviction="FIFO" flushInterval="60000" size="512" readOnly="true"/>
    <select useCache="true" flushCache="true">...</select>
    <update useCache="true" flushCache="true">...</update>
</mapper>
```

其中**readOnly**属性需要注意，当为true的时候,直接读取的是缓存实例，而为false的时候，是通过反序列化返回缓存对象的拷贝，因此<span style="color:red">false时，缓存对象需要实现Serializable接口。</span>

> 关于useCache，flushCache,两个属性，说明可参考官方文档：[select](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#select) 和 [insert, update 和 delete](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#insert_update_and_delete)

官方文档中有这样一个**`提示`** 

> 二级缓存是事务性的。这意味着，当 SqlSession 完成并提交时，或是完成并回滚，但没有执行 flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。

因为flushCache=true的语句被执行会清空一级缓存，**所在namespace的**二级缓存,而insert/delete/update语句默认 flushCache=true。所以上面一级缓存失效有一个原因就是执行了增删改的操作。而当缓存没有被清空的时候，那么只有在SqlSession 完成并提交时，或是完成并回滚的时候，本次sqlSession中的查询的数据才会被缓存进二级缓存。

<span style="color:red">若同时开启一级缓存和二级缓存，那么获取顺序是先从二级缓存中获取，再去一级缓存中获取。</span>

## cache-ref

上面说明了，二级缓存是分不同namespace的，也就是**映射器配置文件**,因此对于关联查询的数据，比如部门表和员工表肯定是两个mapper文件，那么修改了员工数据，并不会刷新部门mapper里面关联查询的员工数据，这就导致了脏读问题，因此可以使用`cache-ref`标签来共享不同映射器配置文件的二级缓存。

```xml
<cache-ref namespace="com.someone.application.data.SomeMapper"/>
```

# 第三方缓存

前面讲到的mybatis二级缓存有一定局限性

- cache-ref只能指定一个缓存关联，实际开发中，关联查询大多不只一个，就存在脏读问题。
- 默认二级缓存是在java应用层面做的，若缓存数据过多，会导致OOM。

因此，我们可以通过整合专业的缓存中间件来解决这个问题。

## mybatis整合redis

