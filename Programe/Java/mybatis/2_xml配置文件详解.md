在前面搭建好mybatis之后，可以看到实际开发过程中还是需要我们去写要执行的sql，正常开发也就是curd。我们先看insert,update,delete

# [主键自增](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html)

- 我们主要关注insert中对于表主键的处理，对于MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段，我们可以通过`useGeneratedKeys`和`keyProperty`来生成主键ID，返回到pojo中：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="mybatis.mapper.ExpReportHeaderMapper">
     <insert id="insertOne" useGeneratedKeys="true" keyProperty="expReportHeaderId" >
        insert into exp_report_header(EXP_REPORT_NUMBER) value (#{expReportHeader.expReportNumber})
    </insert>
</mapper>
```

对应的mapper接口：

```java
package mybatis.mapper;

import mybatis.dto.ExpReportHeader;
import org.apache.ibatis.annotations.Param;

public interface ExpReportHeaderMapper {
	int insertOne(@Param("expReportHeader")ExpReportHeader expReportHeader);
}
```

在应用层代码中可以获取到对应的主键ID

```java
try (SqlSession session = sqlSessionFactory.openSession(true)) {
			ExpReportHeaderMapper mapper = session.getMapper(ExpReportHeaderMapper.class);
			ExpReportHeader expReportHeader = new ExpReportHeader();
			expReportHeader.setExpReportNumber("haoze");
			mapper.insertOne(expReportHeader);
			System.out.println(expReportHeader);
		}
```

此时在java中返回的pojo类中就可以获取到生成的主键ID

- 当对于oracle这种不支持主键自增的数据库时，可以通过`selectKey`来操作:

```xml
<insert id="insertOne" >
    <selectKey keyProperty="expReportHeaderId" resultType="int" order="BEFORE">
        select max(erh.EXP_REPORT_HEADER_ID)+1 from exp_report_header erh
    </selectKey>
    insert into exp_report_header(EXP_REPORT_HEADER_ID,EXP_REPORT_NUMBER) value (#{expReportHeaderId},#{expReportHeader.expReportNumber})
</insert>
```

# select

## 参数占位符

- #{} ：使用这种参数占位符时，mybatis会预编译sql，通过？来占位，替换参数，可以预防sql注入的问题。
- ${} :   使用这种参数占位符时，mybatis会把参数直接替换sql中对应的占位，可以更加动态的生成sql，也就造成了sql注入的风险。

## parameterType

在XML文件中通过parameterType定义，mybatis内置别名可参考[mybatis类型别名（typeAliases）](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases)

- **单个参数**：`SelectEmp(Integer id);` ,使用#{输入任何字符获取参数}

- **多个参数**：`SelectEmp(Integer id,String username);`

  mybatis 会将传进来的参数封装成map,1个值就会对应2个map项,比如

  ```properties
  id： {key:arg0 ,value:id的值},{key:param1 ,value:id的值}
  username：{key:arg1 ,value:id的值},{key:param2 ,value:id的值}
  ```

  因此获取的时候可以通过`#{arg0}`,`#{param1}`来获取到id，`#{arg1}`,`#{param2}`来获取到username，或者使用<span style="color:red">`@Param`</span>注解来修饰参数，如`SelectEmp(@Param("id") Integer id,@Param("username") String username);`则可以通过#{id}直接获取到id

- **单个javaBean的参数** ：`Emp SelectEmp(Emp emp);` 可以直接使用属性名
- **多个javaBean参数** ：`SelectEmp(@Param('emp')Emp emp,Unit unit);` 建议使用<span style="color:red">`@Param`</span>标注之后，通过#{emp.id}来获取ID

- **集合或者数组参数**：`SelectEmp(List<String> usernames)`

  如果是list,MyBatis会自动封装为map:{key:"list":value:usernames}，因此可以使用#{list[0]}

  如果是数组,MyBatis会自动封装为map:{key:"array":value:usernames},可以使用#{array[0]}

  若使用<span style="color:red">`@Param`</span>标注之后`SelectEmp(@Param('usernames')List<String> usernames)`，可以使用#{usernames[0]}

- **map参数**：同javaBean参数

## resultType

在XML文件中通过resultType定义时

- 返回list ：返回值的类型依然写的是集合中具体的类型

```xml
  <select id="selectList" resultType="mybatis.dto.ExpReportLine">
        SELECT
            *
        FROM
            exp_report_line l
    </select>
```

```java
package mybatis.mapper;

import mybatis.dto.ExpReportLine;

import java.util.List;

public interface ExpReportLineMapper {
	List<ExpReportLine> selectList();
}
```

- 返回map ：当mybatis查询完成之后会把列的名称作为key列的值作为value，转换到map中。只用于返回一条数据的情况

```java
public interface ExpReportLineMapper {
	Map selectMap();
}
```

```java
<select id="selectMap" resultType="map">
    SELECT
    *
    FROM
    exp_report_line l
</select>
```

- 返回map若返回多条数据：在mapper接口上必须使用@MapKey来设置map的key为结果集中的那个字段

```java
public interface ExpReportLineMapper {
	@MapKey("EXP_REPORT_LINE_ID")
	Map<Long, ExpReportLine> selectMap();
}
```

```xml
    <select id="selectMap" resultType="map">
        SELECT
            *
        FROM
            exp_report_line l
    </select>
```

## resultMap

resultType 和 resultMap 都是用来声明返回结果集的，因此二者只能选其一。