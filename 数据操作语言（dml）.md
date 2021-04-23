# 数据操作语言（DML）

- DML 操作返回受影响的行数

## INSERT 语法

```sql
-- 空值用 null 表示
-- MySQL 特有的语法：用一条 insert 语句插入多条数据记录
-- 如果使用了 ignore，在执行语句时出现的错误被当作警告处理，例如，一个要插入的行与现有的 unique 索引或 primary key 值重复，则该行不被插入，且不会出现错误
-- 如果指定了 on duplicate key update，并且插入行后会导致在一个 unique 索引或 primary key 中出现重复值，则执行旧行 update（受影响行的值：1-如果被作为新行插入；2-如果原有的行被更新；3-如果原有的行被设置为其当前值）
-- values() 函数只在 insert...update 语句中有意义，其它时候会返回 null
insert [ignore] into tb1_name (col_name, ...)
values ({expr | default}, ...), (...), ...
[ on duplicate key update col_name = expr, ... ]
[ on duplicate key update col_name = expr | values(col_name) , ... ]

insert into tb1_name
set col_name = {expr | default}, ...
[ on duplicate key update col_name = expr, ... ]

-- 插入查询结果
insert into tb1_name (col_name, ...)
select ...
[ on duplicate key update col_name = expr, ... ]
```

## REPLACE 语法

```sql
-- 如果表中的一个旧记录与一个用于 primary key 或一个 unique 索引的新记录具有相同的值，则在新记录被插入之前，旧记录被删除
-- 返回被删除和被插入的行数的和
replace [into] tbl_name [(col_name, ...)]
values ({expr | default}, ...), (...), ...

replace [into] tbl_name
set col_name = {expr | default}, ...

-- 替换查询结果
replace [into] tbl_name [(col_name,...)]
select ...
```

## UPDATE 语法

- update 语句只支持更新前多少行，不支持从某行到另一行，即只能 `limit 30`，不能 `limit 20, 10`

- 单表语法

  ```sql
  update  tb1_name
  set col_name1 = expr1 [, col_name2 = expr2 ...]
  [where where_definition]
  [order by ...]
  [limit row_count]
  ```

- 多表语法

  ```sql
  update table_references
  set col_name1 = expr1 [, col_name2 = expr2 ...]
  [where where_definition]
  
  update items, month
  set items.price = month.price
  where items.id = month.id;
  -- 多表 update 语句可以使用在 select 语句中允许的任何类型的联合，比如 left join
  -- order by 或 limit 不能与多表 update 同时使用
  ```

## DELETE 语法

1. 单表语法

   ```sql
   delete from tb1_name
   [where where_definition]
   [order by ...]
   [limit row_count]
   ```

2. 多表语法

   ```sql
   -- 只删除列于 from 子句之前的表中的对应的行
   delete tb1_name [, tb2_name, ...]
   from table_references
   [where where_definition]
   
   -- 只删除列于 from 子句之中（在 using 子句之前）的表中的对应的行
   delete from tb1_name [, tb2_name, ...]
   using table_references
   [where where_definition]
   
   delete t1, t2 from t1, t2, t3 where t1.id = t2.id and t2.id = t3.id;
   delete from t1, t2 using t1, t2, t3 where t1.id = t2.id and t2.id = t3.id;
   -- 多表 delete 语句除了使用逗号操作符的内部联合外，还可以使用 select 语句中允许的所有类型的联合，比如 left join
   ```