## 数据库操作
~~~shell
# 创建数据库
CREATE DATABASE [数据库名称];

# 查看已经存在的数据库
\l

# 切换使用数据库
\c [数据库名称]

# 删除数据库，IF EXISTS：如果数据库不存在则发出提示信息，而不是错误信息
DROP DATABASE [IF EXISTS] [数据库名称]
~~~

## schema模式
- schema模式相当于对在数据库下多划分了一层，一个数据库下可以有多个模式（数据库创建之后默认就有一个public的模式，默认创建的表就是放在public模式中），一个模式可以包含视图、索引、数据类型、函数和操作符等对象
- 多个模式中可以创建同名的对象，例如多个模式中可以创建相同名称的表格
- schema模式主要是用于第三方应用业务的对象可以放在独立的模式中，这样它们就不会与其他模式下的对象名称发生冲突，例如在多个业务系统中都有user表，使用多个schema模式就可以直接进行区分而不是更上层的多个数据库进行区分
~~~sql
-- 创建模式
create schema [模式名称]

-- 在模式中创建数据表 
create table [模式名称].[表名](
   ID   INT              NOT NULL,
   NAME VARCHAR (20)     NOT NULL,
   AGE  INT              NOT NULL,
   PRIMARY KEY (ID)
);

-- 删除空的模式
DROP SCHEMA [模式名称]

-- 删除模式和里面的对象
DROP SCHEMA [模式名称] CASCADE
~~~


## 表格结构操作
~~~sql
-- 创建数据表，如果需要整数自增可以选择如下类型：
-- smallserial	        2 字节	自增的小范围整数	     1 到 32767
-- serial	            4 字节	自增整数	            1 到 2147483647
-- bigserial	        8 字节	自增的大范围整数	     1 到 9223372036854775807
-- 其他的字段约束条件如下：
-- NOT NULL：指示某列不能存储 NULL 值。
-- UNIQUE：确保某列的值都是唯一的。
-- PRIMARY Key：NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。。
-- FOREIGN Key： FOREIGN KEY 即外键约束，指定列(或一组列)中的值必须匹配另一个表的某一行中出现的值
-- CHECK： 保证列中的值符合指定的条件。
-- EXCLUSION ：排他约束，保证如果将任何两行的指定列或表达式使用指定操作符进行比较，至少其中一个操作符比较将会返回 false 或空值。
CREATE TABLE table_name(
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL UNIQUE,
   EMP_ID         INT      references EMP(ID),  -- EMP表的ID
   SALARY         REAL    CHECK(SALARY > 0),
   EXCLUDE USING gist
   (NAME WITH =,   -- 如果满足 NAME 相同，AGE 不相同则不允许插入，否则允许插入
   AGE WITH <>),   -- 其比较的结果是如果整个表达式返回 true，则不允许插入，否则允许
   .....
   columnN datatype,
   PRIMARY KEY( 一个或多个列 )
);

-- 查看当前数据库中的数据表列表
\d 

-- 查看指定数据表信息
\d [tablename]

-- 删除数据表
DROP TABLE [tablename];

-- 删除表的数据，但不删除表结构
TRUNCATE TABLE  table_name

-- 给表添加列
ALTER TABLE [table_name] ADD [column_name] datatype;

-- 给表删除列
ALTER TABLE [table_name] DROP COLUMN [column_name];

-- 修改表中某列的 DATA TYPE（数据类型）
ALTER TABLE [table_name] ALTER COLUMN [column_name] TYPE datatype;

-- 给表中某列添加 NOT NULL 约束
ALTER TABLE [table_name] ALTER [column_name] datatype NOT NULL;

-- 给表中某列 ADD UNIQUE CONSTRAINT（ 添加 UNIQUE 约束）
ALTER TABLE [table_name] ADD CONSTRAINT MyUniqueConstraint UNIQUE(column1, column2...);
~~~

## 数据操作
~~~sql
-- 新增数据
INSERT INTO [table_name] (column1, column2, column3,...columnN) VALUES (value1, value2, value3,...valueN);

-- 删除数据，注意不加WHERE子句会删除表中所有数据
DELETE FROM [table_name] WHERE [条件筛选];

-- 更新数据
UPDATE [table_name] SET column1 = value1, column2 = value2...., columnN = valueN WHERE [条件筛选];

-- 使用WHERE对结果进行数据筛选
SELECT * FROM [table_name] WHERE [条件筛选];                                                                           

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


-- GROUP BY根据列进行分组
SELECT column1, COUNT(column1) FROM [table_name] GROUP BY column1;

-- WHERE在数据分组前进行过滤行数据，HAVING在数据分组后进行过滤组数据
select column1 from class group by column1, column2 having count(column1)>=2

-- ORDER BY 排序，ASC是升序（默认不写是升序），DESC是降序，对查询结果先按照column1升序再按照column2降序进行排序返回
SELECT * FROM [table_name] WHERE ORDER BY column1 ASC, column2 DESC;


-- UNION 操作符合并两个或多个 SELECT 语句的结果，请注意，UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同
-- 默认直接使用 UNION 会对多个SELECT 语句查询结果合并时去重，如果不需要去重请使用 UNION ALL
SELECT column1 [, column2 ] FROM table1 [, table2 ] [WHERE condition]
UNION
SELECT column1 [, column2 ] FROM table1 [, table2 ][WHERE condition]


-- 两表内连接，只查询order和goods表连接列数据相等的数据，inner join 和 join 都是内连接
select * from `order` inner join goods on `order`.goods_id=goods.id

-- 以下是左连接，右连接类似
select * from `order` left join goods on `order`.goods_id=goods.id
--order和goods表左连接，查询on 表达式左边表格的全部数据
~~~












