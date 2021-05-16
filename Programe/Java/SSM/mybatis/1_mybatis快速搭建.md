官方文档:[https://mybatis.org/mybatis-3/zh/index.html](https://mybatis.org/mybatis-3/zh/index.html)

# 快速搭建

1. 引入依赖，idea记得put into WEB-INF/lib

```xml
<!--mybatis-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.6</version>
</dependency>
```

2. 在项目`resource`目录下定义mybatis配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--定义属性变量-->
    <properties resource="mybatis.properties"/>
    <settings>
        <!--开启驼峰命名自动映射,数据库字段名A_COLUMN,dto属性为aColumn时,未开启此参数会无法获取pojo-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

- [properties](https://mybatis.org/mybatis-3/zh/configuration.html#properties) : 定义变量属性，在其他地方通过`${xxxx}`的格式来引用。
- [settings](https://mybatis.org/mybatis-3/zh/configuration.html#settings) ：定义一些mybatis的全局属性,

- [environments](https://mybatis.org/mybatis-3/zh/configuration.html#environments) ：定义环境信息
- [mappers](https://mybatis.org/mybatis-3/zh/configuration.html#mappers) :定义mapper映射器

3. 手动创建`sqlSessionFactory`

```java
package mybatis;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;

public class testXmlConfig {
	SqlSessionFactory sqlSessionFactory;

	@Before
	public void sqlSessionFactory(){
		String resource = "mybatis-config.xml";
		InputStream inputStream = null;
		try {
			inputStream = Resources.getResourceAsStream(resource);
		} catch (IOException e) {
			e.printStackTrace();
		}
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
	}

	@Test
	public void testConnection(){
		try (SqlSession session = sqlSessionFactory.openSession()) {
			session.commit();
		}
	}
}

```

4. 前面获取到了sqlSession,就可以通过mapper映射器来执行具体的sql了,日常开发中，一般使用接口绑定的方式。

# 执行sql

## statement id

- mybatis全局配置文件添加mapper映射器:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="mybatis.properties"/>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--定义mapper的相对路径-->
        <mapper resource="mybatis/mapper/ExpReportHeaderMapper.xml"/>
    </mappers>
</configuration>
```

- mapper映射器定义,其中namespace为mapper的全局ID

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="mybatis.dto.ExpReportHeaderMapper">

    <select id="selectExpReportHeader" resultType="mybatis.dto.ExpReportHeader">
        select * from exp_report_header erh where erh.EXP_REPORT_HEADER_ID = #{id}
    </select>

</mapper>
```

- sqlSession通过mapper映射器namespace及具体的dml语句ID，来执行其中的sql。

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
			ExpReportHeader expReportHeader = (ExpReportHeader)session.selectOne("mybatis.dto.ExpReportHeaderMapper.selectExpReportHeader", 1);
			System.out.println(expReportHeader.getExpReportNumber());
		}
```

## 接口绑定方式

- mybatis全局配置文件添加mapper映射器:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="mybatis.properties"/>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <environments default="development">
        <environment id="development">
            <!--使用jdbc做事务管理 若值为MANAGED则不运用事务-->
            <transactionManager type="JDBC"/>
            <!--POOLED ：mybatis默认的数据源连接池
				UNPOOLED:不使用连接池
				JNDI:JNDI连接池，可在tomcat中使用
				-->
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!--此报下所有mapper接口都会被注册-->
        <package name="mybatis.mapper"/>
    </mappers>
</configuration>
```

- 在mapper扫描的包下新建mapper接口

```java
package mybatis.mapper;

import mybatis.dto.ExpReportHeader;
import org.apache.ibatis.annotations.Param;

public interface ExpReportHeaderMapper {
	ExpReportHeader selectOne(@Param("id") int expReportHeaderId);
}

```

- 定义mapper.xml文件,其中namespace必须指定为mapper接口的完全限定名

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="mybatis.mapper.ExpReportHeaderMapper">

    <select id="selectOne" resultType="mybatis.dto.ExpReportHeader">
        select * from exp_report_header erh where erh.EXP_REPORT_HEADER_ID = #{id}
    </select>

</mapper>
```

- sqlSession获取mapper接口，执行方法获取sql执行结果

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
			ExpReportHeaderMapper mapper = session.getMapper(ExpReportHeaderMapper.class);
			ExpReportHeader expReportHeader = mapper.selectOne(1);
			System.out.println(expReportHeader);
		}
```

跟踪源码其底层是通过jdk动态代理来创建具体实现类,在`org.apache.ibatis.binding.MapperProxyFactory`中

```java
Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
```

## 注解方式

在接口绑定方式中，可以抛弃xml配置，在mapper接口中通过注解直接写sql语句,不过这种方式对于一些复杂的sql并不推荐使用

```java
package mybatis.mapper;

import mybatis.dto.ExpReportHeader;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;

public interface ExpReportHeaderMapper {
	@Select("select * from exp_report_header erh where erh.EXP_REPORT_HEADER_ID = #{id}")
	ExpReportHeader selectOne(@Param("id") int expReportHeaderId);
}
```

