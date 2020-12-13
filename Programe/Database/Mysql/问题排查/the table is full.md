# 记录The table is full问题排查

**问题描述：**正式环境线上突然出现the table is full的报错

**问题排查：**

```shell
# 查看系统磁盘空间
df -h
# 查看文件夹下所有内容占用大小
du -sh *
# a表示显示目录下所有的文件和文件夹（不含子目录），h表示以人类能看懂的方式，max-depth表示目录的深度
du -ah --max-depth=1
```

查看发现mysql 的binlog日志文件占用满了磁盘空间,检查binlog日志,原因是某个同步主数据job次数太频繁。

**解决办法:**删除binlog备份,释放磁盘空间。

排查原因发现是一个同步主数据的job跑的太频繁(1s一次),导致binlog日志暴涨。

**问题描述**:mysql主主同步失效

**问题排查：**

```mysql
# 查看主从同步状态
show slave status \G
```

```mysql
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.75.1.225
                  Master_User: repuser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000841
          Read_Master_Log_Pos: 19343
               Relay_Log_File: mysql-relay-bin.002370
                Relay_Log_Pos: 407
        Relay_Master_Log_File: mysql-bin.000840
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 1032
                   Last_Error: Could not execute Delete_rows event on table hap_prod.qrtz_simple_triggers; Can't find record in 'qrtz_simple_triggers', Error_code: 1032; handler error HA_ERR_KEY_NOT_FOUND; the event's master log mysql-bin.000840, end_log_pos 807
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 234
              Relay_Log_Space: 253632
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 1032
               Last_SQL_Error: Could not execute Delete_rows event on table hap_prod.qrtz_simple_triggers; Can't find record in 'qrtz_simple_triggers', Error_code: 1032; handler error HA_ERR_KEY_NOT_FOUND; the event's master log mysql-bin.000840, end_log_pos 807
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 3c3b3fcd-ed89-11ea-9923-00505695a2be
             Master_Info_File: /data/mysqldata/data/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State:
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp: 201123 10:34:17
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 3c3b3fcd-ed89-11ea-9923-00505695a2be:1-5263903
            Executed_Gtid_Set: 0ab842fc-ed93-11ea-ac5b-005056954519:1-69427,
3c3b3fcd-ed89-11ea-9923-00505695a2be:1-5263899
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```

**解决办法**

1. 跳过错误Event

```mysql
#暂停同步
stop slave;
#跳过event
set global sql_slave_skip_counter=1;
#启动同步
start slave;
```

2. 跳过所有1032错误

   更改my.cnf文件，在Replication settings下添加：

```mysql
#跳过所有错误码为1032的event
slave-skip-errors＝1032
```

并重启数据库，然后start salve。

注意：因为要重启数据库，不推荐，除非错误事件太多。

3. 还原被删除的数据

   根据错误提示信息，用mysqlbinlog找到该条数据event SQL并逆向手动执行。如delete 改成insert。

   本例中，此事件在主服务器Master binlog中的位置是 mysql-bin.000003, end_log_pos 440267874。

   1）利用mysqlbinlog工具找出440267874的事件

   /usr/local/mysql-5.6.30/bin/mysqlbinlog --base64-output=decode-rows -vv mysql-bin.000003 |grep -A 20 '440267874'

   或者/usr/local/mysql-5.6.30/bin/mysqlbinlog --base64-output=decode-rows -vv mysql-bin.000003 --stop-position=440267874 | tail -20

   或者usr/local/mysql-5.6.30/bin/mysqlbinlog --base64-output=decode-rows -vv mysql-bin.000003 > decode.log 

   （ 或者加上参数-d, --database=name 来进一步过滤）

   ```
   #160923 20:01:27 server id 1223307  end_log_pos 440267874 CRC32 0x134b2cbc      Delete_rows: table id 319 flags: STMT_END_F
   ### DELETE FROM `db_99ducj`.`tbuservcbgolog`
   ### WHERE
   ###   @1=10561502 /* INT meta=0 nullable=0 is_null=0 */
   ###   @2=1683955 /* INT meta=0 nullable=0 is_null=0 */
   ###   @3=90003 /* INT meta=0 nullable=0 is_null=0 */
   ###   @4=0 /* INT meta=0 nullable=0 is_null=0 */
   ###   @5='2016-09-23 17:02:24' /* DATETIME(0) meta=0 nullable=1 is_null=0 */
   ###   @6=NULL /* DATETIME(0) meta=0 nullable=1 is_null=1 */
   # at 440267874
   ```

   以上为检索出来的结果，事务语句为：delete from db_99ducj.tbuservcbgolog where @1=10561502 and @2=1683955 ...

   其中@1 @2 @3...分别对应表tbuservcbgolog的列名，填补上即可。

   我们可以逆向此SQL 将deleter 变成Insert,手动在从库上执行此Insert SQL,之后restart slave就好了。