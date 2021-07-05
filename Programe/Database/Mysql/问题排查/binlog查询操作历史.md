# 通过mysqlbinlog查询操作历史

- **问题描述**：

  线上环境出现数据莫名奇妙被删除的异常,通过代码无法定位问题,找不到是哪里的代码删除了数据。

- 解决办法：

  使用的是mysql数据库且做了主从同步，因此存在binlog日志记录操作历史,通过binlog日志来寻找蛛丝马迹。

- 解决步骤:

  - 复制出近期的binlog日志

  ```shell
  cp /data/binlog/mysql-bin.* /tmp/haoze
  ```

  - 编写bash遍历脚本

  ```shell
  #!/bin/bash进入临时目录
  cd /tmp/haozevim
  # 开始循环遍历目录
  for path in `ls . |grep mysql-bin.0`
  do
  #记录一些基础信息，比如当前遍历的mysqlbinlog日志
  echo ">>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>查询结果" >> z_grep.log
  echo "begin ..."
  echo $path >> z_grep.log
  # 需要搜索出csh_repayment_register_dist表的记录，记录到总日志z_grep.log
  mysqlbinlog --base64-output=DECODE-ROWS -v $path |grep -i -C 100 delete |grep -C 100 csh_repayment_register_dist >> z_grep.log
  echo "end." >>z_grep.log
  done
  
  
  #另一个mysqlbinlog查看日志语句-查看维值删除的语句
  mysqlbinlog --no-defaults  --base64-output=decode-rows  -v /tmp/haoze/mmysqlbinlog  --no-defaults  --base64-output=decode-rows  -v /tmp/haoze/mysql-bin.002505|sed -n '/###fnd_dimension_value v/,/COMMIT/p' > /tmp/delete.txtysql-bin.002505 | sed -n '/### DELETE FROM `hap_prod`.`fnd_dimension_value`/,/COMMIT/P' | sed -n 's\### \\p' | sed "s/\/\*.*\*\///g" | sed 's/`//g' > /tmp/1.txt
  #进入mysql命令行可以通过内置表查看binlog
  show binlog events in 'mysql-bin.002505' from 56326173 limit 10
  #重复执行binlog事务
  mysqlbinlog --start-position=36506331 --stop-position=56326204 /tmp/haoze/mysql-bin.002505|mysql -uroot -p xinchaoDB2020!@#
  ```

  - 执行脚本vim

  ```shell
  bash -x find_guolichao.sh
  ```
  
  - 分析查找出来的binlog日志
  
  根据的last_committed来区分事务，找到删除的语句所在的事务,通过对执行语句的判断,可以找到大概是审核拒绝时删除了不属于拒绝单据的分配行。
  
  ```
  #210302 13:48:01 server id 2  end_log_pos 51478159 CRC32 0xe28d58a5 	GTID	last_committed=6417	sequence_number=6418	rbr_only=yes
  /*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
  SET @@SESSION.GTID_NEXT= '0ab842fc-ed93-11ea-ac5b-005056954519:27975308'/*!*/;
  # at 51478159
  #210302 13:48:01 server id 2  end_log_pos 51478248 CRC32 0x587e7c06 	Query	thread_id=8395	exec_time=0	error_code=0
  SET TIMESTAMP=1614664081/*!*/;
  BEGIN
  /*!*/;
  # at 51478248
  # at 51479147
  #210302 13:48:01 server id 2  end_log_pos 51479271 CRC32 0x63cad789 	Table_map: `hap_prod`.`csh_repayment_register_hd` mapped to number 3165401
  # at 51479271
  #210302 13:48:01 server id 2  end_log_pos 51479788 CRC32 0xc8c4c565 	Update_rows: table id 3165401 flags: STMT_END_F
  ### UPDATE `hap_prod`.`csh_repayment_register_hd`
  ### WHERE
  ###   @1=208
  ###   @2='XCHK21030100428'
  ###   @3=10956
  ###   @4=267
  ###   @5=6831
  ###   @6=17403
  ###   @7=1
  ###   @8=1
  ###   @9=5
  ###   @10=1
  ###   @11='2021-03-01 00:00:00'
  ###   @12=1614590599
  ###   @13=1614590599
  ###   @14=730.00
  ###   @15='138219320637'
  ###   @16='西部大区开年红包还款730元。'
  ###   @17='SUBMITTED'
  ###   @18=NULL
  ###   @19=NULL
  ###   @20=NULL
  ###   @21=NULL
  ###   @22=NULL
  ###   @23='2021-03-01 17:23:19'
  ###   @24=21341
  ###   @25=21341
  ###   @26='2021-03-01 17:23:18'
  ###   @27=2
  ###   @28=-1 (18446744073709551615)
  ###   @29=-1 (18446744073709551615)
  ###   @30=21341
  ###   @31=NULL
  ### SET
  ###   @1=208
  ###   @2='XCHK21030100428'
  ###   @3=10956
  ###   @4=267
  ###   @5=6831
  ###   @6=17403
  ###   @7=1
  ###   @8=1
  ###   @9=5
  ###   @10=1
  ###   @11='2021-03-01 00:00:00'
  ###   @12=1614590599
  ###   @13=1614590599
  ###   @14=730.00
  ###   @15='138219320637'
  ###   @16='西部大区开年红包还款730元。'
  ###   @17='CASHIER_REJECTED'
  ###   @18=NULL
  ###   @19=NULL
  ###   @20=NULL
  ###   @21=NULL
  ###   @22=NULL
  ###   @23='2021-03-01 17:23:19'
  ###   @24=21341
  ###   @25=19427
  ###   @26='2021-03-02 13:48:01'
  ###   @27=3
  ###   @28=-1 (18446744073709551615)
  ###   @29=-1 (18446744073709551615)
  ###   @30=19427
  ###   @31=NULL
  # at 51479788
  # at 51479911
  #210302 13:48:01 server id 2  end_log_pos 51480013 CRC32 0xbfe2e80a 	Table_map: `hap_prod`.`csh_repayment_register_dist` mapped to number 3171884
  # at 51480013
  #210302 13:48:01 server id 2  end_log_pos 51480168 CRC32 0x34950cd6 	Delete_rows: table id 3171884 flags: STMT_END_F
  ### DELETE FROM `hap_prod`.`csh_repayment_register_dist`
  ### WHERE
  ###   @1=248
  ###   @2=142
  ###   @3=192410
  ###   @4=192412
  ###   @5=12.80
  ###   @6='N'
  ###   @7=215462
  ###   @8=215464
  ###   @9='N'
  ###   @10=NULL
  ###   @11=NULL
  ###   @12='2021-02-26 18:24:13'
  ###   @13=19427
  ###   @14=19427
  ###   @15='2021-02-26 18:24:25'
  ###   @16=2
  ###   @17=-1 (18446744073709551615)
  ###   @18=-1 (18446744073709551615)
  ###   @19=19427
  # at 51480168
  # at 51480291
  #210302 13:48:01 server id 2  end_log_pos 51480393 CRC32 0xe55b88d7 	Table_map: `hap_prod`.`csh_repayment_register_dist` mapped to number 3171884
  # at 51480393
  #210302 13:48:01 server id 2  end_log_pos 51480548 CRC32 0x3fb00d48 	Delete_rows: table id 3171884 flags: STMT_END_F
  ### DELETE FROM `hap_prod`.`csh_repayment_register_dist`
  ### WHERE
  ###   @1=232
  ###   @2=150
  ###   @3=184610
  ###   @4=184612
  ###   @5=142.92
  ###   @6='N'
  ###   @7=213550
  ###   @8=213552
  ###   @9='N'
  ###   @10=NULL
  ###   @11=NULL
  ###   @12='2021-02-24 16:44:44'
  ###   @13=24483
  ###   @14=24483
  ###   @15='2021-02-24 16:44:49'
  ###   @16=2
  ###   @17=-1 (18446744073709551615)
  ###   @18=-1 (18446744073709551615)
  ###   @19=24483
  # at 51480548
  # at 51480671
  ························
  ······················
  ···············
  #210302 13:48:01 server id 2  end_log_pos 51490601 CRC32 0xfaf1faf5 	Table_map: `hap_prod`.`exp_document_history` mapped to number 3167104
  # at 51490601
  #210302 13:48:01 server id 2  end_log_pos 51490787 CRC32 0xa384ad7d 	Write_rows: table id 3167104 flags: STMT_END_F
  ### INSERT INTO `hap_prod`.`exp_document_history`
  ### SET
  ###   @1=2157752
  ###   @2='CSH_REPAYMENT_REGISTER'
  ###   @3=208
  ###   @4=NULL
  ###   @5='CASHIER_REJECTED'
  ###   @6=19427
  ###   @7=19427
  ###   @8='2021-03-02 13:48:02'
  ###   @9=1614664082
  ###   @10=1614664082
  ###   @11=''
  ###   @12='2021-03-02 13:48:02'
  ###   @13=19427
  ###   @14=19427
  ###   @15='2021-03-02 13:48:02'
  ###   @16=1
  ###   @17=-1 (18446744073709551615)
  ###   @18=-1 (18446744073709551615)
  ###   @19=19427
  ###   @20=''
  ###   @21=NULL
  # at 51490787
  # at 51491304
  #210302 13:48:01 server id 2  end_log_pos 51491468 CRC32 0x2fae34ec 	Table_map: `hap_prod`.`cux_doc_audit_return_itf` mapped to number 3165294
  # at 51491468
  #210302 13:48:01 server id 2  end_log_pos 51491693 CRC32 0xb78c9928 	Write_rows: table id 3165294 flags: STMT_END_F
  ### INSERT INTO `hap_prod`.`cux_doc_audit_return_itf`
  ### SET
  ###   @1=157368
  ###   @2=NULL
  ###   @3='NEW'
  ###   @4='CSH_REPAYMENT_REGISTER'
  ###   @5='XCFSC'
  ###   @6='XCHK21030100428'
  ###   @7='OVERRULE'
  ###   @8='XC10962'
  ###   @9='2021-03-02 13:48:02'
  ###   @10='单据审核拒绝！'
  ###   @11='138219320637'
  ###   @12='T001'
  ###   @13=-1 (18446744073709551615)
  ###   @14=-1 (18446744073709551615)
  ###   @15=1
  ###   @16=19427
  ###   @17='2021-03-02 13:48:02'
  ###   @18=19427
  ###   @19=19427
  ###   @20='2021-03-02 13:48:02'
  ###   @21=NULL
  ###   @22=NULL
  ###   @23=NULL
  ###   @24=NULL
  ###   @25=NULL
  ###   @26=NULL
  ###   @27=NULL
  ###   @28=NULL
  ###   @29=NULL
  ###   @30=NULL
  ###   @31=NULL
  ###   @32=NULL
  ###   @33=NULL
  ###   @34=NULL
  ###   @35=NULL
  ###   @36=NULL
  # at 51491693
  # at 51492597
  #210302 13:48:01 server id 2  end_log_pos 51492761 CRC32 0x104f36b1 	Table_map: `hap_prod`.`cux_doc_audit_return_itf` mapped to number 3165294
  at 51492761
  #210302 13:48:01 server id 2  end_log_pos 51493186 CRC32 0x94ace208 	Update_rows: table id 3165294 flags: STMT_END_F
  ### UPDATE `hap_prod`.`cux_doc_audit_return_itf`
  ### WHERE
  ###   @1=157368
  ###   @2=NULL
  end.		