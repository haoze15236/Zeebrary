**错误描述:**

```
Mapper method 'com.hand.hssp.xcitf.zt.mapper.AcrInvoiceImportItfMapper.batchInsert' has an unsuppor
```

**错误原因:**

Mapper使用的insert语句,mapper.xml对应的interface的方法返回值类型不对。



**错误描述**

```
sevice.selectOptions 报错NPE
```

**错误原因**：

在DTO类上使用了@JoinCode关联查询的注解，对应的Code没有定义