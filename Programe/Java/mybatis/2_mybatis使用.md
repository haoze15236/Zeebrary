在前面搭建好mybatis之后，可以看到实际开发过程中还是需要我们去写要执行的sql，正常开发也就是curd。我们先看insert,update,delete

# [主键自增](https://mybatis.org/mybatis-3/sqlmap-xml.html#insert_update_and_delete)

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

我们再来看看select中一些需要注意的部分:

# [参数占位符](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#Parameters)

- #{} ：使用这种参数占位符时，mybatis会预编译sql，通过？来占位，替换参数，可以预防sql注入的问题。
- ${} :   使用这种参数占位符时，mybatis会把参数直接替换sql中对应的占位，可以更加动态的生成sql，也就造成了sql注入的风险。
- `#{age,javaType=int,jdbcType=NUMERIC,typeHandler=MyTypeHandler}` 类似这种针对参数设置类型处理器的，可参考 [类型处理器](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers)

# parameterType

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

# resultType

在XML文件中通过resultType定义时

## 返回list 

返回值的类型依然写的是集合中具体的类型，此时返回的数据列名会和javabean的属性名自动映射。

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

## 返回map 

- 返回一条Map数据：当mybatis查询完成之后会把列的名称作为key列的值作为value，转换到map中。

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

- 返回Map若返回多条数据：在mapper接口上必须使用@MapKey来设置map的key为结果集中的那个字段，value必须与查询sql的字段列名相等

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

# [resultMap](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html#Result_Maps)

resultType 和 resultMap 都是用来声明返回结果集的，像resultType返回list使用javabean时，就是通过创建一个reesultMap,来根据属性名映射列名到javaBean上。而通过显示定义resultMap，然后通过`extends` 继承来复用，或者通过`resultMap`关联来复用

## 多对一

而当我们需要关联查询时，比如多行一头，可以通过resultMap手动映射关联查询的返回值。

### 手动映射

```xml
<resultMap id="BaseResultMap" type="mybatis.dto.ExpReportLine">
    <id property="billLineId" column="EXP_REPORT_LINE_ID"/>
    <result property="billHeaderId" column="EXP_REPORT_HEADER_ID"/>
    <result property="description" column="DESCRIPTION"/>
    <result property="payRatio" column="PAT_RATIO"/>
</resultMap>
<resultMap id="expReportHeaderLineMap" extends="BaseResultMap" type="mybatis.dto.ExpReportHeaderLine">
    <result property="expReportHeader.billHeaderId" column="EXP_REPORT_HEADER_ID"/>
    <result property="expReportHeader.billNumber" column="EXP_REPORT_NUMBER"/>
    <result property="expReportHeader.bankInfo" column="BANK_INFO"/>
</resultMap>

    <select id="selectHeaderLine" parameterType="int" resultMap="expReportHeaderLineMap">
        SELECT
            l.*,
            h.*
        FROM
            exp_report_line l
        left join exp_report_header H on l.EXP_REPORT_HEADER_ID = H.EXP_REPORT_HEADER_ID
        where H.EXP_REPORT_HEADER_ID = #{id,javaType=int,jdbcType=NUMERIC}
    </select>
```

定义一个查询返回的javabean，包含头javaBean,resultMap映射中如上使用javabean.属性来映射

```java
package mybatis.dto;
public class ExpReportHeaderLine extends ExpReportLine{
    
	private ExpReportHeader expReportHeader;

	@Override
	public String toString() {
		return super.toString()+"ExpReportHeaderLine{" +
				"expReportHeader=" + expReportHeader.toString() +
				'}';
	}
}
```

### association

是mybatis封装的专门用于多对一的组件,根据resultMap中的`id`标签来确定唯一性,若id标签对应的数据重复，则会舍弃重复数据，保证多对一,若resultMap中没有ID字段，则无法确定唯一性，那么就会显示所有数据。

```xml
<resultMap id="BaseResultMap" type="mybatis.dto.ExpReportLine">
    <!--确定数据唯一性,id字段值一致,association关联数据如果不一致,会舍弃查询出来的数据-->
    <id property="billLineId" column="EXP_REPORT_LINE_ID"/>
    <result property="billHeaderId" column="EXP_REPORT_HEADER_ID"/>
    <result property="description" column="DESCRIPTION"/>
    <result property="payRatio" column="PAT_RATIO"/>
</resultMap>
<resultMap id="expReportHeaderLineMap" extends="BaseResultMap" type="mybatis.dto.ExpReportHeaderLine">
    <association property="expReportHeader"resultMap="mybatis.mapper.ExpReportHeaderMapper.BaseResultMap" >
    </association>
</resultMap>

<select id="selectHeaderLine" parameterType="int" resultMap="expReportHeaderLineMap">
    SELECT
    l.*,
    h.*
    FROM
    exp_report_line l
    left join exp_report_header H on l.EXP_REPORT_HEADER_ID = H.EXP_REPORT_HEADER_ID
    where H.EXP_REPORT_HEADER_ID = #{id,javaType=int,jdbcType=NUMERIC}
</select>
```

可以看到，使用了association可以宠用resultMap,非常方便。但是有时候可能存在字段重名的情况,此时可以使用associationD的`columnPrefix`属性来定义关联的 **一** 的字段名前缀来区分。

## 一对多

### collection

使用resultMap的collection子节点来定义一对多，比如一个单据头，有多个单据行的情况。用法基本与association相同，同样通过ID来确定唯一性,若没有ID，则会查出多行

```xml
<resultMap id="BaseResultMap" type="mybatis.dto.ExpReportHeader">
    <id column="EXP_REPORT_HEADER_ID" property="billHeaderId"/>
    <result column="EXP_REPORT_NUMBER" property="billNumber"/>
    <result column="BANK_INFO" property="bankInfo"/>
</resultMap>

<resultMap id="expReportHeaderLine" extends="BaseResultMap" type="mybatis.dto.ExpReportHeader">
    <collection property="expReportLineList" ofType="mybatis.dto.ExpReportLine" resultMap="mybatis.mapper.ExpReportLineMapper.BaseResultMap"/>
</resultMap>

<select id="selectOne" resultMap="expReportHeaderLine">
    SELECT
    l.*,
    h.*
    FROM
    exp_report_header H
    left join exp_report_line l on l.EXP_REPORT_HEADER_ID = H.EXP_REPORT_HEADER_ID
    where H.EXP_REPORT_HEADER_ID = #{id,javaType=int,jdbcType=NUMERIC}
</select>
```

- **ofType** 指定关联集合的javabean。

# sql

在编写sql的时候，可能有很多都是重复的，通过这个元素可以用来定义可重用的 SQL 代码片段，以便在其它语句中使用。 参数可以静态地（在加载的时候）确定下来，并且可以在不同的 include 元素中定义不同的参数值。比如：

```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
```

这个 SQL 片段可以在其它语句中使用，例如：

```xml
<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```

也可以在 include 元素的 refid 属性或内部语句中使用属性值，例如：

```xml
<sql id="sometable">
  ${prefix}Table
</sql>

<sql id="someinclude">
  from
    <include refid="${include_target}"/>
</sql>

<select id="select" resultType="map">
  select
    field1, field2, field3
  <include refid="someinclude">
    <property name="prefix" value="Some"/>
    <property name="include_target" value="sometable"/>
  </include>
</select>
```

注意：<span style="color:red">sql中获取参数要使用${}</span>,如果使用#{}会被拼上引号，因为是我们自己定义的参数，所以不存在sql注入风险。

# [动态SQL](https://mybatis.org/mybatis-3/zh/dynamic-sql.html)

## 常用OGNL表达式

```txt
e1 or e2
e1 and e2
e1 == e2 或者可以使用e1 eq e2
e1 != e2 或者可以使用e1 neq e2
e1 lt e2 ：小于
e1 lte e2 ：小于等于，其他gt（大于）,gte（大于等于）
e1 in e2
e1 not in e2
e1 + e2,e1 * e2,e1/e2,e1 ‐ e2,e1%e2 :支持运算
!e,not e：非，求反
e.method(args)调用对象方法
e.property对象属性值
e1[e2]按索引取值，List,数组和Map
@class@method(args)调用类的静态方法
@class@field调用类的静态字段值
```

## Mybatis批量操作

### 动态sql批量操作

- **批量删除**

```XML
<delete id="deleteBatch">
    <if test="list != null and list.size>0">
        delete from puma_employee_basic where employee_no in
        <foreach collection="list" item="item" open="(" separator="," close=")">
            #{item.employeeNo}
        </foreach>
    </if>
</delete>
```

- **批量插入**

```xml
<insert id="insertBatch" useGeneratedKeys="true" keyProperty="unitId">
        <if test="list != null and list.size > 0">
            INSERT INTO csh_repayment_register_head_itf(UNIT_CODE
          ,CREATED_BY,CREATION_DATE,LAST_UPDATED_BY,LAST_UPDATE_DATE,OBJECT_VERSION_NUMBER,REQUEST_ID,PROGRAM_ID,LAST_UPDATE_LOGIN) VALUES
            <foreach collection="list" item="item" index="index" separator=",">
                (#{item.unitCode},#{request.userId} ,now() ,#{request.userId},now() ,1 ,null ,null ,#{request.userId})
            </foreach>
        </if>
    </insert>
```

- **批量更新**

<span style="color:red">由于数据库默认一次只能执行一个语句，所以想支持通过`；`分割语句批量执行，需要设置`allowMultiQueries` = ture;</span>

```properties
db.url=jdbc:mysql://10.75.1.61:3306/hap_dev?allowMultiQueries=true
```

```xml
<update id="updateBatch">
        <if test="list != null and list.size > 0">
            <foreach collection="list" item="item" index="index" separator=";">
                update pm_apr_act_obj_itm_ele_tmp
                set TMP_ID = IFNULL(#{item.tmpId},TMP_ID),
                ACTIVITY_ID = IFNULL(#{item.activityId},ACTIVITY_ID),
                OBJECT_TYPE = IFNULL(#{item.objectType},OBJECT_TYPE),
                OBJECT_ID = IFNULL(#{item.objectId},OBJECT_ID),
                VALUE = IFNULL(#{item.value},VALUE),
                SOURCE_VALUE = IFNULL(#{item.sourceValue},SOURCE_VALUE),
                ITEM_ELEMENT_DESCRIPTION = IFNULL(#{item.itemElementDescription},ITEM_ELEMENT_DESCRIPTION),
                ITEM_ID = IFNULL(#{item.itemId},ITEM_ID),
                ELEMENT_ID = IFNULL(#{item.elementId},ELEMENT_ID),
                LAST_UPDATED_BY = IFNULL(#{item.lastUpdatedBy},LAST_UPDATED_BY),
                LAST_UPDATE_DATE = IFNULL(#{item.lastUpdateDate},LAST_UPDATE_DATE),
                OBJECT_VERSION_NUMBER = IFNULL(#{item.objectVersionNumber},OBJECT_VERSION_NUMBER) +1,
                WHERE TMP_ID = #{item.tmpId}
                AND OBJECT_VERSION_NUMBER = #{item.objectVersionNumber}
            </foreach>
        </if>
    </update>
```

- **批量查询**

```xml
 <select id="selectBatch" resultType="com.hand.hssp.fnd.dto.FndDimensionValue">
        SELECT
    DIMENSION_VALUE_ID,DIMENSION_ID,DIMENSION_VALUE_CODE,DESCRIPTION,SUMMARY_FLAG,ENABLED_FLAG,
        CREATED_BY,CREATION_DATE,LAST_UPDATED_BY,LAST_UPDATE_DATE,OBJECT_VERSION_NUMBER,REQUEST_ID,PROGRAM_ID,
        LAST_UPDATE_LOGIN
        FROM
        fnd_dimension_value f
        <where>
            1=1
            <if test="list != null and list.size()>0">
                AND
                <foreach collection="list" item="item" index="index" open="(" close=")"
                         separator="or">
                    (f.DIMENSION_VALUE_CODE=#{item.dimensionValueCode}
                    and f.DIMENSION_ID = #{item.dimensionId})
                </foreach>
            </if>
        </where>
    </select>
```

### 设置sqlSession的Executor

在通过sqlSessionFactory.openSession()获取到的sqlsession中，mybatis默认采用的Executor是SIMPLE，底层是通过把sql语句包装在PrepareStatement中执行,这也就导致了每次执行xml中的DML语句，都需要与数据库交互一次，对于经常批量插入，更新的系统，可以通过设置使用Executor为BATCH，来使用JDBC底层的BatchStatement，提高性能。

- [全局设置](https://mybatis.org/mybatis-3/zh/configuration.html#settings) ：设置setting属性`defaultExecutorType` 为BATCH；

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="mybatis.properties"/>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="defaultExecutorType" value="BATCH"/>
    </settings>
</configuration>
```

- 单个sqlSession设置：通过sqlSessionFactory.openSession()方法传入执行器

```java
@Test
public void testExpReportHeaderLine(){
    try (SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH,true)) {
        ExpReportLineMapper mapper = session.getMapper(ExpReportLineMapper.class);
        System.out.println(mapper.selectHeaderLine(1));
    }
}
```

