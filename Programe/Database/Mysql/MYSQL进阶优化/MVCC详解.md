MVCC(**Multi-Version Concurrency Control**)机制保证了mysql可重复读和读已提交这两种事物隔离级别，

- **undo日志版本链**

  一行数据被多个事务修改，会通过trx_id 和 roll_pointer 形成一条数据的日志版本链。其中trx_id:只有在update时才会生成。

- **read-view**

​		可重复读级别下，当事务开启，执行了第一个查询语句时，会生成当前事务的一个read-view视图，记录了执行查询时所有未提交的最小事务ID和已创建的最大事务ID。在此事务中执行的任何sql查询结果，都会比对read-view

若事务ID小于read-view最小事务ID,则说明事务已提交数据可见

若事务ID大于read-view最大ID，则说明事务未开始数据不可见

若事务ID在read-view视图中，说明事务未提交，数据不可见。

