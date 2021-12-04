# MySQL 数据库操作

## 数据库操作

- MySQL 书写规则
  1. 在 MySQL 数据库中，**SQL 语句大小写不敏感**
  2. SQL 语句可单行或多行书写，以 ; 或 \g 或 \G 作为每条命令的结束符
  3. 在 SQL 语句中，关键字不能跨多行或缩写
  4. 不能使用 MySQL 中的关键字作为标识符，**除非用反引号（`）引起来**
- cmd 命令
  1. 启动 MySQL 服务：net start MySQL
  2. 停止 MySQL 服务：net stop MySQL
  3. 连接 MySQL：mysql -u 账户 -p [密码] -h 主机名 -P 端口
     如：`mysql -uroot -padmin -h127.0.0.1 -P3306`
     如果连接的数据库服务器在本机上，并且端口是 3306，则可以简写：mysql -uroot -padmin
  4. 导出 SQL 脚本：mysqldump -u 账户 -p 密码 数据库名称 > 脚本文件地址
     `mysqldump -uroot -padmin jdbcdemo > C:/shop_bak.sql`
     对于全部是 InnoDB 引擎的库，建议使用 mysqldump 备份数据库时添加 --single-transaction 参数（在导数据前会启动一个事务，来确保拿到一致性视图，由于 MVCC 的支持，这个过程中数据可以正常更新）
  5. 导入 SQL 脚本：mysql -u 账户 -p 密码 数据库名称 < 脚本文件地址
     `mysql -uroot -padmin jdbcdemo < C:/shop_bak.sql`
- MySQL 数据库系统中 4 个系统自带的数据库（information_schema、mysql、performance_schema、sys）不能被修改
- [关键字和保留字](https://dev.mysql.com/doc/refman/5.7/en/keywords.html)

> MySQL 通讯的数据包大小默认为 4 M，可通过 max_allowed_packet 修改
>
> 查看数据库或表的数据大小：
> `select concat(round(sum(data_length/1024/1024),2),'MB') as data from information_schema.tables where table_schema='db_name' and table_name='table_name';`

## 常见 MySQL 存储引擎

1. MyISAM：拥有较高的插入、查询速度，但不支持事务，不支持外键，不支持行级锁
2. InnoDB：支持事务，支持外键，支持行级锁，支持热备份，比 MyISAM 处理效率差，且会占用更多的磁盘空间以保留数据和索引

## [MySQL 支持的列类型](https://dev.mysql.com/doc/refman/5.7/en/data-types.html)

- MySQL 记录行长度最大为 64K（The maximum row size for the used table type, not counting BLOBs, is **65535**）

- [Data Type Storage Requirements](https://dev.mysql.com/doc/refman/5.7/en/storage-requirements.html)

- 整数类型（可分为有符号和无符号两种）：tinyint、int 或 integer、bigint（可指定**位宽**）

- 浮点数类型（可分为有符号和无符号两种）：float(p)、float(M,D)、double(M,D)、decimal(M,D)

- 定点数类型（可分为有符号和无符号两种）：decimal(M,D)

  > p 表示精度（以位数表示）
  > M 表示该值的总位数（精度），D 表示小数点后面的位数（标度）
  > float 和 double 在不指定精度时，**默认**会按照实际的精度来显示
  > decimal 在不指定精度时，默认整数为 10，小数为 0
  > 浮点数类型应使用 decimal，禁止使用 float 和 double

- 字符类型（字符使用**单引号**引起来）：char(**字符**数)、varchar(**字符**数) 、text 系列类型、json（使用`json 列->'$.键'`或`json_extract(json 列 , '$.键')`）（str 的字符个数：char_length(str)）

  > **char、varchar 定义的长度不要超过 5000，否则应该定义为 text**
  >
  > varchar 需要使用 1 或 2 个额外字节记录字符串的长度：如果列的最大长度小于或等于 255 字节，则只使用 1 个字节表示，否则使用 2 个字节
  >
  > 使用 varchar(5) 和 varchar(200) 存储 'hello' 的**占用磁盘的存储空间是一样的**，但 varchar(200) 会**消耗更多的内存**，因为 MySQL 通常会分配固定大小的内存块来保存内部值（尤其是使用临时表进行排序会操作时，会消耗更多的内存）

- 日期时间类型（允许“不严格”语法：任何标点符都可以用做日期部分或时间部分之间的间割符，或者没有间割符）
  date （与时区无关，值使用**单引号**括起来，检索和显示格式为 'YYYY-MM-DD'，如 '2017-01-01'）
  datetime （与时区无关，值使用**引号**括起来，检索和显示格式为 'YYYY-MM-DD HH:MM:SS'）
  timetamp：时间戳，显示的值依赖于时区，MySQL 服务器、操作系统以及客户端连接时设置的时区

  > current_timestamp：当要向表执行插入操作时，如果有个 timestamp 或 datetime 类型的字段的默认值为 `current_timestamp`，则无论这个字段有没有 set 值都插入当前系统时间
  > on update current_timestamp：使用 `on update current_timestamp` 放在 timestamp 或 datetime 类型的字段后面，在数据发生**更新**时该字段将自动更新时间

- 二进制类型：bit （一般用来存储 0 或 1，Java 中的 boolean/Boolean 类型的值）（可指定**位宽**）

[![MySQL支持的列类型](https://zeroone-bucket.oss-cn-beijing.aliyuncs.com/blog/20210423155950.jpeg)](https://zeroone-bucket.oss-cn-beijing.aliyuncs.com/blog/20210423155950.jpeg)图 1 MySQL支持的列类型

## 数据库管理语句

1. 查看数据库服务器存在哪些数据库：show databases;
2. 进入指定的数据库：use 数据库名;
3. 创建指定名称的数据库：create database [if not exists] 数据库名 [default charset utf8mb4] [default collate utf8mb4_general_ci];
4. 更改数据库的默认字符集：alter database 数据库名 default character set utf8mb4 default collate utf8mb4_general_ci;
5. 删除数据库：drop database 数据库名;
6. 查看当前的所有连接：**show full processlist;**

## 常用性能突发事件分析命令

- 当前数据库的运行的所有线程：`show processlist;`
- 当前运行的所有事务：`select * from information_schema.INNODB_TRX;`
- 当前事务出现的锁的语句信息：`select * from information_schema.INNODB_LOCKS;`
- 锁等待的对应关系：`select * from information_schema.INNODB_LOCK_WAITS;`
- 查看哪些表在使用中：`show open tables where In_use >0;`
- Innodb 状态：`show engine innodb status;`
- 锁性能状态：`show status like 'innodb_row_lock_%';`