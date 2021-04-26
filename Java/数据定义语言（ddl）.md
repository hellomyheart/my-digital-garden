# 数据定义语言（DDL）

1. 创建表（创建表前先进入某一数据库）

   ```sql
     -- 每个列定义之间用逗号隔开，最后一个列定义后**不需要**写逗号
    create table student (
      --列名 列类型 [默认值]
      id bigint primary key auto_increment,
      name varchar(20),
      age int
    ) engine=InnoDB auto_increment=1 default charset=utf8mb4;
   
    -- 复制表结构（不包括外键约束）
    create table student_bak like student;
    -- 只复制表数据到新表
    create table student_bak select * from student；
   ```

2. 修改表结构

   ```sql
     -- 增加列定义
    alter table 表名 add (
      新列名 列类型 [默认值],
      ...
    );
   
    -- 修改列定义
    alter table 表名
    modify 列名 列类型 [默认值];
   
    -- 删除列
    alter table 表名
    drop 列名;
   ```

3. 查看当前数据库中存在哪些表：show tables;

4. 查看表结构：desc 表名; 或 describe 表名;

5. 查看表的详细定义（定义表的 SQL 语句）：show create table 表名;

6. 删除表：drop table 表名;

7. 截断表（删除表里的全部数据，但**保留表结构**）：truncate 表名;

8. 修改表的存储引擎：alter table 表名 ENGINE='InnoDB';

9. 表的约束（列级约束），关键字之间不用加逗号

   1. default '值'：默认值
   2. not null：非空约束，该列的内容不能为空
   3. unique：唯一约束，在该表中，该列的内容必须唯一，但可以出现多个 null 值
   4. primary key：主键约束，相当于非空约束和唯一约束
   5. auto_increment[=值]：自增长，**只能用于指定整型主键列**，默认从 1 开始，步长为 1，向该表插入记录时可**不为该列指定值，或指定为 null 或 0**（可以通过设置 `sql_mode = 'NO_AUTO_VALUE_ON_ZERO'` 将自增值设置为 0）（**只能有一个自增列，且必须被定义为主键**）
   6. foreign key (外键列) references 主表 (参考列)：外键约束
      从表外键列的值必须在主表被参照列的值范围之内，或者为 null，要求从表和主表的存储引擎都为 InnoDB

## Online DDL

- MySQL 5.5 版本之前（不包括 5.5），对于索引的添加或者删除的这类 DDL 操作，MySQL 数据库的操作过程为：

  1. 创建一张新的**临时表**，表结构为通过命令 ALTER TABLE 新定义的结构
  2. 把原表中数据逐行复制到临时表，在此期间会阻塞 DML
  3. 删除原表
  4. 把临时表重名为原来的表名

- MySQL 5.5 版本中对添加索引操作引入了新特性 Fast Index Create（FIC 特性）

- MySQL 5.6 版本开始支持 **Online DDL**（在线数据定义）操作，其允许某些 DDL 操作的同时，还允许 DML 操作

- 语法

  ```sql
  ALTER TABLE tbl_name
  | ADD {INDEX|KEY} [index_name] [index_type] (index_col_name, ...) [index_option] ...
  ALGORITHM [=] {DEFAULT|INPLACE|COPY}
  LOCK [=] {DEFAULT|NONE|SHARED|EXCLUSIVE}
  ```

  - ALGORITHM 指定了创建或删除索引的算法：
    - COPY 表示按照 MySQL 5.5 版本之前的工作模式，即创建临时表的方式
    - INPLACE 表示索引创建或删除操作不需要创建临时表
    - DEFAULT 表示根据参数 old_alter_table 来判断是通过 INPLACE 还是 COPY 的算法，该参数的默认值为 OFF，表示采用 INPLACE 的方式
  - LOCK 部分为索引创建或删除时对表添加锁的情况
    - NONE：执行索引创建或者删除操作时，对目标表不添加任何的锁，即事务仍然可以进行读写操作，不会收到阻塞。因此这种模式可以获得最大的并发度
    - SHARE：执行索引创建或删除操作时，对目标表加上一个 S 锁，对于并发地读事务，依然可以执行，但是遇到写事务，就会发生等待操作
    - EXCLUSIVE：执行索引创建或删除操作时，对目标表加上一个 X 锁，读写事务都不能进行，因此会阻塞所有的线程
    - DEFAULT：先判断当前操作是否可以使用 NONE 模式，若不能，则判断是否可以使用 SHARE 模式，最后判断是否可以使用 EXCLUSIVE 模式（即 DEFAULT 会通过判断事务的最大并发性来判断执行 DDL 的模式）

- InnoDB 存储引擎实现 Online DDL 的原理是在执行创建或者删除操作的同时，将 INSERT、UPDATE、DELETE 这类 DML 操作日志写入到一个**缓存**中，待完成索引创建后再将**重做**应用到表上，以此达到数据的一致性（缓存的大小由参数 innodb_online_alter_log_max_size 控制，默认的大小为 128MB）

- [Online DDL Support](https://dev.mysql.com/doc/refman/5.6/en/innodb-online-ddl-operations.html)

