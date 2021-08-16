# 慢sql日志

**慢查询日志配置** 

默认情况下slow_query_log的值为OFF，表示慢查询日志是禁用的，可以通过设置slow_query_log的值来开启，如下所示：

```java
mysql> show variables  like '%slow_query_log%';
+---------------------+-----------------------------------------------+
| Variable_name       | Value                                         |
+---------------------+-----------------------------------------------+
| slow_query_log      | OFF                                           |
| slow_query_log_file | /home/WDPM/MysqlData/mysql/DB-Server-slow.log |
+---------------------+-----------------------------------------------+
2 rows in set (0.00 sec)
 
mysql> set global slow_query_log=1;
Query OK, 0 rows affected (0.09 sec)
 
mysql> show variables like '%slow_query_log%';
+---------------------+-----------------------------------------------+
| Variable_name       | Value                                         |
+---------------------+-----------------------------------------------+
| slow_query_log      | ON                                            |
| slow_query_log_file | /home/WDPM/MysqlData/mysql/DB-Server-slow.log |
+---------------------+-----------------------------------------------+
2 rows in set (0.00 sec)
 
mysql> 
```

**long_query_time**

那么开启了慢查询日志后，什么样的SQL才会记录到慢查询日志里面呢？  这个是由参数long_query_time控制，默认情况下long_query_time的值为10秒，可以使用命令修改，也可以在my.cnf参数里面修改。关于运行时间正好等于long_query_time的情况，并不会被记录下来。也就是说，在mysql源码里是判断大于long_query_time，而非大于等于。从MySQL  5.1开始，long_query_time开始以微秒记录SQL语句运行时间，之前仅用秒为单位记录。如果记录到表里面，只会记录整数部分，不会记录微秒部分。

```
mysql> show variables like 'long_query_time%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)
 
mysql> set global long_query_time=4;
Query OK, 0 rows affected (0.00 sec)
 
mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)
```

如下是一个记录到日志文件中的慢sql的示例

　　![img](https://img.jbzj.com/file_images/article/201709/201709190917475.png)

**记录慢查询日志到表**

配置：需要添加一个log_output的配置，就可以将慢查询记录到表中了

![img](https://img.jbzj.com/file_images/article/201709/201709190917476.png)

mysql库下面有一个默认的slow_log表，可以直接将`slow_query_log_file = slow_log`，即可将慢查询日志记录到表中。

　　![img](https://img.jbzj.com/file_images/article/201709/201709190917477.png)

记录到的slow sql如下，可以发现sql_text是一个二进制的信息，并非原始的sql文本

　　![img](https://img.jbzj.com/file_images/article/201709/201709190917478.png)

可以通过CONVERT函数转换一下即可。

　　![img](https://img.jbzj.com/file_images/article/201709/201709190917479.png)

**关于慢查询记录到日志文件和表中的区别：**

1、慢查询记录到日志文件和表中，记录本身差别不大，如果是记录在表中，慢查询的执行时间信息无法精确到微妙，

2、如果将慢查询信息记录在表中，方便查询，但因为是结构化的数据，可能会比记录在慢查询日志文件中（平面文本文件）要慢一点点（个人猜测），如果是记录到文件，需要mysqldumpslow工具解析。

3、慢查询不记录执行失败的查询，比如long_query_time设置为10（10秒钟），一个查询超过了10秒钟，但是因为其他原因执行失败，MySQL的慢查询将无法记录此查询信息。
