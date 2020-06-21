# Mybatis批量操作

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
<insert id="insertBatch" useGeneratedKeys="true" keyProperty="employeeId">
    <if test="list != null and list.size > 0">
        INSERT INTO exp_employee(EMPLOYEE_CODE) VALUES
        <foreach collection="list" item="expEmployee" index="index" separator=",">
            (#{expEmployee.employeeCode})
        </foreach>
    </if>
</insert>
```

- **批量更新**

```xml
<update id="updateBatch">
    <if test="list != null and list.size > 0">
        <foreach collection="list" item="item" index="index" separator=";">
            update sys_user
            set EMAIL = IFNULL(#{item.email},EMAIL)
            ,PHONE = IFNULL(#{item.phone},PHONE)
            ,STATUS = IFNULL(#{item.status},STATUS)
            ,LAST_UPDATED_BY =  #{request.userId}
            ,LAST_UPDATE_DATE = now()
            ,OBJECT_VERSION_NUMBER = OBJECT_VERSION_NUMBER + 1
            WHERE 1=1
            AND EMPLOYEE_ID = #{item.employeeId}
            AND OBJECT_VERSION_NUMBER = #{item.objectVersionNumber}
        </foreach>
    </if>
</update>
```

# 错误合集

- ```shell
  nested exception is org.apache.ibatis.binding.BindingException: Parameter '__frch_item_0' not found. Available parameters are [collection, list]
  ```

  - 查看parameterType的类型是不是Java.util.List类型
  - 看foreach的collection属性是不是list
  - 看foreach里取的属性值是否写错，大小写是否相同
  - 查看foreach里取的属性值实体对象中是否存在