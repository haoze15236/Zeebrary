# Mysql慢查询日志

**概念：** 记录所有执行时间超过设定的阈值查询sql的日志。

**应用场景：** 统计影响mysql数据库性能的查询sql，有针对性的进行sql优化或者重构。

## 相关系统参数说明

- **slow_query_log**  

  是否开启慢查询日志，1表示开启，0表示关闭。

  ```shell
  #查看数据库当前配置
  show variables  like '%slow_query_log%';
  #临时开启慢查询日志
  set global slow_query_log=1;
  #永久开启需要修改my.cnf配置文件
  ```

- **slow_query_log_file**

  （5.6及以上版本）MySQL数据库慢查询日志存储路径。可以不设置该参数，系统则会默认给一个缺省的文件host_name-slow.log，

  （5.6以下版本）使用的是`log-slow-queries`

- **long_query_time** 

  慢查询阈值，当查询时间多于设定的阈值时，记录日志，单位是秒。

  ```shell
  #查看数据库当前配置
  show variables  like '%long_query_time%';
  #临时开启慢查询日志,需重新建立连接才能看到生效
  set global long_query_time=2;
  #永久开启需要修改my.cnf配置文件
  ```

- **log_queries_not_using_indexes**

  未使用索引的查询也被记录到慢查询日志中。1表示开启，0表示关闭。

- **log_output**

  日志存储方式。log_output='FILE'表示将日志存入文件，默认值是'FILE'。log_output='TABLE'表示将日志存入数据库，这样日志信息就会被写入到mysql.slow_log表中。MySQL数据库支持同时两种日志存储方式，配置的时候以逗号隔开即可，如：log_output='FILE,TABLE'。日志记录到系统的专用日志表中，要比记录到文件耗费更多的系统资源，因此对于需要启用慢查询日志，又需要能够获得更高的系统性能，那么建议优先记录到文件。

- **Slow_queries**

  查看目前系统的慢查询数量。

## 慢sql日志说明

日志文件中基本都是由以下5个部分组成

```shell
# Time: 2021-04-26T11:02:21.155988+08:00 
# User@Host: hap_prod[hap_prod] @  [10.75.1.192]  Id: 11172		
# Query_time: 49.056324  Lock_time: 0.003756 Rows_sent: 1  Rows_examined: 4605615
SET timestamp=1619406141;
SELECT count(0) FROM (table);
```

- **Time** ：sql执行的日期
- **User@Host** :请求的用户IP
- **Query_Time** : sql执行的时间
- **Lock_time** : sql执行获取锁的时间
- **Rows_sent** : 返回给客户端的数据量
- **Rows_examined** : 服务层扫描的数据量

慢日志如果数据比较多的话,并不能很直观的找到我们想要优化的最垃圾的sql,因此我们使用一个第三方日志分析工具

# 日志分析工具-mysqlsla

## 安装

1. 下载zip包:[*https://github.com/daniel-nichter/hackmysql.com*](https://github.com/daniel-nichter/hackmysql.com)

2. 安装并编译:

   ```shell
   #下载mysqlsla
   [root@server-10 ~]# yum install  perl  perl-DBI  perl-DBD-MySQL perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker
   #解压下载的zip,并进入解压文件夹的/mysqlsla目录下,开始编译
   [root@server-10 mysqlsla]# perl Makefile.PL 
   Checking if your kit is complete...
   Looks good
   Writing Makefile for mysqlsla
   [root@server-10 mysqlsla]# make
   cp lib/mysqlsla.pm blib/lib/mysqlsla.pm
   cp bin/mysqlsla blib/script/mysqlsla
   /usr/bin/perl -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/mysqlsla
   Manifying blib/man3/mysqlsla.3pm
   [root@server-10 mysqlsla]# make install
   Installing /usr/local/share/perl5/mysqlsla.pm
   Installing /usr/local/share/man/man3/mysqlsla.3pm
   Installing /usr/local/bin/mysqlsla 　　　　　　　　　　　　//生成了mysqlsla命令
   Appending installation info to /usr/lib64/perl5/perllocal.pod
   ```

## 使用

### 参数说明

- -lt

  **log­-type **:用于指定解析的是什么类型的日志，不过放心，即便你不指定，它默认也会自动去判断基本的日志，slow，general，binary，msl or udl。

- -sf 

  **statement-­filter** :语句类型过滤,可选的值就是 :SELECT,CREATE,DROP,UPDATE,INSERT等等,多个使用,分开.如 -sf "+select,update"表示查看select和update语句.

- --top N

  显示默认降序的top N， default 为10.

- -mf 

  **meta-property-filter** : 元数据属性过滤,使用格式:`-mf [meta][op][value]`

  如:	`-mf 't_sum>100,t_sum<200' ` 表示查SQL执行时间总和大于100秒,小于200秒.

  `[op]` :只有三个可选项:`>`,`<`,`=`

  `[meta]`:可选参数及解释如下.

  | log类型 | meta     | 解释                  | 限制                              |
  | :------ | :------- | :-------------------- | :-------------------------------- |
  | all     | c_sum    | SQL次数总和           | 无                                |
  | all     | db       | database名称          | 只能用作meta-filter，不能用作sort |
  | all     | exec     | 真实执行时间          | 只能用做sort，不能filter          |
  | all     | exec_sum | c_sum*exec            | 只能作用sort，不能filter          |
  | slow    | host     | 主机名                | 只能用作meta-filter，不能用作sort |
  | slow    | ip       | ip地址                | 只能用作meta-filter，不能用作sort |
  | slow    | l_avg    | 锁的平均等待时间      | 无                                |
  | slow    | re_sum   | rows_examined的总和   | 无                                |
  | slow    | re_avg   | rows_examined的平均值 | 无                                |
  | slow    | rs_sum   | rows_sent的总和       | 无                                |
  | slow    | rs_avg   | rows_sent的平均值     | 无                                |
  | slow    | t_sum    | SQL执行时间的总和     | 无                                |
  | slow    | t_avg    | SQL执行时间的平均值   | 无                                |
  | slow    | user     | 用户名                | 无                                |
  | general | cid      | 连接id                | 无                                |
  | general | host     | 主机名                | 无                                |
  | general | user     | 用户名                | 无                                |
  | binary  | ext      | 执行时间              | 无                                |
  | udl     | 无       | 无                    | 无                                |

- --sort 

  **sort** : 根据Meta的值，进行排序。默认slow log的meta 为t_sum(SQL执行时间的总和)

### 案例

```shell
#统计慢查询文件为slow.log中的所有select的慢查询sql，并显示执行时间最长的100条sql，并写到sql_select.log中去
mysqlsla -lt slow  -sf "+select" -top 100  slow.log >/soft/mysqlsla/sql_select.log

#查询记录最多的20个sql语句，并写到select.log中去
mysqlsla -lt slow --sort t_sum --top 20 slow.log > /tmp/select.log

#统计慢查询文件为slow.log的数据库为mydata的所有select和update的慢查询sql，并查询次数最多的100条sql，并写到sql_num.sql中去
mysqlsla -lt slow  -sf "+select,update" -top 100 -sort c_sum  -db mydata slow.log >/tmp/sql_num.log

#统计慢查询文件为slow.log的数据库为 hap_prod 的所有select和update的慢查询sql，并查询总时长最多的100条sql，并写到sql_num.sql中去
mysqlsla -lt slow  -sf "+update,insert" -top 100 -sort t_sum  -db hap_prod slow.log >/tmp/sql_num.log
```

# 参考资料

mysqlsla资料:[https://developer.aliyun.com/article/59260](https://developer.aliyun.com/article/59260)