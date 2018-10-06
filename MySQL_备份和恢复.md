## MySQL 备份和恢复

    第 41 天  【备份和基础(02)】

** 为什么要备份？ **

1、灾难恢复：硬件故障、软件故障、自然灾害、黑客攻击、误操作；

2、 测试：从线上数据库导出数据，导入到测试库进行测试；

** 备份要注意的要点： **

1、能容忍最多丢失多少数据；

2、恢复数据需要在多长时间内完成；

3、需要恢复哪些数据；

4、做还原测试，用于测试备份的可用性；

5、还原演练；

### 备份类型

根据备份的数据集区分：

- 完全备份：备份整个数据集；
- 部分备份：只备份数据子集；


根据备份时间轴的数据变化量区分：

- 完全备份
- 增量备份：仅备份最近一次完全备份或增量备份（如果存在增量）以来变化的数据；
- 差异备份：仅备份最近一次完全备份以来的变化的数据；


根据备份时业务系统能否使用区分：

- 热备份：线上系统在备份过程中，读写操作均可执行；
- 温备份：线上系统在备份过程中，读操作可执行，但写操作不可执行；
- 冷备份：线上系统在备份过程中，读写操作均不可执行；

MyISAM：温备，不能热备；
InnoDB：热备；

根据备份的数据类型区分：

- 物理备份：直接复制数据文件进行备份；
- 逻辑备份：从数据库中“导出”数据另存而进行的备份；与存储引擎无关；

** 备份时要考虑的因素： **

1、持锁多久；

2、备份过程的时长；

3、备份负载；

4、恢复过程的时长

** 备份什么？ **

1、 数据；

2、二进制日志、InnoDB的事务日志；

3、代码（存储过程、存储函数、触发器、事件调度器）；

4、服务器的配置文件；

** 设计备份方案：**

1、数据集：完全 + 增量；

2、备份手段：物理，逻辑；

** 备份工具：**

mysqldump：逻辑备份工具，适用所有存储引擎，温备；完全备份、部分备份；对InnoDB存储引擎支持热备；

cp,tar等文件系统复制归档工具：物理备份工具，适用所有存储引擎；冷备；完全备份，部分备份；

lvm2的快照：几乎热备；借助于文件系统管理工具实现物理备份；

mysqlhotcopy：几乎冷备；仅适用于MyISAM存储引擎；

** 备份工具的选择：**

mysqldump+复制binlog：
```
    mysqldump：完全备份；
    复制binlog中指定时间范围的event：增量备份；
```

lvm2快照+复制binlog：
```
    lvm2快照：使用cp或tar等做物理备份；完全备份；
    复制binlog中指定时间范围的event：增量备份；
```

xtrabackup：
```
    由Percona提供的支持对InnoDB做热备(物理备份)的工具；
    开源，支持完全备份、增量备份；
```

    第 42 天  【MySQL备份与恢复(01)】

#### 逻辑备份工具

mysqldump, mydumper, phpMyAdmin

Schema和数据存储在一起、巨大的SQL语句、单个巨大的备份文件；

缺点：对于较大的数据库备份较慢；

** mysqldump：**

客户端命令，通过mysql协议连接至mysqld服务器；

```
mysqldump [options] [db_name [tbl_name ...]]

   shell> mysqldump [options] db_name [tbl_name ...]  //不会备份创建数据库，仅备份创建表；
   shell> mysqldump [options] --databases db_name ...
   shell> mysqldump [options] --all-databases

[root@node1 ~]# mysqldump -uroot --databases hellodb > hellodb2.sql -p
Enter password: 
[root@node1 ~]# ll hellodb2.sql 
-rw-r--r--. 1 root root 9460 6月   5 10:13 hellodb2.sql
```

MyISAM表：支持温备；要锁定备份库，而后启动备份操作；

锁定方法：
```
--lock-all-tables, -x：锁定所有库的所有表；
--lock-tables：对于每个单独的数据库，在启动备份之前锁定其所有表；

对InnoDB表一样生效，实现温备；
```

InnoDB：支持热备；
```
    --single-transaction：启动一个大的单一事务实现备份，不会锁表；

[root@node1 ~]# mysqldump -uroot --single-transaction --databases hellodb > hellodb3.sql -p
Enter password: 
[root@node1 ~]# ll hellodb3.sql 
-rw-r--r--. 1 root root 9460 6月   5 10:15 hellodb3.sql
```

其它选项：
```
-E，--events：备份自定数据库相关的所有event scheduler；
-R，--routines：备份指定数据库相关的所有存储过程和存储函数；
--triggers：备份表相关的触发器；

--master-data[=#]：记录备份时，数据库系统使用的binlog文件及position（end_log_pos)；
    1：记录为CHANGE MASTER TO语句，此语句不被注释；
    2：记录为注释的CHANGE MASTER TO语句;
        -- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000004', MASTER_LOG_POS=7841;

--flush-logs:
    锁定表完成后，执行flush logs命令，滚动binlog日志；
```

注意：二进制日志文件不应该与数据文件放置在同一块磁盘；

练习：有一100MB级别的数据库：

（1）备份脚本；

（2）制作备份策略；

** 基于lvm2的备份：** 

1、请求锁定所有表：
```
mysql> FLUSH TABLES WITH READ LOCK;
```

2、记录二进制日志文件及事件位置：
```
mysql> FLUSH LOGS;       // 滚动日志
mysql> SHOW MASTER STATUS;

# mysql -e 'SHOW MASTER STATUS' > /PATH/TO/SIMEFILE
    # mysql -e 'SHOW MASTER STATUS;' > /mysql_backup/binlog-`date +%F`
```

3、创建快照：
```
lvcreate -L # -s -p r -n NAME /DEV/VG_NAME/LV_NAME
    
        -L #：大小要根据备份过程中业务数据会增长的大小而定；
        -s：创建快照卷；
        -p：策略，r(只读)；
        -n：指明快照卷名称；

    # lvcreate -L 500M -s -p r -n mydata-snap  /dev/myvg/mydata
```

4、释放锁：
```
mysql> UNLOCK TABLES;
```

5、挂载快照卷，执行数据备份：
```
# mount -r /dev/myvg/mydata-snap /mnt
# cp -a /mnt/mydata/* /mysql_backup/
```

6、备份完成后，删除快照卷：
```
# umount /mnt
# lvremove /dev/myvg/mydata-snap
```

7、制定好策略，通过原卷备份二进制日志：
```
# cat /mysql_backup/binlog-2017-07-03 
File    Position    Binlog_Do_DB    Binlog_Ignore_DB
mysql-bin.000007    245 

# mysqlbinlog --start-position=245 /mydata/data/mysql-bin.000007 > /mysql_backup/binlog-`date +%F`.sql
```
