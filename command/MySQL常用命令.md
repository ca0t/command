[TOC]



## MySQL常用命令

 

将常用到的MySQL的一些命令在这里做个记录，便于查询和更新。

mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个默认密码，这时候第一次登录mysql数据库时需要修改这个生成的默认密码。

 

```sql
systemctl start mysqld.service 启动
systemctl stop mysqld.service 停止
systemctl restart mysqld.service 重启

#mysql配置文件
etc/my.cnf 
```

 

### 数据库管理命令

#### 1、登入数据库相关

**登录数据库**

```sql
#登录数据库
mysql -h localhost -u root -p dbName

#本地
mysql -u root -p
>输入密码


#查看数据库信息
select version();   #查看MySQL当前的版本
show databases;     #查看有哪些数据库
use testdb;         #切换数据库
show tables;        #查看表
show engines;       #查看存储引擎
```

#### 2、用户相关

```sql
select user,host,password from mysql.user; #查询mysql数据库用户

CREATE USER 'test@%' IDENTIFIED BY "123456";                #创建用户
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';#创建用户


#某个用户从某台机器访问本台mysql服务器
GRANT ALL ON dbname.tablename to user1@192.168.67.1;        #用户授权
#mysql8授权：
GRANT SELECT, INSERT ON db.* TO 'username'@'%';             #用户授权
GRANT ALL PRIVILEGES ON db.* TO 'username'@'localhost';     #用户授权

show grants for user_name@localhost;                #查看用户权限


ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';     #修改用户密码

#上面新建用户，修改用户密码，给用户授权后使用
flush privileges;                                   #刷新权限
```

 

#### 3、数据库配置相关

```sql
#创建数据库
#utf8_bin 区分大小写，utf8_general_ci 不区分大小写
CREATE DATABASE IF NOT EXISTS test DEFAULT CHARSET utf8 COLLATE utf8_bin; 

#查看 MySQL 加载配置文件的顺序
#后面的配置会覆盖前面相同的配置项
mysqld --help --verbose | grep -A 1 'Default options'

#查看MySQL的一些配置
#比如查看MySQL的数据库文件存放在那个目录就可以用下面的命令
show variables where Variable_name ='datadir'; 

#关闭更新时的安全模式
SET SQL_SAFE_UPDATES=0;

#关闭自动提交
SET AUTOCOMMIT=0; # 只对当前会话生效

#设置自增从 10000 开始
ALTER TABLE tableName AUTO_INCREMENT=10000; 

#关闭外键 约束
SELECT @@FOREIGN_KEY_CHECKS; 
SET FOREIGN_KEY_CHECKS=1; // 开启外键约束 
SET FOREIGN_KEY_CHECKS=0; // 关闭外键约束 

#查看大小写是否敏感
#mysql中控制数据库名和表名的大小写敏感由参数lower_case_table_names控制，为0时表示区分大小写，为1时，表示将名字转化为小写后存储，不区分大小写并且以_ci（大小写不敏感）、_cs（大小写敏感）或_bin 大小写敏感
SHOW VARIABLES LIKE '%case%'; 

# 查看安装的plugin
show plugins; 
```

 

#### 4、日志相关

```sql
#查看参数值
show variables like "%log%"; 

#查看错误日志的存放位置
show variables like '%log_error%';

#刷新binlog
# 在mysql中flush logs操作会生成一个新的binlog文件 
flush logs;

#查看最后一个bin日志
show master status; 

#清空所有日志
reset master; 
```

 

#### 5、数据库空间查看

```sql
#查看数据库占用空间
SELECT 
    table_schema,
    SUM(data_length + index_length) / 1024 / 1024 AS total_mb,
    SUM(data_length) / 1024 / 1024 AS data_mb,
    SUM(index_length) / 1024 / 1024 AS index_mb,
    COUNT(*) AS tables,
    CURDATE() AS today
FROM
    information_schema.tables
GROUP BY table_schema
ORDER BY 2 DESC; 

#查看某个数据库表中的情况
SELECT 
    table_name,
    (data_length / 1024 / 1024) AS data_mb,
    (index_length / 1024 / 1024) AS index_mb,
    ((data_length + index_length) / 1024 / 1024) AS all_mb,
    table_rows
FROM
    information_schema.tables
WHERE
    table_schema = 'db_name'; 

#查看某个库的具体情况
show table status from db_name;

#查看数据库中表碎片的情况
SELECT TABLE_SCHEMA
      ,TABLE_NAME 
      ,ENGINE
      ,ROUND(((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024), 2) AS SIZE_MB
      ,ROUND(DATA_FREE/1024/1024,2) AS FREE_SIZ_MB
FROM information_schema.TABLES 
WHERE DATA_FREE >=100*1024*1024
ORDER BY FREE_SIZ_MB DESC;
```

 

### 数据库操作SQL语句

#### 1、表操作

```sql
#查看创建表的sql语句
show create table t1; 

#表删除
truncate table1;        #删除一张表里所有的数据
drop table table1;      #删除一张表

#表复制
create table bs_test2 like bs_test1 # 复制表结构 
INSERT INTO bs_test1 SELECT * FROM bs_test2; #复制表中的数据 

#给表添加注释
ALTER TABLE 表名 COMMENT '注释的内容'
#查看某个表的注释
SELECT table_name,table_comment FROM information_schema.tables where table_name='表名' 

#查询出 数据库 中所有的 表信息
select table_name from information_schema.tables where table_schema='数据库名' and table_type='base table'; select * from information_schema.tables where table_schema='数据库名' and table_type='base table'; 

#查看一张表 或 一条sql语句的执行情况
(DESC 或 EXPLAIN) DESC SELECT * FROM bs_member DESC bs_member


#创建 json 列，创建虚拟列 user_name，address
CREATE TABLE `test` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `info` json NOT NULL,
  `user_name` varchar(128) GENERATED ALWAYS AS (json_extract(`info`,'$.name')) VIRTUAL,
  `address` varchar(128) GENERATED ALWAYS AS (json_extract(`info`,'$.address')) STORED,
  PRIMARY KEY (`id`),
  KEY `user_name_index` (`user_name`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8
```

#### 2、表查询

```sql
#排序
SELECT * FROM `tableName` ORDER BY colName desc, colName asc 
#将字符型的 数字（ID_）转成数字来排序 +0 或 *1
SELECT * FROM `tableName` order by ID_+0 asc; 

#分组
SELECT `colName `,count(*) as total FROM `tableName` GROUP BY colName 

#查询 最后 10 条数据（id 自增）
select * from wy_user order by id desc limit 10;

#按某一列的值的长度查找，UTF8 编码中文长度为 3
SELECT * FROM `bs_member` WHERE city like '%北京%' and length(city) > 7


#查询重复记录
SELECT
    id,email
FROM
    wy_user2
WHERE
    id IN (
        SELECT
            id
        FROM
            wy_user2
        GROUP BY
            email
        HAVING
            count(email) > 1
    )
    
    
#删除重复记录，并保留id最小的记录
delete from wy_user2 where id not in (select minid from (select min(id) as minid from wy_user2 group by email) b);
```

#### 3、索引相关

```sql
#添加主键索引
#它 是一种特殊的唯一索引，不允许有空值
ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` ) 

#添加唯一索引
#与"普通索引"类似，不同的就是：索引列的值必须唯一，但允许有空值。
ALTER TABLE `table_name` ADD UNIQUE ( `column` ) 

#添加普通索引
ALTER TABLE `table_name` ADD INDEX index_name ( `column` ) 
ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` ) 

#添加全文索引
#仅可用于 MyISAM 表，针对较大的数据，生成全文索引很耗时好空间 (适用于，大块数据，如文章内容)
ALTER TABLE `table_name` ADD FULLTEXT ( `column`) 

#查看表的索引信息
show index from table1

#删除索引
DROP INDEX index_name ON table_name 

#查询时禁止使用主键索引
select * from tableName ignore index(PRI)

#查询时禁止使用某些索引
select * from tableName ignore index(PRI, indexName) 

#查询时强制使用主键索引
select * from tableName force index(PRI)

#查询时强制使用某些索引
select * from tableName force index(PRI, indexName)

```

#### 4、数据备份与导入

```sql
#数据库备份
/usr/local/mysql/bin/mysqldump -u root -p lemon > lemon.sql /usr/local/mysql/bin/mysqldump -u root -p dbName tableName --where="..." > table.sql 

#将 CSV 文件导入 Mysql 中
LOAD DATA LOCAL INFILE '/home/db-friend/aff11.csv' into table user1 FIELDS TERMINATED BY ',' lines terminated by '\n' ignore 1 lines (pwsid,email,country,sex,birthday,state,zip,ip); 
```