# 数据查询语言（DQL）

- DQL 操作会返回一个结果集
- 注意：
  1. 字符串和字符串格式的日期要用单引号括起来
  2. 数字类型直接书写
  3. 字符串是大小写不敏感的，可在需要区分大小写的字符串前添加 binary 关键字
  4. 日期值是格式敏感的

## 单表查询

- 语法

  ```sql
  (8)select (9)distinct select_list -- 确定选择的列
  (1)from left_table -- 确定查询哪一张表
  (3)join_tpye join right_table
  (2)on join_condition
  (4)where where_condition -- 确定选择的行（不能使用 select 中定义的别名）
  (5)group by group_by_list -- 对结果集分组（MySQL 中对查询做了加强，可以使用 select 中定义的别名）
  (6)with cube|rollup
  (7)having having_condition -- 对分组过滤
  (10)order by order_by_list -- 对结果集按照某列排序
  (11)limit start_number, limit_number -- 对结果集分页
  ```

- SQL 的执行顺序

  1. form：对 from 的左边的表和右边的表计算笛卡尔积，产生虚表 VT1
  2. on：对虚表 VT1 进行 on 筛选，只有那些符合 join_condition 的行才会被记录在虚表 VT2 中
  3. join：如果指定了 outer join（比如 left join、right join），那么保留表中未匹配的行就会作为外部行添加到虚拟表 VT2 中，产生虚拟表 VT3；如果 from 子句中包含两个以上的表的话，那么就会对上一个 join 连接产生的结果 VT3 和下一个表重复执行步骤 1~3 这三个步骤，一直到处理完所有的表为止
  4. where：对虚拟表 VT3 进行 where 条件过滤，只有符合 where_condition 的记录才会被插入到虚拟表 VT4 中
  5. group by：根据 group by 子句中的列，对 VT4 中的记录进行分组操作，产生 VT5
  6. cube | rollup：对表 VT5 进行 cube 或者 rollup 操作，产生表 VT6
  7. having：对虚拟表 VT6 应用 having 过滤，只有符合 having_condition 的记录才会被 插入到虚拟表 VT7 中
  8. select：执行 select 操作，选择指定的列，插入到虚拟表 VT8 中
  9. distinct：对 VT8 中的记录进行去重，产生虚拟表 VT9
  10. order by：将虚拟表 VT9 中的记录按照 order_by_list 进行排序操作，产生虚拟表 VT10
  11. limit：取出指定行的记录，产生虚拟表 VT11，并将结果返回

### 简单数据查询

```sql
select {*, 列 [[as] 别名], ...}
from   表名;

-- 如果列别名中使用关键字，或强制区分大小写，或有空格时，需使用 "" 或 '' 括起来

-- 使用 distinct 关键字从査询结果中清除重复行，必须放在要查询字段的开头
select distinct 列名, ...
from   表名;

-- 实现数学运算查询
-- 对数值型数据列可以使用算算术运算符（+  -  *  /）创建表达式
-- 两个日期之间可以进行减法运算，日期和数值之间可以进行加、减运算

-- 包括空值的任何算术表达式都等于空
```

### 使用 where 子句限定返回的记录

```sql
select <selectList> 
from   表名
where  条件表达式;

-- 优先级：所有的比较运算符、not（!）、and（&&）、or（||）

-- 可以使用 >、>=、<、<=、= 和 <> 等基本的比较运算符比较数值、字符串、日期之间的大小
-- SQL 中判断两个值是否相等的比较运算符是单等号，判断不相等的运算符是 <> 或 !=，SQL 中的赋值运算符是冒号等号（:=）

-- 特殊的比较运算符：between、in、is null、like

-- between 比较运算符，选出某一值域范围（闭区间）的记录
where 列名 between val1 and val2;

-- in 比较运算符，判断列的值是否在指定的集合中
where 列名 in (值1, 值2, ...);

-- is null 比较运算符、is not null 比较运算符，判断列的值是否为空
where 列名 is null;

-- like 比较运算符，执行通配查询/模糊查询
-- %  表示零或多个任意的字符
-- _  表示一个任意的字符
where 列名 like '_%';
```

### 使用 order by 子句将结果集进行排序

- asc：升序，**缺省**；desc：降序
- 注意：当 order by 子句中有使用了带引号的别名时，无法排序
- **如果数据量小则在内存中进行**，如果数据量大则需要使用磁盘

```sql
select <selectList> 
from   table_name
-- order by field(列名, val1, val2, val3) [asc|desc]：将获取出来的数据根据指定的顺序排序，即该列的其它值（视为相等） < val1 < val2 < val3，其中列名后面的参数自定义，不限制参数个数
order by 列1 [asc|desc], 列2 [asc|desc], field(列3, 值1, 值2, 值3, ...) [asc|desc], ...;
```

### 使用 limit 子句进行分页查询

- `limit {[offset,] row_count]` 或 `limit row_count OFFSET offset}`
- 使用两个自变量时，offset 指定返回的**第一行的偏移量**（初始行的偏移量为 0），row_count 指定**返回的行数的最大值**
- 使用一个自变量时，row_count 指定**从结果集合的开头返回的行数**，即 `limit n` 与 `limit 0, n` 等价

```sql
-- MySQL 特有
-- limit 子句中不能进行数学运算
-- beginIndex：从结果集中的哪一条索引开始显示（beginIndex 从 0 开始）
-- beginIndex = (当前页数 - 1) * pageSize
-- pageSize：页面大小（每页最多显示多少条记录）
select  <selectList>
from    表名
[where    condition]
limit   beginIndex, pageSize;
```

### 使用 group by 子句对结果集进行显式分组

- 将查询结果按某个字段或多个字段进行分组
- 分组后的结果集隐式按升序排列
- with rollup 关键字将会在所有记录的最后加上一条记录，该记录是上面所有记录的总和
- 如果查询列表中使用了聚合函数，或者 select 语句中使用了 group by 子句，则要求出现在 select 列表中的字段，**要么使用聚合函数或 group_concat() 包起来，要么必须出现在 group by 子句中**（MySQL 5.7 之后默认 `sql_mode = 'ONLY_FULL_GROUP_BY'`）
- 如果 group by 语句不指定 order by 条件会导致无谓的排序产生，所以当**不需要排序**建议添加 **order by null**

### 使用 having 子句对分组进行过滤

```sql
select [distinct] *|分组字段1[, 分组字段2, …] | 统计函数
from   表名
[where 条件]
group by 分组字段1[, 分组字段2, …] [with cube|rollup]
[having 过滤条件（可以使用统计函数）]
```

## 多表连接查询

- 如果表有别名，则不能再使用表的真名
- MySQL 执行关联查询的过程：MySQL 先在一个表中循环取出单条数据，然后再**嵌套循环**到下一个表中寻找匹配的行，依次下去，直到找到所有表中匹配的行为止。然后根据各个表匹配的行，返回査询中需要的各个列（**嵌套循环**关联）。
- nested loop join 步骤：确定一个驱动表（outer table），另一个表为 inner table，驱动表中的每一行与 inner 表中的相应记录 JOIN。类似一个嵌套的循环。适用于**驱动表的记录集比较小**（<10000）而且 **inner 表需要有有效的访问方法**（Index）。需要注意的是：JOIN 的顺序很重要，驱动表的记录集一定要小，返回结果集的响应时间是最快的。**减少内循环的次数，从而减少表加载次数**
- cost = outer access cost + (inner access cost * outer cardinality)

> - 从 MySQL 8.0.18 开始可以使用 hash join 进行连接查询（[Hash Join Optimization](https://dev.mysql.com/doc/refman/8.0/en/hash-joins.html)）
> - hash join 步骤：将两个表中较小的一个在内存中构造一个 hash 表（对 JOIN KEY），扫描另一个表，同样对 JOIN KEY 进行 hash 后探测是否可以 JOIN。适用于记录集比较大的情况。需要注意的是：如果 hash 表太大，无法一次构造在内存中，则分成若干个 partition，写入磁盘的 temporary segment，则会多一个写的代价，会降低效率。
> - cost = (outer access cost * # of hash partitions) + inner access cost

[![SQL JOINS](https://zeroone-bucket.oss-cn-beijing.aliyuncs.com/blog/20210423163504.png)](https://sdky.gitee.io/img/SQL_JOINS.png)图 2 SQL JOINS

### 内连接查询

```sql
-- 1. 隐式内连接：使用 where 指定连接条件，如等值连接（如果没有连接条件，会得到笛卡尔积）
select <selectList>
from   A, B
where  连接条件;

-- 2. 显式内连接查询
select <selectList>
from A [inner] join B on 连接条件;

-- 在做等值连接的时候，若 A 表中和 B 表中的列名相同，则可以简写:
select <selectList>
from A [inner] join B using(同名列);

-- 3. straight_join
-- straight_join 功能同 join 类似，但能让左边的表来驱动右边的表，能改变优化器对于联表查询的执行顺序
```

> STRAIGHT_JOIN is similar to JOIN, except that the left table is always read before the right table.
> This can be used for those (few) cases for which the join optimizer puts the tables in the wrong order.

### 外连接查询

- 左外连接（left [outer] join）：查询出 join 左边表的全部数据，右边的表不能匹配的数据使用 null 来填充数据
- 右外连接（right [outer] join）：查询出 join 右边表的全部数据，左边的表不能匹配的数据使用 null 来填充数据
- 全外连接（full [outer] join）：MySQL 不支持，可以通过 union + 左右连接查询来完成

```sql
select <selectList>
from A left|right [outer] join B on 连接条件;
```

### 自连接查询

- 如果同一个表中的**不同记录之间**存在主、外键约束关联，则需要使用自连接查询
- 本质：把一个表当成两个表来用，使用别名区分

## 子查询

- 子查询必须要位于圆括号中
  - **不能把同一个表既用于子查询的 from 子句，又用于更新目标**，如 `update t1 set column2 = (select max(column1) from t1);`

### 子查询分类

1. 标量子查询：子查询返回的结果是一个数据（一行一列），当成一个标量值使用，可以使用单行记录比较运算符

```
2. 列子查询：返回的结果是一列（一列多行），当成一个值列表，需要使用 in、any 和 all 等关键字，any 和 all 可以与 >、>=、<、<=、<>、= 等运算符结合使用
   in：与列表中的任意一个值相等
   any：与列表中的任意一个值比较，=any、>any、all、<all
```

3. 行子查询：返回的结果是一行（一行多列），多个字段合起来作为一个元素（行元素）参与运算

   ```sql
   select * from t1 where (column1, column2) = (select column1, column2 from t2);
   select * from t1 where row(column1, column2) = (select column1, column2 from t2);
   ```

4. 表子查询：返回的结果是多行多列，一般出现在 from 子句中当成临时表（行内视图）使用，使用前必须给临时表设置别名

   ```sql
   -- 在表 t1 中查找同时也存在于表 t2 中的所有的行
   select column1, column2, column3 from t1
   where (column1, column2, column3) in (select column1, column2, column3 from t2);
   ```

5. exists 子查询：返回的结果 1 或者 0（类似布尔操作），用于检查子查询是否至少会返回一行数据

   ```sql
   -- exists 对外表用 loop 逐条查询，每次查询都会查看 exists 的条件语句，当 exists 里的条件语句能够返回记录行时，条件就为真，将该条数据加入查询结果集中；如果没有记录行，条件为假，则丢弃该条数据
   select * from a where exists (select 1 from b where b.id = a.id)
   -- 等价于
   select * from a where id in (select id from b)
   -- 外查询表大，子查询表小，用 in；外查询表小，子查询表大，用 exists；若两表大小差不多，则差别不大
   -- in 子句在查询的时候有时会被转换为 exists 的方式来执行，变成逐条记录进行遍历
   -- MySQL 5.6 已针对子查询做了优化，应该都使用 in，https://dev.mysql.com/doc/refman/5.6/en/subquery-optimization.html
   -- 无论哪个表大，用 not exists 都比 not in 要快
   ```

## 集合运算

- 对两个 select 查询得到的结果集进行交（intersect）、并（union）和差（minus）运算
- 须满足：两个结果集所包含的数据列的数量必须相等，且数据列的数据类型也必须兼容
- MySQL 不支持 intersect、minus 运算

```sql
-- union/union all 用于把表纵向连接
select column_name(s) from table_name1
union|union all  -- union all 表示允许重复的行（性能高），而 union 会去掉重复的行
select column_name(s) from table_name2
```