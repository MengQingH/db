## 聚合函数
* **COUNT(col)**   统计查询结果的行数
* **MIN(col)**   查询指定列的最小值
* **MAX(col)**   查询指定列的最大值
* **SUM(col)**   求和，返回指定列的总和
* **AVG(col)**   求平均值，返回指定列数据的平均值

## 其他函数
* CAST(expression AS TYPE)：cast()函数可以将任何类型的值转换为具有指定类型的值，目标类型可以是一下类型之一：BINARY，CHAR，DATE，DATETIME，TIME，DECIMAL，SIGNED，UNSIGNED，JSON
    ```sql

    ```


## 流程控制函数
* **CASE value WHEN [compare-value] THEN result [WHEN [compare-value] THEN result ...] [ELSE result] END**：对值进行判断，并返回结果
    ** SELECT CASE 11 WHEN 1 THEN 'one' WHEN 2 THEN 'two' ELSE 'more' END  --more;
* **CASE WHEN[test1] THEN [result1]...ELSE [default] END**：对testn进行判断，如果testn为真，则返回resultN，否则返回default。
* **if(expr1, expr2, expr3)** ：如果expr1是true，则返回expr2，否则返回expr3。
    * SELECT IF(1<2,'yes ','no');

* **Strcmp(str1,str2)**：如果str1大于str2，返回1，等于返回0，小于返回-1。根据前面元素的次序进行排序。
* **IFNULL(arg1,arg2)**：   如果arg1不是空，返回arg1，否则返回arg2

## 数学函数
* **abs(X)**  ：返回x的绝对值
* **mod(n,m)  n%m** ：返回n除以m的余数
* **floor(X)**  ：向下取整，返回不大于X的最大整数
* **ceiling(X)** ：向上取整，返回不小于X的最大整数
* **round(X)**  ：返回X四舍五入后的结果
* **format(X,n)** ：返回X四舍五入保留n位小数的结果
* sign(X)：返回X的符号，负数、0、正数分别返回-1、0、1

## 字符串函数
* **concat(str1, str2)** ：返回参数先后连接后的字符串，如果参数存在null，返回null。可以有超过两个参数，也可以传入数字参数。
* char_length(str)：返回字符串s的字符数
* concat(str1,str2)：将字符串str1、str2等多个字符串合并为一个字符串
* upper(s) lower(s)：将s的所有字母变成大写/小写
* **length(str)** ：返回字符串str的长度。
* **locate(substr, str)** ：返回子串substr在字符串str出现的第一个位置，如果不存在，返回0。
* **instr(str, substr)**  ：返回字串substr在str中出现的第一个位置。
* **left(str, len)** ：返回字符串str最左边的len个字符。
* **right(str, len)**  ：返回str最右边的len个字符。
* **substring(str, pos)**  ：返回str从第pos个字符到结尾的子串
* **trim(str)**   ：删除字符串左右两边的空格。
    * ltrim(str)：删除字符串左边的空格
    * rtrim(str)：删除字符串右边的空格
* **replace(str, from_str, to_str)**  ：替换字符串str中的from_str为to_str
* **repeat(str, count)**  ：返回str重复count次的记过，如果count<=0，返回空字符串，如果str或者count是null，返回null
* **reverse(str)**  ：前后颠倒str的顺序
* **insert(str, begin, len, newstr)**  ：str的begin位置开始len个字符长的子串由newstr代替。

## 时间和日期函数
* **curdate() current_date()**：以 YYYY-MM-DD 或YYYYMMDD格式返回今天日期值，取决于函数在一个字符串还是数字上下文被使用。
* **curtime() current_time()**：以 HH:MM:SS 或HHMMSS格式返回当前时间值
* **now()**  ：以 YYYY-MM-DD HH:MM:SS 或YYYYMMDDHHMMSS格式返回当前的日期和时间
* **unix_timestamp( [time] )**：以UNIX时间戳的形式返回当前时间/传入的时间
* from_unixtime(10位时间戳)：把unix时间戳转换为 YYYY-MM-DD HH:MM:SS 格式
* **datediff(date1, date2)**：返回date1开始 到 date2结束之间的天数
* **timediff(date1, date2)**：返回date1到date2相差的时间
* **date_format()**
* **date_add(date, INTERVAL num type)  date_sub(date, INTERVAL num type)**： 或者直接进行+-操作，进行日期增加或减少的操作，可以精确到秒
    * SELECT '1997-12-31 23:59:59' + INTERVAL 1 SECOND; 
    * SELECT INTERVAL 1 DAY + '1997-12-31';
    * SELECT DATE_ADD('1997-12-31 23:59:59', INTERVAL 1 SECOND);
* **adddate  subdate**
* **subtime addtime**

## json函数
Mysql中的json分为jsonArray和jsonObject，$代表整个json对象，在标识json中的数据时用下标或者键值：
```
例如：[3, {"a": [5, 6], "b": 10}, [99, 100]]，那么：
$[0] ：3
$[1] ： {"a": [5, 6], "b": 10}
$[2] ：[99, 100]
$[3] ： NULL
$[1].a ：[5, 6]
$[1].a[1] ：6
$[1].b ：10
$[2][0] ：99
```
比较规则：json中的数据可以用 =, <, <=, >, >=, <>, !=, and <=> 进行比较。但json里的数据类型可以是多样的，那么在不同类型之间进行比较时，就有优先级了，高优先级的要大于低优先级的（可以用JSON_TYPE()函数查看类型）。优先级从高到低如下：
```
BLOB  BIT  OPAQUE  DATETIME  TIME  DATE  BOOLEAN  ARRAY  OBJECT  STRING  INTEGER  DOUBLE  NULL
```
### 创建函数
* JSON_ARRAY(val1,val2,val3...) ：生成一个包含指定元素的json数组。
* JSON_OBJECT(key1,val1,key2,val2...)：生成一个包含指定K-V对的json object。如果有key为NULL或参数个数为奇数，则抛错。
* CONVERT(json_string,JSON) ：把json字符串转换成json对象

### 查询函数
* JSON_CONTAINS(json_doc, val[, path])：查询json文档是否在指定path包含指定的数据，包含则返回1，否则返回0。如果有参数为NULL或path不存在，则返回NULL。
    ```sql
    mysql> SET @j = '{"a": 1, "b": 2, "c": {"d": 4}}';
    mysql> SET @j2 = '1';
    mysql> SELECT JSON_CONTAINS(@j, @j2, '$.a'); -- 1
    mysql> SELECT JSON_CONTAINS(@j, @j2, '$.b'); -- 0
    ```
* JSON_CONTAINS_PATH(json_doc, one_or_all, path[, path] ...)：查询是否存在指定路径，存在则返回1，否则返回0。如果有参数为NULL，则返回NULL。one_or_all只能取值"one"或"all"，one表示只要有一个存在即可；all表示所有的都存在才行。
    ```sql
    mysql> SET @j = '{"a": 1, "b": 2, "c": {"d": 4}}';
    mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a', '$.e'); --1
    mysql> SELECT JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.e'); --0
    mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.c.d'); --1
    ```
* JSON_EXTRACT(json_doc, path[, path] ...) ：从json文档里抽取数据。如果有参数有NULL或path不存在，则返回NULL。如果抽取出多个path，则返回的数据封闭在一个json array里。在MySQL 5.7.9+里可以用"->"替代，在MySQL 5.7.13+，还可以用"->>"表示去掉抽取结果的"号
    ```sql
    mysql> SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[1]'); --20
    mysql> SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[1]', '$[0]'); --[20,10]
    mysql> SELECT JSON_EXTRACT('[10, 20, [30, 40]]', '$[2][*]'); --[30,40]

    mysql> SELECT * FROM TEST WHERE JSON_EXTRACT(c, "$.id") > 1 ORDER BY JSON_EXTRACT(c, "$.name"); 
    mysql> SELECT * FROM TEST WHERE c->"$.id" > 1 ORDER BY c->"$.name";

    -- JSON_UNQUOTE(column -> path)  JSON_UNQUOTE( JSON_EXTRACT(column, path) )  可以去掉json结果的"号，也可以直接使用column->>path

    ```
* JSON_KEYS(json_doc[, path])：获取json文档在指定路径下的所有键值，返回一个json array。如果有参数为NULL或path不存在，则返回NULL。
    ```sql
    mysql> SELECT JSON_KEYS('{"a": 1, "b": {"c": 30}}');  --["a", "b"]
    ```
* JSON_SEARCH(json_doc, one_or_all, search_str[, escape_char[, path] ...])：查询包含指定字符串的paths，并作为一个json array返回。如果有参数为NUL或path不存在，则返回NULL。one_or_all："one"表示查询到一个即返回；"all"表示查询所有。search_str：要查询的字符串。可以用LIKE里的'%'或‘_’匹配。path：在指定path下查。
    ```sql
    mysql> SET @j = '["abc", [{"k": "10"}, "def"], {"x":"abc"}, {"y":"bcd"}]';
    mysql> SELECT JSON_SEARCH(@j, 'one', 'abc'); --"$[0]"
    mysql> SELECT JSON_SEARCH(@j, 'all', 'abc'); --["$[0]", "$[2].x"]
    --限定范围
    mysql> SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$'); --"$[1][0].k"
    mysql> SELECT JSON_SEARCH(@j, 'all', '10', NULL, '$[*][0].k'); --"$[1][0].k"
    --使用通配符
    mysql> SELECT JSON_SEARCH(@j, 'all', '%b%'); --["$[0]", "$[2].x", "$[3].y"]
    ```

### 修改函数
* JSON_ARRAY_APPEND(json_doc, path, val[, path, val] ...) ：在指定path的json array尾部追加val。如果指定path是一个json object，则将其封装成一个json array再追加。如果有参数为NULL，则返回NULL。
    ```sql
    mysql> SET @j = '["a", ["b", "c"], "d"]';
    mysql> SELECT JSON_ARRAY_APPEND(@j, '$[1]', 1); --["a", ["b", "c", 1], "d"]
    mysql> SELECT JSON_ARRAY_APPEND(@j, '$[0]', 2); --[["a", 2], ["b", "c"], "d"]

    mysql> SET @j = '{"a": 1, "b": [2, 3], "c": 4}';
    mysql> SELECT JSON_ARRAY_APPEND(@j, '$.b', 'x'); --{"a": 1, "b": [2, 3, "x"], "c": 4}
    ```
* JSON_ARRAY_INSERT(json_doc, path, val[, path, val] ...) ：在path指定的json array元素插入val，原位置及以右的元素顺次右移。如果path指定的数据非json array元素，则略过此val；如果指定的元素下标超过json array的长度，则插入尾部。
    ```sql
    mysql> SET @j = '["a", {"b": [1, 2]}, [3, 4]]';
    mysql> SELECT JSON_ARRAY_INSERT(@j, '$[1]', 'x'); -- ["a", "x", {"b": [1, 2]}, [3, 4]]
    mysql> SELECT JSON_ARRAY_INSERT(@j, '$[1].b[0]', 'x'); --["a", {"b": ["x", 1, 2]}, [3, 4]] 
    ```
* JSON_INSERT(json_doc, path, val[, path, val] ...) ：在指定path下插入数据，如果path已存在，则忽略此val（不存在才插入）。
* JSON_REPLACE(json_doc, path, val[, path, val] ...)：替换指定路径的数据，如果某个路径不存在则略过（存在才替换）。如果有参数为NULL，则返回NULL。
* JSON_SET(json_doc, path, val[, path, val] ...)：设置指定路径的数据（不管是否存在）。如果有参数为NULL，则返回NULL。
    ```sql
    mysql> SET @j = '{ "a": 1, "b": [2, 3]}';
    mysql> SELECT JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]'); --{"a": 1, "b": [2, 3], "c": "[true, false]"}
    mysql> SELECT JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]'); --{"a": 10, "b": [2, 3]}  
    ```
* JSON_MERGE(json_doc, json_doc[, json_doc] ...)：merge多个json文档。规则如下：
    * 如果都是json array，则结果自动merge为一个json array；
    * 如果都是json object，则结果自动merge为一个json object；
    * 如果有多种类型，则将非json array的元素封装成json array再按照规则一进行mege。
* JSON_REMOVE(json_doc, path[, path] ...) ：移除指定路径的数据，如果某个路径不存在则略过此路径。如果有参数为NULL，则返回NULL。
* JSON_UNQUOTE(val) ：去掉val的引号。如果val为NULL，则返回NULL。