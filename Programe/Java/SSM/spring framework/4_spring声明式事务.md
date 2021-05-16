# spring 声明式事务

## 概念

> 事务是：把一组业务当成一个业务来做；要么都成功，要么都失败，保证业务操作完整性的一种数据库机制。



- 事务的四大特性(ACID):

**A 原子性**：原子性指的是,在一组业务操作下,要么都成功,要么都失败在一组增删改查的业务下,要么都提交,要么都回滚。

**C 一致性**：事务前后的数据要保证数据的一致性在一组的查询业务下，必须要保证前后关联数据的一致性。

**I 隔离性**：在并发情况下,事物之间要相互隔离。

**D 持久性**：数据一旦保存就是持久性的。



- 事务的种类

**编程式事务：**在代码中直接加入处理事务的逻辑，可能需要在代码中显式调用beginTransaction()、commit()、rollback()等事务管理相关的方法。

**声明式事务：**在方法的外部添加注解或者直接在配置文件中定义，将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。spring的AOP恰好可以完成此功能：事务管理代码的固定模式作为一种横切关注点，通过AOP方法模块化，进而实现声明式事务。

## 用法

### 配置元数据

- **引入pom依赖**

```xml
<!--spring 声明式事务依赖-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
<!--配置druid数据源依赖-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.23</version>
</dependency>
```

#### 基于xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <context:component-scan base-package="com.transaction"></context:component-scan>

     <context:property-placeholder location="classpath:database.properties"></context:property-placeholder>
     <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
       <property name="username" value="${jdbc.username}"></property>
       <property name="password" value="${jdbc.password}"></property>
       <property name="url" value="${jdbc.url}"></property>
       <property name="driverClassName" value="${jdbc.driverClassName}"></property>
     </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <constructor-arg name="dataSource" ref="dataSource"></constructor-arg>
    </bean>

    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--开启基于注解的事务控制模式，依赖tx名称空间-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
</beans>
```

#### 基于javaconfig配置

```java
package com.transaction;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.*;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@ComponentScan("com.transaction")
@PropertySource("classpath:database.properties")
@EnableAspectJAutoProxy
@EnableTransactionManagement  //开启事务管理
public class TransactionAppConfig {
    @Value("${jdbc.username}")
    private String name;
    @Value("${jdbc.password}")
    private String password;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.driverClassName}")
    private String driverName;
    @Bean
    public DruidDataSource dataSource(){
        DruidDataSource dataSource=new DruidDataSource();
        dataSource.setUsername(name);
        dataSource.setPassword(password);
        dataSource.setUrl(url);
        dataSource.setDriverClassName(driverName);
        return dataSource;
    }

    @Bean
    public DataSourceTransactionManager transactionManager(DruidDataSource dataSource){
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DruidDataSource dataSource){
        return new JdbcTemplate(dataSource);
    }

}

```



### **Transactional注解**

@Transactional标记在类上面则当前类所有的方法都运用上了事务，标记在方法则只是当前方法运用事务，如果类和方法都存在@Transactional会以方法的为准。如果方法上面没有@Transactional会以类上面的为准。

建议：@Transactional写在方法上面，控制粒度更细，   建议@Transactional写在业务逻辑层上，因为只有业务逻辑层才会有嵌套调用的情况。

| 属性                   | 说明                                          |
| ---------------------- | --------------------------------------------- |
| isolation              | 设置事务的隔离级别                            |
| propagation            | 事务的传播行为                                |
| noRollbackFor          | 那些异常事务可以不回滚                        |
| noRollbackForClassName | 填写的参数是全类名                            |
| rollbackFor            | 哪些异常事务需要回滚                          |
| rollbackForClassName   | 填写的参数是全类名                            |
| readOnly               | 设置事务是否为只读事务                        |
| timeout                | 事务超出指定执行时长后自动终止并回滚,单位是秒 |

#### isolation

隔离级别（isolation）用来解决并发事务所产生一些问题如：

- 脏读：读取到其他事务未提交的数据。

- 不可重复度：同一事务内,第一次查询的结果行在第二次查询时出现改变 。可通过行锁解决

- 幻读: 读取到其他事务插入的新数据。只能通过表锁解决

![https://note.youdao.com/yws/public/resource/9ea06d3b1e9532e871342b8b7e7d9b0c/xmlnote/13862778AEC34E07A30008F19FBF8094/53AD5A6656A5480AA8B2C2C66A3F054C/1916](https://note.youdao.com/yws/public/resource/9ea06d3b1e9532e871342b8b7e7d9b0c/xmlnote/13862778AEC34E07A30008F19FBF8094/53AD5A6656A5480AA8B2C2C66A3F054C/1916)

对应spring有5中隔离级别：

| 隔离级别                   | 说明                                    |
| -------------------------- | --------------------------------------- |
| ISOLATION_DEFAULT          | 使用数据库默认的事务隔离级别            |
| ISOLATION_READ_UNCOMMITTED | 都未提交,会出现脏读、不可重复读、幻读   |
| ISOLATION_READ_COMMITTED   | 读已提交,会出现不可重复读、幻读问题     |
| ISOLATION_REPEATABLE_READ  | 可重复读,会出幻读（锁定所读取的所有行） |
| ISOLATION_SERIALIZABLE     | 串行化(表锁)                            |

#### propagation

当我们使用@Transaction注解声明了一个事务，若被声明事务内部还调用了另外的声明事务，则可以通过指定事务的传播行为,来控制事务如何处理。

| **事务传播行为类型** | **外部不存在事务** | 外部存在事务               | 作用                                                         |
| -------------------- | ------------------ | -------------------------- | ------------------------------------------------------------ |
| REQUIRED（默认）     | 开启新的事务       | 融合到外部事务中           | 保持只有一个事务,适用增删改查                                |
| SUPPORTS             | 无事务方式运行     | 融合到外部事务中           | 等于不使用@Transaction，一般用于查询                         |
| REQUIRES_NEW         | 开启新的事务       | 挂起外部事务，创建新的事务 | 外层异常不影响内层,内层异常抛出在外层事务未捕获会影响外层    |
| NOT_SUPPORTED        | 无事务方式运行     | 挂起外部事务               | 被挂起的事务会阻塞,内部以无事务方式运行                      |
| NEVER                | 无事务方式运行     | 抛出异常                   | 保证此方法以无事务方式执行                                   |
| MANDATORY            | 抛出异常           | 融合到外部事务中           | 保证此方法与外部事务处于同一个事务                           |
| NESTED               | 开启新的事务       | 融合到外部事务中           | savapoint自治事务,外层异常影响内层,内层异常抛出在外层事务未捕获会影响外层 |

注意:<span style="color:#ff0000">外部事务被挂起，若内部抛出异常,外层未捕获,则外部事务会因为异常而执行异常策略</span>

#### timeout

指定事务等待的最长时间（秒）当前事务访问数据时，有可能访问的数据被别的数据进行加锁的处理，那么此时事务就必须等待，如果等待时间过长给用户造成的体验感差。

#### **readOnly**

readonly:只会设置在查询的业务方法中,数据库就会对当前只读做相应优化

#### noRollbackFor&rollbackFor

异常处理策略,当出现指定异常时是否回滚。