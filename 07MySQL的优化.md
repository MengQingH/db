# MySQL的优化
MySQL的优化主要设计SQL语句及索引的优化、数据表结构的优化、系统配置的优化和硬件的优化四个方面，如下所示：
```
 高 ↑    硬件       |低
成本|    系统配置    |效果
    |    数据表结构  |
 低 |    SQL及索引   ↓高
```

# SQL语句的优化
SQL语句的优化主要包括三个问题，即如何发现有问题的SQL、如何分析SQL的执行计划以及如何优化SQL。
### 发现有问题的SQL
通过MySQL的慢查询日志对有效率问题的SQL进行监控。MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中相应时间超过阈值的语句，具体指运行时间超过long_query_time值的语句，会被记录到慢查询日志中。long_query_time的默认值是10，意思是运行时间10s以上的语句。慢查询日志的相关参数如下：
* slow_query_log：是否开启慢查询日志，1表示开启，0表示关闭。
* log_slow_queries：旧版（5.6以下版本）MySQL数据库慢查询日志存储路径。可以不设置该参数，系统则会默认给一个缺省的文件hostname-slow.log 
* slow_query_log_file：新版（5.6及以上版本）MySQL数据库慢查询日志存储路径。可以不设置该参数，系统则会默认给一个缺省的文件hostname-s1ow.1og 
* long_query_time：慢查询阈值，当查询时间多于设定的i值时，记录日志。
* logqueries_not_using_indexes：未使用索引的查询也被记录到慢查询日志中（可选项）。
* log_output：日志存储方式。log_output='FILE'表示将日志存入文件，默认值是'FILE'。log_output='TAEIE'表示将日志存入数据库，这样日志信息就会被写入到mysql.slow_log表中。MySQL数据库支持同时两种日志存储方式，配置的时候以逗号隔开即可，如：log_output='FIE','TABLE'。日志记录到系统的专用日志表中，要比记录到文件耗费更多的系统资源，因此对于需要启用慢查询日志，又需要能够获得更高的系统性能，那么建议优先记录到文件。

通过慢查询日志，可以查询出执行次数多占用时间长的SQL，可以通过pt_query_disgest(一种MySQL慢日志分析工具)分析Rows examine项(MySQL执行器需要检查的行数)找出IO大的SQL，以及发现未命中索引的SQL，对于这些SQL，都是优化的对象。

### 使用explain查询和分析SQL的执行计划
使用explain关键字可以直到MySQL是如何处理SQL语句的，一遍分析查询语句或是表结构的性能瓶颈。通过explain关键字可以得到表的读取顺序，数据读取操作的操作类型，哪些索引可以使用，哪些索引可以实际使用，表之间的引用以及每张表有多少行被优化器查询等问题。当扩展列出现Using filesort和Using temporay，则表示SQL需要被优化了。

### 优化SQL语句
* 优化insert语句：一次插入多值。
* 尽量避免在where子句中使用!=或<>操作符，否则引擎将放弃使用索引而进行全表扫描。
* 尽量避免在where语句中对字段进行null值判断，否则将导致引擎放弃使用索引而进行全盘扫描。
* 优化嵌套查询：子查询可以被更有效率的join代替。
* 使用exists代替in。

# EXPLAIN关键字
explain关键字放在select语句的前面， 可以看见sql语句的执行情况。有以下几项内容：
1. id：sql执行顺序的标识，id相同时，从上向下执行，id不相同时，先执行id大的，再执行id小的。注：union的结果会放在一个临时表中，**临时表的id为null**。
2. **select_type**：select子句的类型
    * simple：简单的select查询，没有子查询或union
    * primary：查询中若包含任何复杂的子部分**最外层的select**被标记为PRIMARY
    * subquery：**包含在select中**的子查询（select (子查询) from···  该子查询为subquery
    * derived：**包含在from中**的子查询，MySQL会将结果存放在一个临时表中，也称为派生表（derived的英文含义）
    * union：union中的**第二个和后面的select语句**
    * union result：从union临时表检索的select
3. table：记录查询用到的表。
    * 当from子句中有子查询时，table列是**derivenN**格式，表示当前查询依赖id=N的查询，于是先执行id=N的查询。
    * 当有union时，UNION RESULT的table列的值为**union1,2**，1和2表示参与union的select行id。
4. partitions：
5. type：表示MySQL在表中**找到所需行的方式**，又称“访问类型”。常用的类型有：**ALL, index, range, ref, eq_ref, const, system, NULL**（从左到右，性能从差到好）
    * null：mysql能够在优化阶段分解查询语句，在执**行阶段不用再访问表或索引**。例如使用查找某个索引的最小值，直接在索引中就可以找到，不需要再访问表。
    * const, system：mysql能对查询的某部分进行优化并将其转化成一个常量（可以看show warnings的结果）。用于primary key或unique key的所有列与常数比较时，所以表最多有一个匹配行，读取1次，速度比较快。
    * eq_ref：primary key 或 unique key 索引的所有部分被连接使用 ，最多只会返回一条符合条件的记录。
    * ref：相比 eq_ref，不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行。
    * ref_or_null：类似ref，但是可以搜索值为NULL的行。
    * range：使用一个索引来检索给定范围的行，通常出现在in, between, > <等操作中。
    * index：只扫描索引
    * all：全表扫描，需要遍历全表来获取需要的行，这种情况下就需要添加索引了。
6. possible_keys：搜索表的时候**可能使用哪个索引（不一定使用）**。如果为null，则可以为表创建相关的索引。MySQL中可能会出现possible_keys有列，但是key显示为null的情况，是因为表中的数据不多，MySQL认为此索引帮助不大，而选择全表扫描。
7. key：显示了MySQL**实际采用的索引**。
8. key_len：显示了mysql在索引里使用的字节数，通过这个值可以算出具体使用了索引中的哪些列。例如MySQL使用了联合索引a_b，a b都是int型，而key_len的值为4，就可以得知使用了a索引。
9. rows：MySQL**查询到结果需要读取或者检测的行数**，不是结果集的行数。
10. Extra：展示额外信息，常见值如下：
    * distinct：不查询重复行，一旦MySQL查询到和行相匹配的行，就不再搜索了。
    * Using index：这发生在对表的请求列都是同一索引的部分的时候，返回的列数据只使用了索引中的信息，而没有再去访问表中的行记录。性能较高。
    * Using where：mysql服务器将在存储引擎检索行后再进行过滤。就是先读取整行数据，再按 where 条件进行检查，符合就留下，不符合就丢弃。
    * Using temporary：mysql需要创建一张临时表来处理查询。出现这种情况一般是要进行优化的，首先是想到用索引来优化。


# 使用show profile
默认关闭，可以通过修改MySQL的系统变量来打开。可以使用查询profile来剖析执行查询的每个步骤以及花费的时间。
```sql
# 打开profile功能
set profiling = 1
# 查询所有的profile
show profiles；
# 查询某一个具体的profile，查询queryid为n的具体信息
show profile for query n;

```

# 索引的优化
某些情况下会导致引擎弃用索引进行全盘扫描：
* 在where中对字段进行null判断
* 以 % 开头的like语句，模糊匹配。
* or语句前后没有同时使用索引。
* 在where子句中使用!=或<>操作符
在where语句中对字段进行null值判断
* 数据类型出现隐式转化。


## limit的优化
随着数据量的增大，页数会越来越多，直接使用limit的效率也会越来越低。优化方法：避免数据量大时扫描过多的记录。
```sql
-- 查询1000000之后的30条记录
select * from stu order by id limit 1000000,30;

select * from stu where id > (select id from stu order by id limit 1000000,1) limit 30;
```
因为要取出所有字段内容，第一种需要跨越大量数据块并取出，而第二种通过**直接根据索引字段定位后，才取出相应内**容，效率自然大大提升。对limit的优化，不是直接使用limit，而是**首先获取到offset的id**，然后直接使用limit size来获取数据。




# 数据库表结构的优化
数据库表结构的优化包括选择合适的数据类型、表的范式的优化、表的垂直拆分和表的水平拆分等。
## 选择合适的数据类型
* 使用较小的数据类型解决问题。
* 使用简单的数据类型（mysql处理int要比varchar容易）。
* 尽可能使用not null定义字段。
* 尽可能避免使用text类型。
## 表的范式的优化
表的设计应该遵循三大范式
## 表的垂直拆分
把含有多个列的表拆分成多个表，解决表的宽度问题，具体包括以下几种拆分手段：
* 把不常用的字段单独放在一个表中。
* 把大字段独立放入一个表中。
* 把经常使用的字段放在一起。

好处：拆分后业务清晰、拆分规则明确、系统之间整合或扩展容易、数据维护简单。
## 表的水平拆分
表的水平拆分用于解决数据表中数据量过大的问题你，水平拆分每一个表的结构都是完全一致的。一般的，将数据平分到n张表中的常用方法包括以下两种：
* 针对ID进行hash运算，如果要拆分成5个表，mod(id,5)中取出0-4个值。
* 针对不同的hashID将数据存入不同的表中。
## 系统配置的优化
* 操作系统配置的优化：增加TCP支持的队列数。
* MySQL配置文件优化：Innodb缓存池设置和缓存池的个数。

