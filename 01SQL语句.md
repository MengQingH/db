## 数据库中的数据类型
数据类型   |含义
:---       |:---
char(n)：   |长度为n的定长字符串
varchar(n)  |长度为n的变长字符串
clob        |字符串大对象
blob        |二进制大对象
int         |长整数，4字节
smallint    |短整数，2字节
bigint      |大整数，8字节
real        |取决于机器精度的单精度浮点数
double precision|取决于机器精度的双精度浮点数
boolean     |逻辑布尔量
date        |日期，包含年月日，格式为YYYY-MM-DD
time        |时间，包含时分秒，格式为HH:MM:SS
timestamp   |时间戳类型，包含年月日时分秒
interval    |时间间隔类型

## 基本表
### 基本表的定义
```sql
create table 表名 (列名 数据类型 [列级完整性约束条件],
                列名 数据类型 [列级完整性约束条件],
                ···,
                [表级完整性约束条件]);
```
### 基本表的修改
删除列时：
* cascade：当该列被其他对象引用，则自动删除引用了该列的其他对象
* restrict：如果该列别其他对象引用，拒绝删除该列
```sql
alter table 表名
-- 新增列或添加列级完整性约束
add 新列名 数据类型 [完整性约束]
add 表级完整性约束
-- 删除列
drop 列名 [cascade|restrict]
drop constraint 完整性约束
-- 修改列
alter column 列名 数据类型;
```
### 删除基本表
```sql
drop table 表名 [restrict|cascade];
```

## 索引
### 建立索引
表示在该表上建立索引：
* 次序可选ASC（升序）、DESC（降序），默认为ASC。
* unique表示此索引的每一个索引值只对应唯一的数据记录。
* cluster表示要建立的索引是聚簇索引
```sql
create [unique] [cluster] index 索引名
on 表名(列名 [次序][,列名 索引]···);
```
### 修改和删除索引
```sql
-- 修改
alter index 旧索引名 rename to 新索引名;
-- 删除
drop index 索引名;
```

## 数据更新
### 插入数据
1. 插入元组：values中的字符串常量需要用单引号括起来。
    ```sql
    insert into 表名[(属性列···)] values (常量···);
    ```
2. 插入子查询结果：子查询不仅可以嵌套在select语句中，也可以嵌套在insert语句中插入批量数据。
    ```sql
    insert into 表名[(属性列···)] 子查询;
    ```
### 修改数据
```sql
update 表名 set 列名=表达式 [,列名=表达式] where 条件;
```
### 删除数据
```sql
delete from 表名 where 条件;
```

### delete drop truncate的区别：
* DELETE语句执行删除的过程是每次从表中**删除一行**，并且同时将该行的删除操作作为事务记录在日志中保存以便进行进行回滚操作。

* TRUNCATE TABLE 则一次性地从表中**删除所有的数据**并**不把单独的删除操作记录记入日志保存**，删除行是**不能恢复**的。并且在删除的过程中不会激活与表有关的删除触发器。**执行速度快**。
    * 只删除数据，表结构、约束、索引还是存在。
    * 
* DROP表示删除整个表，包括结构和数据。

delete语句为DML（data maintain Language),这个操作会被放到 rollback segment中,事务提交后才生效。如果有相应的 tigger,执行的时候将被触发。
truncate、drop是DLL（data define language),操作立即生效，原数据不放到 rollback segment中，不能回滚。

## 角色相关
1. 创建角色。刚创建的角色是空的，没有任何内容，可以用grant进行授权。
    ```sql
    create role 角色名;
    ```
2. 给角色授权。可以将权限授予一个或几个角色。
    ```sql
    grant 权限 [,权限]··· 
    on 对象类型 对象名 
    to 角色1[,角色2];
    ```
3. 将一个角色授予其他角色或用户。被授予的角色（角色3）就拥有了授予角色（角色1，角色2）的权限的总和。如果加上了with admin option，被授予的角色还可以把权限再授予其他用户或角色。
    ```sql
    grant 角色1 [,角色2]··· 
    to 角色3[,用户]···
    [with admin option];
    ```
4. 角色权限的收回。
    ```sql
    revoke 权限1 [,权限2]···
    on 对象类型 对象名 
    from 角色1[,角色2];
    ```

## 视图
### 建立视图
```sql
create view 视图名 [列名 [,列名]···]
as 子查询
[with check option];
```
* 子查询可以是任意的select语句。
* with check option表示对视图进行update、insert和delete操作时要保证更新、插入、删除的行满足视图定义中的谓词条件（即子查询中的表达式）。
* 组成视图的属性列名或者全部省略，或者全部指定，没有第三种选择，如果隐藏了属性列名那么该视图由子查询中select子句目标列中的诸字段组成。
* 在以下三种情况下必须指定组成视图的所有列名：
    * 某个目标列不是单纯的属性名，而是聚集函数或者列表达式；
    * 多表连接时选出了几个同名列作为视图的字段。
    * 需要在视图为某个列启用新的更合适的名字。
### 删除修改视图
视图删除后将视图的定义从数据字典中删除，如果该视图上还导出了其他视图，则使用cascade可以将该视图和它导出的视图一起删除。
```sql
drop view 视图名 [cascade]
```

## 其他语句
```sql
SHOW DATABASES                                //列出 MySQL Server 数据库。
SHOW TABLES [FROM db_name]                    //列出数据库数据表。
SHOW CREATE TABLES tbl_name                    //导出数据表结构。
SHOW TABLE STATUS [FROM db_name]              //列出数据表及表状态信息。
SHOW COLUMNS FROM tbl_name [FROM db_name]     //列出资料表字段
SHOW FIELDS FROM tbl_name [FROM db_name]，DESCRIBE tbl_name [col_name]。
SHOW FULL COLUMNS FROM tbl_name [FROM db_name]//列出字段及详情
SHOW FULL FIELDS FROM tbl_name [FROM db_name] //列出字段完整属性
SHOW INDEX FROM tbl_name [FROM db_name]       //列出表索引。
SHOW STATUS                                  //列出 DB Server 状态。
SHOW VARIABLES                               //列出 MySQL 系统环境变量。
SHOW PROCESSLIST                             //列出执行命令。
SHOW GRANTS FOR user                         //列出某用户权限
```