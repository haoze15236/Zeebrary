[MYSQL timestamp NOT NULL插入NULL的报错问题](https://www.cnblogs.com/lonecloud/p/9732396.html)

1. 在开发两个数据库数据同步功能的时候，需要在本地搭建一个本地的数据库作为一个本地库，然后用于同步开发库中的数据。在插入的时候出现了一个问题。

**问题描述：**

  我们每张表中都会存在一个create_time 以及update_time两个字段。该两个字段的定义如下：

```sql
`create_date` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT ``'创建日期'``,
​```update_date` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP(0) COMMENT ``'更新时间'``,
```

  其中创建时间采用timestamp类型并且其默认值为CURRENT_TIMESTAMP。

  当我向数据库中插入一条数据的时候，create_time与Update_time设置为null的时候，开发库中会走默认值，但是本地库并不会，并且报错“create_time” cannot be null

**问题解决：**

查询google后发现

MySql系统变量explicit_defaults_for_timestamp： 该变量的作用为：

[![img](https://upload-images.jianshu.io/upload_images/2031765-8e917d07ed3c4813.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/613/format/webp)](https://upload-images.jianshu.io/upload_images/2031765-8e917d07ed3c4813.png?imageMogr2/auto-orient/strip|imageView2/2/w/613/format/webp)

查看了一下解释

[![img](https://upload-images.jianshu.io/upload_images/2031765-567b80cff7d5fe11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/838/format/webp)](https://upload-images.jianshu.io/upload_images/2031765-567b80cff7d5fe11.png?imageMogr2/auto-orient/strip|imageView2/2/w/838/format/webp)

大概是这么说的：

当你设置为false的时候：

- 未明确声明为NULL属性的TIMESTAMP列被分配为NOT NULL属性。 （其他数据类型的列，如果未显式声明为NOT NULL，则允许NULL值。）将此列设置为NULL将其设置为当前时间戳。
- 表中的第一个TIMESTAMP列（如果未声明为NULL属性或显式DEFAULT或ON UPDATE子句）将自动分配DEFAULT CURRENT_TIMESTAMP和ON UPDATE CURRENT_TIMESTAMP属性。
- 第一个之后的TIMESTAMP列（如果未声明为NULL属性或显式DEFAULT子句）将自动分配DEFAULT'0000-00-00 00:00:00'（“零”时间戳）。 对于不指定此列的显式值的插入行，该列将分配“0000-00-00 00:00:00”，并且不会发生警告。

当你设置为true的时候：

[![img](https://upload-images.jianshu.io/upload_images/2031765-7cd2a8bd7c64c405.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/829/format/webp)](https://upload-images.jianshu.io/upload_images/2031765-7cd2a8bd7c64c405.png?imageMogr2/auto-orient/strip|imageView2/2/w/829/format/webp)

- 未明确声明为NOT NULL的TIMESTAMP列允许NULL值。 将此列设置为NULL将其设置为NULL，而不是当前时间戳。
- 没有TIMESTAMP列自动分配DEFAULT CURRENT_TIMESTAMP或ON UPDATE CURRENT_TIMESTAMP属性。 必须明确指定这些属性。
- 声明为NOT NULL且没有显式DEFAULT子句的TIMESTAMP列被视为没有默认值。 对于不为此列指定显式值的插入行，结果取决于SQL模式。 如果启用了严格的SQL模式，则会发生错误。 如果未启用严格的SQL模式，则会为列分配隐式默认值“0000-00-00 00:00:00”，并发出警告。 这类似于MySQL如何处理其他时间类型，如DATETIME。

```shell
#查看数据库参数
SHOW VARIABLES LIKE '%timestamp%';
#修改数据库参数
set global  explicit_defaults_for_timestamp=off;
#查看数据库配置/etc/my.cnf
explicit_defaults_for_timestamp=true
```

