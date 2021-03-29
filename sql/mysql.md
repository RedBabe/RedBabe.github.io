## 1. 了解SQL

### 数据库基础

> DBMS(Database Management System)，数据库管理系统。

表(table)：某种特定类型数据的结构化清单。

模式(schema)：关于数据库和表的布局和特性的信息。

列(column)：表中的一个字段。

数据类型(datatype)：所容许的数据的类型。

行(row)：表中的一条记录。

主键(primary key)：一列或一组列，其值可以区分每一行。任意列都可以为主键，只需满足：1. 任意两行具有不同的主键值；2. 主键值不为NULL。使用主键的最好习惯是：**主键值是从不更新的**。

### SQL

> Structured Query Language，结构化查询语言。

大多的DBMS都支持SQL语言，但是实现上会有些许不同。

## 2. MySQL简介

书写习惯：MySQL关键字大写，列名和表名小写。

## 3. 使用MySQL

### 连接

```bash
# 启动
mysql.server start
# 连接
mysql -uroot
```

### 查看数据库

```sql
-- 显示所有数据库
show databases;
# 选择数据库
use sys;
# 显示所有表
show tables;
# 显示指定表的列：两种方法都可以
show columns from metrics;
describe metrics;
```

### SHOW的其它用法

```sql
-- 显示服务器状态信息
show status;
-- 显示创建数据库或表的SQL语句
show create database sys;
show create table metrics;
-- 显示用户安全权限
show grants;
-- 显示错误或警告
show errors;
show warnings;
```

## 4. 检索数据

### SELECT基本使用

```sql
-- 从表中检索指定列
select Variable_name from metrics;
-- 检索多个列
select Variable_name, Variable_value from metrics;
```

### 检索不同行

```sql
-- 指定列必须具有不同值，值相同的行会被舍去；会检索所有行
select distinct Variable_value from metrics;
```

### 限制行数量

```sql
-- 显示前5行 [1, 5]
select Variable_name from metrics limit 5;
-- 显示6-15行 [6, 15]：两种写法都对
select Variable_name from metrics limit 5,15;
select Variable_name from metrics limit 15 offset 5;
```

### 使用完全限定的表名和列名

```sql
select metrics.Variable_name from sys.metrics limit 5;
```

有时候会用上完全限定的名称。

## 5. 排序检索数据

### 排序显示

```sql
-- 按照指定列顺序显示
select Variable_name from metrics order by Variable_name limit 5;

-- 按照多个列排序
select Variable_name, Variable_value from metrics order by Variable_name, Variable_value limit 5;
```

排序默认都是升序(ascending)。如果需要降序排列，则需要在指定列名后加上`DESC`关键字。如：

```sql
select Variable_name, Variable_value from metrics order by Variable_name, Variable_value limit 5;
```

> `LIMIT`必须在`ORDER BY`后面使用。

## 6. 过滤数据

### WHERE基本使用

```sql
-- 指定搜索条件
select Variable_name, Variable_value from metrics where Variable_value > 0 order by Variable_name limit 10;
```

> `WHERE`必须在`ORDER BY`之前使用。

### WHERE子句操作符

`WHERE`支持的所有条件操作符：

| 操作符        | 说明               |
| ------------- | ------------------ |
| =             | 等于               |
| <>            | 不等于             |
| !=            | 不等于             |
| <             | 小于               |
| <=            | 小于等于           |
| >             | 大于               |
| >=            | 大于等于           |
| BETWEEN (AND) | 在指定的两个值之间 |

> 在进行比较时，如果值的类型是字符串，则需要使用单引号进行界定。

```sql
select Variable_name, Variable_value from metrics where Variable_value between 10 and 10000 order by Variable_name limit 10;
```

## 7. 过滤数据2

### 组合WHERE子句：AND

```sql
-- 用and连接多个过滤条件
select Variable_name, Variable_value from metrics where Variable_value between 10 and 10000 and Variable_name > 'ae' order by Variable_name limit 10;
```

### OR

```sql
-- 用or实现或条件
select * from metrics where Variable_value < 1000 and Variable_name > 'b' or Variable_value > 1000 limit 5;
```

> AND的优先级高于OR，需要结合使用时，尽可能加上括号。

### IN

`IN`后面跟一个圆括号，圆括号中是所有可能的值的集合。

```sql
select * from metrics where Variable_name in ('bytes_received', 'os_data_writes','os_data_reads');
```

> IN的优点：
>
> 1. 更清楚更直观；
> 2. 计算次序更容易管理；
> 3. IN执行速度比OR更快；
> 4. 也是最大优点，能动态建立WHERE子句。

### NOT

> MySQL支持使用NOT对IN、BETWEEN、和EXISTS子句进行取反，这与多数DBMS允许使用NOT对各种条件取反有很大区别。

```sql
select * from metrics where Variable_name not in ('bytes_received', 'os_data_writes','os_data_reads') limit 3;
```

## 8. 用通配符进行过滤

### LIKE操作符

`LIKE`是模糊查询的关键字。

### 百分号

`%`表示任意字符的任意次数。

> `%`不能匹配`NULL`。

```sql
-- 以指定字符串开头的列
select * from metrics where Variable_name like 'os\\_data%';
```

> 如果需要准确查询百分号和下划线，需要转义。
>
> SQL对大小写非常不敏感，查'OS'和'os'是一样的。

### 下划线

`_`匹配任意字符的1次。

```sql
SELECT * FROM metrics where Variable_name like 'OS\_data_______';
```

## 9. 用正则表达式进行搜索

### 正则表达式

> 和LIKE不同的是，LIKE是精准匹配，正则是模糊匹配。
>
> 使用正则`'OS_data'`时，等价于使用了`'.*OS_data.*'`。

```sql
-- 正则
select * from metrics where Variable_name regexp 'OS_data';
-- 正则中的或
 select * from metrics where Variable_name regexp 'OS_data|cpu';
-- 使用中括号：匹配几个字符之一
select * from metrics where Variable_name regexp 'OS_data|cpu_[su]';
select * from metrics where Variable_name regexp 'OS_data|cpu_[s-u]';
```

> 转义需要用`\\`，如`\\.`表示`.`。
>
> 正则表达式库解释一个，MySQL解释一个。

### 特殊字符

| 元字符 | 说明     |
| ------ | -------- |
| `\\f`  | 换页     |
| `\\n`  | 换行     |
| `\\r`  | 回车     |
| `\\t`  | 制表     |
| `\\v`  | 纵向制表 |



### 字符类

> 系统内置的字符类(character class)。

| 类           | 说明                                                |
| ------------ | --------------------------------------------------- |
| `[:alnum:]`  | 任意字符和数字（同`[a-zA-Z0-9]`）                   |
| `[:alpha:]`  | 任意字符（同`[a-zA-Z]`）                            |
| `[:blank:]`  | 空格和制表（同`[\\t]`）                             |
| `[:cntrl:]`  | ASCII控制字符（ASCII 0到31和127）                   |
| `[:digit:]`  | 任意数字（同`[0-9]`）                               |
| `[:graph:]`  | 同`[:print:]`，但不包括空格                         |
| `[:lower:]`  | 任意小写字母（同`[a-z]`）                           |
| `[:print:]`  | 任意可打印字符                                      |
| `[:punct:]`  | 既不在`[:alnum:]`又不在`[:cntrl:]`中的任意字符      |
| `[:space:]`  | 包括空格在内的任意空白字符（同`[\\f\\n\\r\\t\\v]`） |
| `[:upper:]`  | 任意大写字母（同`[A-Z]`）                           |
| `[:xdigit:]` | 任意十六进制数字（同`[a-fA-F0-9]`）                 |

### 元字符

| 元字符  | 说明                           |
| ------- | ------------------------------ |
| `*`     | 0或多个匹配                    |
| `+`     | 1或多个匹配（同`{1,}`）        |
| `?`     | 0或1个匹配（同`{0,1}`）        |
| `{n}`   | 指定数目的匹配                 |
| `{n,}`  | 不少于指定数目的匹配           |
| `{n,m}` | 指定范围的匹配（m不能超过255） |

### 定位元字符

| 元字符    | 说明     |
| --------- | -------- |
| `^`       | 文本开始 |
| `$`       | 文本结尾 |
| `[[:<:]]` | 词的开始 |
| `[[:>:]]` | 词的结尾 |

> `^`还可以用于否定范围，如`[^0-9]`匹配非数字。

## 10. 创建计算字段

### Concat拼接字段

将多个字段拼接成一个字符串返回。

```sql
-- 返回：name (value) 的格式
select concat(Variable_name, ' (', Variable_value, ')') as info from metrics where Variable_value regexp '[:digit:]{4}' limit 4;
```

AS 用来指定列名。它被称为别名(alias)或导出列(derived column)。

### 执行数学计算

可以对列进行简单计算，包括加、减、乘、除。

```sql
select Variable_value/2 as computed_value from metrics where Variable_value regexp '[:digit:]{4}' limit 4;
```

## 11. 使用数据处理函数

### 函数

SQL支持使用函数。但是，函数的可移植性不强，各个DBMS对函数的实现有较大的差异。所以如果必须使用函数的话，应做好代码注释。

### 常用函数

> 函数名是大小写不敏感的。

| 函数                     | 说明                                 |
| ------------------------ | ------------------------------------ |
| Left(str, len)           | 返回字符串左边的字符（指定长度）     |
| Length(str)              | 返回字符串的长度                     |
| Locate(substr, str)      | 返回子串在字符串当中第一次出现的索引 |
| Lower(str)               | 将字符串转换为小写                   |
| LTrim(str)               | 去掉字符串左边的空格                 |
| Right(str, len)          | 返回字符串右边的字符（指定长度）     |
| RTrim(str)               | 去掉字符串右边的空格                 |
| Soundex(str)             | 返回字符串的Soundex值                |
| SubString(str, idx, len) | 截取字符串的指定部分                 |
| Upper(str)               | 将字符串转换为大写                   |

> Soundex是将字符串按照发音进行转换而不是按照字符。比如对'Knight'和'Night'的转换结果是完全相同的。

```sql
select Variable_name, UPPER(Variable_name) as upper_name from metrics limit 5;
```

### 日期和时间处理函数

| 函数      | 说明 |
| --------- | ---- |
| AddDate() |      |

时间的话，更推荐使用时间戳。

### 数值处理函数

| 函数 | 说明           |
| ---- | -------------- |
| Abs  | 返回数的绝对值 |

很少用，就不写了。

## 12. 汇总数据

### AVG

返回某列的平均值

```sql
-- 所有行的平均值
select avg(Variable_value) as avg_value from metrics;
-- 特定行的平均值
select avg(Variable_value) as avg_value from metrics where Variable_name < 'h';
```

### COUNT

统计行的数量。

> 如果要统计指定条件的行的数量，要使用WHERE过滤语句。

```sql
-- 统计行的数量
select count(*) as count_val from metrics;
-- 过滤掉指定列值为NULL的行的数量
select count(Variable_value) as count_val from metrics;
```

### MAX

返回指定列的最大值

> MAX会忽略NULL。

### MIN

返回指定列的最小值

### SUM

返回指定列的和。SUM可以进行计算。

```sql
select sum(Variable_value * 3) as sum_val from metrics where Variable_name < 'h';
```

### 聚集不同值

默认上述的函数计算时，都是`ALL`行为。

可以通过添加`DISTINCT`关键字，只统计不同的值。

```sql
-- 统计指定列不同的值
select sum(distinct Variable_value) as sum_val from metrics where Variable_name < 'h';
```

## 13. 分组数据

### GROUP BY

按照指定列进行数据统计。

```sql
select count(Variable_value) as count_val from metrics group by Variable_value;
```

> 使用场景：有很多条订单记录，我们要按照address进行统计，此时就需要使用GROUP BY。

注意：

1. GROUP BY 子句可以包含任意数目的列。这使得能对分组进行嵌套， 为数据分组提供更细致的控制。
2. 如果在GROUP BY子句中嵌套了分组，数据将在最后规定的分组上 进行汇总。换句话说，在建立分组时，指定的所有列都一起计算 (所以不能从个别的列取回数据)。
3. GROUP BY子句中列出的每个列都必须是检索列或有效的表达式 (但不能是聚集函数)。如果在SELECT中使用表达式，则必须在 GROUP BY子句中指定相同的表达式。不能使用别名。
4. 除聚集计算语句外，SELECT语句中的每个列都必须在GROUP BY子句中给出。
5. 如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列 中有多行NULL值，它们将分为一组。
6. GROUP BY 必须在 ORDER BY 之前，在 WHERE 之后。

### HAVING

HAVING 可以完成之前学过的 WHERE 语句的所有功能。如：

```sql
-- 下面两条语句是等价的
select Variable_value from metrics where Variable_value > 100000;
select Variable_value from metrics having Variable_value > 100000;
```

区别是：WHERE只能过滤行，但是HAVING可以过滤分组，比如：

```sql
select Variable_value, count(Variable_value) from metrics group by Variable_value having count(Variable_value) > 10;
```

### SELECT 子句顺序

使用顺序：SELECT、FROM、WHERE、GROUP BY、HAVING、ORDER BY、LIMIT

## 14. 使用子查询

### 子查询(Subquery)

比如，有一个`orders`表，存储order_id和customer_id。`orderitems`表存储order信息，`customers`表存储customer信息。

要求：列出所有订购"TNT2"物品的customer信息。

```sql
SELECT order_id FROM orderitems WHERE prod_name = 'TNT2'
-- outputs: 20005, 20007
SELECT customer_id FROM orders WHERE order_id IN (20005, 20007)
-- outputs: 10087, 10096
SELECT * from customers WHERE customer_id IN (10087, 10096)
```

组合书写：

```sql
SELECT * from customers WHERE customer_id IN (SELECT customer_id FROM orders WHERE order_id IN (SELECT order_id FROM orderitems WHERE prod_name = 'TNT2'))
```

### 作为计算字段使用子查询

要求：显示每个客户的订单总数量，同时显示客户名称和客户id。

```sql
SELECT customer_id, customer_name, (SELECT COUNT(*) FROM orders WHERE orders.customer_id = customers.customer_id) AS order_count FROM customers ORDER BY customer_id
```

## 15. 联结表

### 主键和外键

假如：有一个prod_items表，包含所有产品类别，它有name、desc、price和vendor_id。还有vendors表，存储供应商信息。

这里veodors表中的vendor_id是**主键(primary key)**，它在prod_items中称为**外键(foreign key)**。

> 外键为某个表的一个列，它包含另一个表的主键值。

### 维护引用完整性

主键和外键，并不是形式上的，而是需要在创建表时定义的。这样就可以防止外键使用了不存在的主键。

### 创建联结

> 对多个表同时进行操作。

```sql
-- 两个列分别在两个表中。下面两种写法是等价的
SELECT vendor_name, prod_name FROM vendors, products WHERE vendors.id = products.vendor_id
SELECT vendor_name, prod_name FROM vendors INNER JOIN products ON vendors.vendor_id = products.vendor_id
```

#### 笛卡尔积

如果联结没有过滤条件(WHERE)，则会生成笛卡尔积结果。就是结果行数 = 第1个表的检索行数 * 第2个表的检索行数。

过滤条件可以使用 WHERE 或 INNER JOIN - ON。

### 示例

比如，有一个`orders`表，存储order_id和customer_id。`orderitems`表存储order信息，`customers`表存储customer信息。

要求：列出所有订购"TNT2"物品的customer信息。

```sql
SELECT customer_name, customer_contact FROM customers, orderitems, orders WHERE orderitems.order_name = 'TNT2' AND orderitems.order_id = orders.order_id AND orders.customer_id = customers.customer_id
```

## 16. 创建高级联结

### 使用表别名

> 表列名仅供查询时使用。

表可以使用别名，这样可以缩短SQL语句。比如上一节的例子：

```sql
SELECT customer_name, customer_contact FROM customers, orderitems, orders WHERE orderitems.order_name = 'TNT2' AND orderitems.order_id = orders.order_id AND orders.customer_id = customers.customer_id;
```

可以写作：

```sql
SELECT customer_name, customer_contact FROM customers as c, orderitems as oi, orders as o WHERE oi.order_name = 'TNT2' AND oi.order_id = o.order_id AND o.customer_id = c.customer_id;
```

### 自联结

> 联结是联合多个表同时查询。
>
> 自联结就是联合多个自己同时查询。

要求：找出products表中，prod_name为'DTNTR'产品的供应商生产的所有产品

```sql
SELECT p1.prod_id, p1.prod_name FROM products AS p1, products AS p2 WHERE p1.vendor_id = p2.vendor_id AND p2.prod_name = 'DTNTR';
```

### 外部联结

> 它就是一名字。怎么理解呢？
>
> 比如要联合多个表查询所有user的所有订单。但是有的user没有订单，那这个user就不会被显示，因为 WHERE users.uid = orders.uid 语句会过滤掉没有订单的user。

示例：查询所有user的所有订单：

```sql
SELECT customers.customer_id, orders.order_id FROM customers INNER JOIN orders ON customers.customer_id = orders.customer_id;
```

把没有订单的user也显示出来：

```sql
SELECT customers.customer_id, orders.order_id FROM customers LEFT OUTER JOIN orders ON customers.customer_id = orders.customer_id;
```

> 这里注意：OUTER JOIN 前面必须加上 LEFT 或 RIGHT，来指定是哪个表要显示所有列。

### 带有聚集函数的联结

查询所有user的订单数量（0也要显示哦）：

```sql
SELECT customers.customer_id, COUNT(orders.order_id) FROM customers LEFT OUTER JOIN orders ON customers.customer_id = orders.customer_id GROUP BY customers.customer_id;
```

## 17. 组合查询

### UNION

查找价格小于5的所有物品的列表；查询供应商1001和1002生产的所有物品；两者放在一起返回。

单条语句实现：

```sql
SELECT product_id, product_name, price FROM products WHERE price < 5 OR vendor IN (1001, 1002);
```

使用组合查询实现，就是UNION语句：

```sql
SELECT product_id, product_name, price FROM products WHERE price < 5 
UNION
SELECT product_id, product_name, price FROM products WHERE vendor IN (1001, 1002);
```

> 使用UNION需要注意的是：
>
> 1. 有多个SELECT语句；
> 2. 查询的是同样的列

UNION会自动去掉重复的行，即满足两个(或多个)SELECT的行。如果不想去重，使用`UNION ALL`。

注意：对`UNION`语句来说，**只能有一个`ORDER BY`子句放在最后**。

## 18. 全文本搜索

### 全文本搜索

为了进行全文本搜索，必须对要搜索的列进行**索引**。

### 定义索引：FULLTEXT

```sql
CREATE TABLE productnotes (
	note_id	INT	NOT NULL AUTO_INCREMENT,
  prod_id CHAR(10) NOT NULL,
  note_date DATETIME NOT NULL,
  note_text TEXT NULL,
  PRIMARY KEY(note_id),
  FULLTEXT(note_text)
);
```

定义索引后，MySQL自动维护该索引。

> 不要在导入数据时使用FULLTEXT，会很慢；可以在导入完毕后，修改表来定义FULLTEXT。

### 进行全文本搜索

使用 Match 和 Against。

> Match 的参数必须是索引列。
>
> Against 指定搜索文本。

```sql
-- 下面两条语句等价
SELECT note_text FROM productnotes WHERE MATCH(note_text) AGAINST('rabbit');
SELECT note_text FROM productnotes WHERE note_text LIKE '%rabbit%';
```

全文本搜索的优势：

1. 索引比LIKE子句快；
2. 搜索结果有序。

对于有序的搜索结果，可以通过下面的语句查看每行的等级值：

```sql
SELECT note_text, MATCH(note_text) AGAINST('rabbit') AS rank FROM productnotes WHERE MATCH(note_text) AGAINST('rabbit');
```

### 查询扩展

增加搜索结果的范围。比如搜索一个词'anvils'，我们希望搜索到更多相关内容，即便不包括这个词：

```sql
SELECT note_text, MATCH(note_text) AGAINST('rabbit') AS rank FROM productnotes WHERE MATCH(note_text) AGAINST('anvils' WITH QUERY EXPANSION);
```

### 布尔文本搜索

> 布尔文本对于没有索引的列也可以使用，但是会很慢。

使搜索可以提供一些细节：

1. 要匹配的词；
2. 要排斥的词；
3. 排列提示，即指定某些词更重要；
4. 表达式分组；
5. 另外一些内容。

```sql
SELECT note_text FROM productnotes WHERE MATCH(note_text) AGAINST('>rabbit <carrot' IN BOOLEAN MODE);
```

#### 全文本布尔操作符

| 布尔操作符 | 说明                                             |
| ---------- | ------------------------------------------------ |
| +          | 包含，词必须存在                                 |
| -          | 排除，词必须不存在                               |
| >          | 包含，而且增加等级值                             |
| <          | 包含，而且减少等级值                             |
| ()         | 把词组成子表达式，且允许对词进行包含、排除、排列 |
| ~          | 取消一个词的排序值                               |
| *          | 词尾的通配符（这个好像取消了，使用的时候查查）   |
| ""         | 定义一个短语                                     |

## 19. 插入数据

### INSERT

指定表名和新值。

> 如果省略列名，则会按列名顺序指定列值。不推荐这么做。

```sql
INSERT INTO auth_group(
	id,
	name
)
VALUES(
  1001222,
  'test1'
);
```

### 提高整体性能

INSERT操作可能很费时（特别是有很多索引需要更新时）。

如果数据检索是最重要的，可以降低INSERT的优先级：

```sql
INSERT LOW_PRIORITY INTO
```

这个关键字也适用于UPDATE和DELETE。

### 插入多条数据

```sql
INSERT INTO auth_group(
	name
)
VALUES(
	'xixi1'
),
(
	'xixi2'
);
```

### 插入检索出来的数据

就是INSERT-SELECT。将SELECT出的数据INSERT到其它表中。

```sql
INSERT INTO auth_group(
	name
)
SELECT rname FROM myapp_record LIMIT 1;
```

## 20. 更新和删除数据

### UPDATE

更新数据时一定要小心，因为很容易更新所有行。

```sql
UPDATE auth_group
SET name = 'haha'
WHERE id = 1001226;
```

更新多个列：

```sql
UPDATE auth_group
SET name = 'Jack', id = 999
WHERE id = 1001226;
```

### IGNORE

如果更新时发生一个错误，则所有的更新都会被取消。

如果想要忽略错误，则可以写：

```sql
UPDATE IGNORE auth_group ...
```

如果要删除指定列的值，可设置为NULL：

```sql
UPDATE auth_group
SET name = NULL
WHERE id = 999;
```

### DELETE

DELETE用来删除整行。

```sql
DELETE FROM auth_group WHERE id = 999;
```

如果要删除所有行的话，可以使用：

```sql
TRUNCATE TABLE
```

### 使用UPDATE和DELETE的指导原则

1. 必须记得使用WHERE字句；
2. 使用DELETE前，尽量先使用SELECT进行测试。

## 21. 创建和操纵表

### 创建表

```sql
CREATE TABLE customers(
	cust_id int NOT NULL AUTO_INCREMENT,
  cust_name char(50) NOT NULL,
  cust_city char(50) NULL,
  PRIMARY KEY (cust_id)
) ENGINE=InnoDB;
```

### NULL值

每一列要么是NULL，要么是NOT NULL。含义是：可以为空，不可以为空。

### 主键

主键可以使用一个列，也可以使用多个。

```sql
CREATE TABLE orderitems(
	order_num int NOT NULL, -- 订单号
  order_item int NOT NULL, -- 订单物品
  prod_id int NOT NULL, -- 商品id
  quantity int NOT NULL, -- 数量
  item_price int NOT NULL, -- 价格
  PRIMARY KEY (order_num, order_item)
) ENGINE=InnoDB;
```

主键不能使用允许NULL的列。

### AUTO_INCREMENT

自增的列必须被索引；或者通过使它成为主键来被索引。

### 指定默认值

```sql
CREATE TABLE customers(
	cust_id int NOT NULL AUTO_INCREMENT,
  cust_name char(50) NOT NULL,
  cust_gender int NOT NULL DEFAULT 1 -- 1为男性，2为女性
)
```

### 引擎类型

1. InnoDB是一个可靠的事务处理引擎，它不支持全文本搜索；
2. MEMORY在功能等同于MyISAM，但由于数据存储在内存中，速度很快；
3. MyISAM是一个性能极高的引擎，它支持全文本搜索，但不支持事务处理。

> 混用引擎类型有个大缺陷，外键不能跨引擎。

### 更新表的设计

```sql
-- 添加一列
ALTER TABLE vendors
ADD vend_phone char(20);
-- 删除指定列
ALTER TABLE vendors
DROP COLUMN vend_phone;
```

### 定义外键

先创建grade表：

```sql
CREATE TABLE grades(
	id int NOT NULL AUTO_INCREMENT,
  name char(50) NOT NULL,
  PRIMARY KEY (id)
);
```

创建student表：

```sql
CREATE TABLE students(
  id int NOT NULL AUTO_INCREMENT,
  name char(50) NOT NULL,
  grade_id int NOT NULL,
  PRIMARY KEY (id)
);
```

添加外键：

```sql
ALTER TABLE students
ADD CONSTRAINT fk_students_grade
FOREIGN KEY (grade_id)
REFERENCES students (id);
```

> fk_** 这个名字无所谓，随便起，一般情况下命名方式是：`fk_table1_table2`

### 删除表

```sql
DROP TABLE students;
```

### 重命名表

```sql
RENAME TABLE students TO student;
```

## 22. 使用视图

### 视图

> MySQL 5以后才支持。

它是查看数据的一种设施。视图本身不包含数据。

### 规则和限制

1. 命名唯一，不能跟其他视图或表重名；

2. 创建数目没有限制；

3. 为了创建视图，必须有足够的访问权限，这些限制通常由数据库管理人员授予；

4. 视图可以嵌套；

5. ORDER BY可以用在视图中，但如果从该视图检索数据SELECT中也含有ORDER BY，那么该视图中的ORDER BY将被覆盖；

6. 视图不能索引，也不能有关联的触发器或默认值；

7. 视图可以和表一起使用，例如，编写一条联结表和视图的SELECT语句。


### 使用视图

现在我们要实现一个联表查询的功能，查找订购某个产品的用户：

```sql
SELECT cust_name, cust_contact FROM customers, orders, orderitems WHERE customers.cust_id = orders.cust_id AND orderitems.order_num = orders.order_num AND orderitems.prod_id = 'TNT2';
```

使用视图：

```sql
CREATE VIEW productcustomers AS
SELECT cust_name, cust_contact, prod_id FROM customers.cust_id = orders.cust_id AND orderitems.order_num = orders.order_num;
```

它返回订购任意产品的所有客户的列表。现在来查找订购某个产品的用户：

```sql
SELECT cust_name, cust_contact FROM productcustomers WHERE prod_id = 'TNT2';
```

### 视图的其他示例

#### 拼接工作

如：

```sql
SELECT CONCAT(RTRIM(vender_name), ' (', RTRIM(vendor_country), ')') AS vender_title FROM vendors ORDER BY vender_name;
```

可以写作：

```sql
CREATE VIEW vendorlocations AS
SELECT CONCAT(RTRIM(vender_name), ' (', RTRIM(vendor_country), ')') AS vender_title FROM vendors ORDER BY vender_name;

SELECT * from vendorlocations;
```

#### 过滤不想要的数据

过滤掉空的email的用户。

```sql
CREATE VIEW customeremaillist AS
SELECT cust_id, cust_name, cust_email FROM customers WHERE cust_email IS NOT NULL;

SELECT * from customeremaillist;
```

#### 最后

1. 视图可以使用WHERE子句；
2. 视图主要用于查询；最好不要用于更新(INSERT、UPDATE和DELETE)。

