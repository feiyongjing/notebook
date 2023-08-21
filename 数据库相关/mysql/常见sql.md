# 数据库操作sql
~~~sql
SHOW DATABASES;                 -- 查看数据库，默认的存在的四个数据库   information_schema ，mysql， performance_schema ， sys 
CREATE DATABASE [数据库名字];    -- 创建数据库   
DROP DATABASE 数据库名字;        -- 删除数据库
USE 数据库的名字;                -- 选择数据库
~~~
---
# 常见数据类型

- 整数类型                    int和bigint(相对于long)
- 字符串类型                  varchar(最长长度)和text文本类型
- 时间类型                    timestamp

---
# 表格创建sql
~~~sql
-- 创建数据表，IF NOT EXISTS 是当指定名字的表不存在时创建，例子：CREATE TABLE table_name IF NOT EXISTS (column_name column_typ); 
CREATE TABLE table_name (column_name int) ENGINE=InnoDB;   

--删除数据表
DROP TABLE TABLE-NAME;                                                                           
~~~
---
                         
# 数据的增删改查
~~~sql
--插入数据
INSERT INTO table_name ( name, age ) VALUES ( name1, age1), (name2, age2); 
--查询出数据并插入         
INSERT INTO table_name1 ( name, age ) SELECT name1, age1 from table_name2;

--删除数据，注意不加WHERE子句会删除表中所有数据
DELETE FROM table_name WHERE [条件筛选];

--更新数据
UPDATE table_name SET field1=1,field2=2 WHERE 条件筛选;     

--更新数据，IGNORE表示出现更新出错也继续更新后面的数据
UPDATE IGNORE table_name SET field1=1,field2=2 WHERE 条件筛选;

-- 使用WHERE对结果进行数据筛选
SELECT * FROM `USER` WHERE [条件筛选];                                                                           

-- select子句的执行顺序：select、from、where、group by、having、order by、limit

-- where子句过滤
-- 常见操作符：=、>、<、<=、>=与Java一致，表示不相等有两种方式：!= 或 <>

-- between表示范围过滤，例子：where id between 5 and 10; 表示过滤id在5到10之间的数据，包含id等于5和10的数据

-- is null和 is not null 过滤null值和不为null值，例子：where id is null; 表示过滤id是null的数据

-- and组合多个过滤条件，例子：where id=1 and name='张三'; 过滤id等于1并且name是张三的数据
-- or选择过滤，例子：where id=1 or name='张三'; 过滤id等于1 或者 name是张三的数据，满足一项就行了
-- and与or语句有歧义时使用括号

-- in 过滤满足多个数据项，例子：where id in (1,2,3,4,5); 过滤id等于1、2、3、4、5的数据

-- not 否定它之后所跟的任何条件，例子：where id is not null; 表示过滤id不是null的数据

-- like通配符%表示任何字符出现任意次数，例子：where name like '%abc%';  过滤name带有abc的数据
-- like通配符_表示匹配单个的任意字符，例子：where name like '_abc';  过滤name是4个字符长度并且以abc结尾的数据

-- regexp 正则表达式，例子：where name regexp binary 'abc';  regexp之后的字符串是正则表达式，binary表示区分大小写，去除则不区分



-- table_1表只有sex列表示男或女
-- 例子：将男变成女，女变成男
-- 方式一
update table_1 set sex﻿= case sex when '男' then '女' else '男' end;​
-- 方式二
update table_1 set = if(sex='男','女','男');


-- 排行榜
-- Scores表根据分数排名（分数相同的人排名不同）
select Score, row_number() over(order by Score desc) 'Rank' from Scores;
 
-- Scores表根据分数排名（分数相同的人排名相同，且排名连续）
select Score, dense_rank() over(order by Score desc) 'Rank' from Scores;
 
-- Scores表根据分数排名（分数相同的人排名相同，且排名不连续）
select Score, rank() over(order by Score desc) 'Rank' from Scores;
~~~
---
 
# 字段和函数操作
## 聚合函数
~~~sql
-- avg() 函数，例子：select avg(age) from user; 返回user表中age列的平均值，可以使用多个age()函数计算多个列的平均值，age为null的行不包含在内计算

-- count() 函数，例子：select count(age) from user; 返回age列非null值的行数。使用count(*)计算全表行数，不管是否有null值

-- max() 函数，例子：select max(age) from user; 返回age列中最大的值，如果是文本列则返回对应列的最后一个值

-- min() 函数，例子：select min(age) from user; 返回age列中最小的值，如果是文本列则返回对应列的第一个值

-- sum() 函数，例子：select sum(age) from user; 返回age列中非null值的数据总和

-- distinck 可用于上面5个聚合函数，用于代表计算列中值不同聚会函数，例如：select avg(age) from user; 计算age列中值不同的列表平均值
~~~ 
---

## 数字类型函数操作
~~~sql
-- abs() 函数，例子：select abs(age) from user; 返回age列的绝对值

-- mod() 函数，例子：select mod(age，10) from user; 返回age对10取余数，可用于小数 

-- rand() 函数，例子：select rand() from user; 返回随机数

-- pi() 函数，例子：select pi() from user; 返回圆周率
~~~
---

## 文本类型函数操作
~~~sql
-- 拼接字段操作使用 concat() 函数，例子：select concat('姓名：', name, '年龄：', age) from user 查询姓名和年龄然后拼接成一列返回

-- 去除字段空格操作使用 ltrim()、rtrim()、trim() 函数，例子：select ltrim(name) from user 去除name前的空格返回，rtrim() 和 trim() 分别是去除后空格和两端空格

-- 返回列支持运算符：+、-、*、/

-- upper() 函数，例子：select upper(name) from user ; name列转换成大写返回

-- lower() 函数，例子：select lower(name) from user ; name列转换成大写返回

-- left() 函数，例子：select left(name, 2) from user;  返回name列最前面的2个字符

-- right() 函数，例子：select right(name, 2) from user;  返回name列最后面的2个字符

-- length() 函数，例子：select length(name) from user; 返回name列字段的长度

-- locate() 函数，例子：select locatch(name, '张三'，n) from user; 返回张三在name列第一次出现在第几个字符，默认不加n从头开始找，加n从第n个字符开始找

-- substring() 函数，例子：select substring(name, m, n) from user; 返回name列中从头开始第m个字符之后(包含第m个字符)一共n个字符，m是负数时从尾部开始数但是n的性质不变。n不写则默认是末尾，另一种写法是  select substring(name from m for n) from user;  m和n的性质不变

-- soundex() 函数，例子：select * from user where soundex(name)=soundex('张四');  如果张三与张四的soundex值相等则会将name是张三的数据查询出来，即返回name与张四的soundex值相等的数据行
~~~
---

# 日期与时间函数操作
~~~sql
-- adddate() 函数，例子：select adddate(created_at, INTERVAL 10 day) from user;  返回的结果是created_at列表示的时间增加10天

-- addtime() 函数，例子：select addtime(created_at, '1 0:0:0') from user﻿​;  返回的结果是created_at列表示的时间增加一天

-- date_add() 函数，例子：select date_add(created_at, interrval 10 day) from user﻿​; 与 adddate() 函数一样，可指定其他时间单位

-- datediff() 函数，例子：select datediff(updated_at, created_at) from user﻿​; 返回修改与创建时间相差的天数

-- date_format() 函数，例子：select Date_Format(now(), '%Y-%m-%d %T:%f') from `USER`﻿​; 指定返回的日期时间格式 

-- curdate() 函数，例子：select curdate(); 返回的结果是当前的日期，不包含时间

-- curtime() 函数，例子：select curtime(); 返回的结果是当前的时间，不包含日期

-- now() 函数，例子：select now(); 返回的结果是当前的日期和时间

-- date() 函数，例子：select date(created_at) from user;  返回一个日期时间对应的日期部分

-- time() 函数，例子：select time(created_at) from user;  返回一个日期时间对应的时间部分

-- dayofweek() 函数，例子：select DayOfWeek('1998-02-03'); 返回一个日期时间对应的星期几部分，星期天是一个星期的第一天

-- year() 函数，例子：select year(created_at) from user; 返回一个日期时间对应的年份

-- month() 函数，例子：select month(created_at) from user; 返回一个日期时间对应的月份

-- day() 函数，例子：select day(created_at) from user; 返回一个日期时间对应的天数

-- hour() 函数，例子：select hour(created_at) from user; 返回一个日期时间对应的小时

-- minute() 函数，例子：select minute(created_at) from user; 返回一个日期时间对应的分钟数

-- second() 函数，例子：select second(created_at) from user; 返回一个日期时间对应的秒数
~~~
---

## 其他函数
~~~sql
-- last_insert_id() 函数，例子：select last_insert_id(id) from user; 获取user表中插入的最后一条数据的id
~~~
---
  

# 子查询
~~~sql
-- 先根据name在user表查找user的id，再根据user_id查找对应的订单
select * from `order` where user_id in (select user.id from user name='张三' )
~~~
---

# UNION组合查询
~~~sql
select name, age from user where name= '张三' union select name, age from user where age=20; 

-- UNION规则

-- UNION必须由两条或两条以上的SELECT语句组成，语句之间用关键字UNION分隔（因此，如果组合4条SELECT语句，将要使用3个UNION关键字）。
-- UNION中的每个查询必须包含相同的列、表达式或聚集函数（不过各个列不需要以相同的次序列出）。

-- 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含地转换的类型（例如，不同的数值类型或不同的日期类型）。

-- 在用UNION组合查询时，只能使用一条ORDER BY子句，它必须出现在最后一条SELECT语句之后

-- UNION从查询结果集中自动去除了重复的行，即名字叫张三，年龄20的数据只显示一条数据，要显示重复数据使用 union all

~~~
---
 
# LIMIT查询分页
~~~sql
SELECT * FROM `USER` [ LIMIT (N|M) OFFSET(P) ]

-- LIMIT关键字   后面接一个参数N表示返回N条数据，接两个数据N和M返回第N条数据之后的数据M条

-- OFFSET关键字  从第N+1条数据开始返回，类似于LINIT两个数据中的N  
~~~
---                                                                        

 
# ORDER BY 排序
~~~sql
SELECT * FROM `USER` WHERE ORDER BY userId ASC, userWeight DESC;

-- ORDER BY 排序，ASC是升序（默认不写是升序），DESC是降序，对查询结果先按照userId升序再按照userWeight降序进行排序返回
~~~
---  
 
# GROUP BY  分组 和 HAVING分组后过滤数据
~~~sql
SELECT userName, COUNT(userName) FROM `USER` GROUP BY userName;

-- GROUP BY     根据列进行分组
-- 函数：COUNT对指定列计数 ，SUM求和，AVG平均数，MIN最小值，MAX最大值，FIRST第一条数据，LAST最后一条数据

select package_name, group_concat(class_name), count(class_name) from class group by package_name, version having count(package_name)>=2

--查询结果分组，查询class数据表中除了package_name分组列之外的class_name列的数据和条数，先按照package_name分组，在按照version分组,然后获取package_name分组数据数大于等于2条的组

-- WHERE在数据分组前进行过滤行数据，HAVING在数据分组后进行过滤组数据
~~~
---  

# 表连接
~~~sql
-- 两表内连接，只查询order和goods表连接列数据相等的数据，inner join 和 join 都是内连接
select * from `order` inner join goods on `order`.goods_id=goods.id

-- 以下是左连接，右连接类似
select * from `order` left join goods on `order`.goods_id=goods.id
--order和goods表左连接，查询on 表达式左边表格的全部数据
~~~
---  
 

# 索引
~~~sql
-- 普通索引创建
CREATE INDEX indexName ON table_name (column_name);                              
-- 普通索引创建
ALTER TABLE table_name ADD INDEX indexName(columnName)​;                     
-- 普通索引创建
CREATE TABLE table_name ( INDEX [indexName] (username(length)) );            

-- 唯一索引的创建， 唯一索引列的值必须唯一，但允许有空值
CREATE UNIQUE INDEX indexName ON table_name (username(length));        
-- 唯一索引的创建， 唯一索引列的值必须唯一，但允许有空值
ALTER TABLE table_name ADD UNIQUE [indexName] (username(length);​  
-- 唯一索引的创建， 唯一索引列的值必须唯一，但允许有空值
CREATE TABLE mytable( UNIQUE [indexName] (username(length)) ); 

-- CHAR，VARCHAR类型，length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length

-- 删除索引
DROP INDEX [indexName] ON mytable;                                                                 
~~~
--- 
 

# 修改表结构
~~~sql
-- RENAME子句 修改表名
ALTER TABLE TABLE-NAME RENAME TO alter_tbl;                                                                

-- DROP 子句删除列（表只有一列时不能删除）
ALTER TABLE TABLE-NAME DROP column_name;                                                            

-- ADD 子句增加列，默认添加到表末尾
ALTER TABLE TABLE-NAME ADD column_name column_type;                                           

-- ADD 子句增加列，FIRST关键字设置为表的第一列
ALTER TABLE TABLE-NAME ADD column_name column_type FIRST;                                

-- ADD 子句增加列，AFTER关键字设置为表中某原字段之后
ALTER TABLE TABLE-NAME ADD column2_name column_type AFTER column_name;  

-- MODIFY 子句只修改原字段的数据类型
ALTER TABLE TABLE-NAME MODIFY  column_name CHAR(10);                                       

-- 修改字段类型，同时添加条件不为空与默认值
ALTER TABLE TABLE-NAME MODIFY  column_name BIGINT NOT NULL DEFAULT 100;​  

-- 修改字段默认值 
ALTER TABLE TABLE ALTER i SET DEFAULT 1000;                                                                  

-- 删除字段默认值
ALTER TABLE TABLE-NAME ALTER i DROP DEFAULT;                                                            

-- CHANGE子句同时修改原字段的名字和类型（i是原字段名，j是新字段名）
ALTER TABLE TABLE-NAME CHANGE i j BIGINT;                                                                     
~~~
--- 
                           