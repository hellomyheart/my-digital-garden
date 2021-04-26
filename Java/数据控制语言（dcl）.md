# 数据控制语言（DCL）

###  账户管理语句

```sql
-- 创建用户
create user 'guest'@'localhost' identified by '1234';
-- 修改用户密码
alter user 'guest'@'localhost' identified by '123';
-- 当前用户设置密码
set password = password('123');
-- 为当前服务器主机上的一个特定账户设置密码
set password for 'guest'@'localhost' = password('123');

-- 授予用户权限
-- 注意：在授权操作之后，需要使用 flush privileges 命令刷新权限
grant 权限 on 数据库.数据库对象 to 用户名@'主机'
identified by '密码' -- 创建用户，设置密码
with grant option; -- 允许用户继续授权

grant all [privileges] on *.* to guest@'localhost' identified by '1234' with grant option;

-- 创建一个超级管理员，用户名为 dev，密码为 1234，只能在 192.168.%.% 登陆，可以给别人授权
grant all privileges on `test_schema`.* to dev@'192.168.%.%' identified by '1234' with grant option;
flush privileges;

-- 查看用户的权限
show grants [for root@localhost]

-- 回收对用户的授权
revoke 权限 on 数据库对象 from 用户;
revoke all on *.* from guest@localhost;

-- 删除用户
drop user 用户名@'主机';
drop user guest@'%';
```

### SHOW 语法

- 提供有关数据库、表、列或服务器状态的信息
- show [full] processlist：查看哪些线程正在运行，如果不使用 full 关键词，则只显示每个查询的前 100 个字符（如果有 process 权限，可以看到所有线程，否则只能看到自己的线程）
  - User：发送 sql 语句到当前 MySQL 使用的是哪个用户
  - Host: 发送 sql 语句到当前 MySQL 的主机 ip 和端口
  - Db: 连接哪个数据库
  - Command: 连接状态，一般是 sleep（休眠空闲）、query（查询）、connect（连接）
  - **Time**: 连接执行时间
  - **State**: 当前 sql 语句的执行状态，如 Checking table（正在检查数据表）、Sending data（正在处理 select 查询的记录，返回数据）、Sorting for group（正在为 group by 做排序）、Sorting for order（正在为 order by 做排序）、Updating（正在搜索匹配的记录，并且修改它们）、Locked（被其它查询锁住了）
- show [global | session] variables [like 'pattern']：查看服务器系统变量的值，如 'character%'、'%query_cache%'、'validate_password%'
- show [global | session] status [like 'pattern']：查看服务器状态信息，如 'Qcache%'、'Innodb*buffer_pool*%'
- show master logs、show binary logs：列出服务器中的二进制日志文件

### SET 语法

- 用于向用户变量或系统变量赋值
- 用户变量可以被写作 @var_name，并可以进行如下设置：`SET @var_name = expr;`
- 系统变量可以被作为 var_name 或 @@var_name 引用到 SET 语句中
- 全局变量与会话变量
  - 在名称的前面添加 GLOBAL 或 @@global，以指示该变量是全局变量；在名称前面添加 SESSION、@@session 或 @@，以指示该变量是会话变量（LOCAL 和 @@local 是 SESSION 和 @@session 的同义词）；如果没有修改符，则 SET 设置**会话变量**
  - `SET GLOBAL sort_buffer_size=1000000, SESSION sort_buffer_size=1000000;`
  - `SET @@global.sort_buffer_size=1000000, @@local.sort_buffer_size=1000000;`
  - 如果使用 SESSION 设置一个系统变量，则该值仍然有效，直到**当前会话结束**为止；如果使用 GLOBAL 来设置一个系统变量，则该值被记住，并被用于新的连接，直到**服务器重新启动**为止；如果要进行永久式变量设置，应该把其放入一个选项文件
- `set names utf8mb4;` 把**会话**系统变量 character_set_client、character_set_connection、character_set_results 设置为给定的字符集（不会修改 character_set_server、character_set_database）

### 其它管理语句

- kill [connection | query] thread_id：终止线程，kill connection 与不含修改符的 kill 一样，它会终止与给定的 thread_id 有关的连接；kill query 会终止连接当前正在执行的语句，但是会保持连接的原状（如果有 super 权限，可以终止所有线程和语句，否则只能终止自己的线程）