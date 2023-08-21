## 索引分类

- 普通索引（Key Indexes）
- 唯一索引（Unique Indexes）
- 主键索引（Primary Key Indexes）
- 组合索引（Composite Index）
- 前缀索引（ Prefix Indexes）
- 全文索引（Full-Text Indexes）
- 哈希索引（Hash Indexes）
- 空间索引（Spatial Indexes）
---

# 索引查看
~~~sql
SHOW INDEX FROM <表名> [ FROM <数据库名>]
~~~

语法说明如下：
- <表名>：要显示索引的表  
- <数据库名>：要显示的表所在的数据库  

该语句会返回一张结果表，该表有如下几个字段，每个字段所显示的内容说明如下：  

- Table：表的名称  
- Non_unique：用于显示该索引是否是唯一索引。若不是唯一索引，则该列的值显示为 1；若是唯一索引，则该列的值显示为 0  
- Key_name：索引的名称  
- Seq_in_index：索引中的列序列号，从 1 开始计数  
- Column_name：列名称  
- Collation：显示列以何种顺序存储在索引中。在 MySQL 中，升序显示值“A”（升序），若显示为 NULL，则表示无分类  
- Cardinality：显示索引中唯一值数目的估计值。基数根据被存储为整数的统计数据计数，所以即使对于小型表，该值也没有必要是精- 确的。基数越大，当进行联合时，MySQL 使用该索引的机会就越大  
- Sub_part：若列只是被部分编入索引，则为被编入索引的字符的数目。若整列被编入索引，则为 NULL  
- Packed：指示关键字如何被压缩。若没有被压缩，则为 NULL  
- Null：用于显示索引列中是否包含 NULL。若列含有 NULL，则显示为 YES。若没有，则该列显示为 NO  
- Index_type：显示索引使用的类型和方法（BTREE、FULLTEXT、HASH、RTREE）  
- Comment：显示备注信息。

---

# 1. 普通索引
普通索引是最基本的索引，没什么限制，根据创建时机不同，主要有两种方式：

## 1. 在创建表时同时创建普通索引
~~~sql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] table_name
(
    col_name column_definition,
    INDEX|KEY [index_name] [index_type] (col_name [(length)] [ASC | DESC])
);
 
 
index_type:
    USING {BTREE | HASH};
~~~   
其中方括号[]中内容表示可选项，竖线|表示两者选一。

- col_name：列名；
- column_definition：列的具体定义，这里省略了；
- INDEX|KEY：表示使用INDEX和KEY都能创建普通索引，因为在MySQL中，INDEX和KEY是一样的。
- [index_name]：索引的名字，是可选项，如果没有写，默认使用字段名作为索引的名称，一般以idx_字段名作为前缀来命名；
- [index_type]：索引类型，表示索引的数据结构，有两类：BTREE和HASH，如果没有指明，默认是BTREE；
- col_name [(length)] [ASC | DESC]：col_name是要添加索引的列，length表示要在类型为字符串的列的前length个字符构成的字符串上添加索引，[ASC|DESC]表示升序还是降序方式存储索引，默认是升序方式存储；

例：
~~~sql
CREATE TABLE my_table
(
    id INT(11) not NULL auto_increment,
    name VARCHAR(20) NOT NULL,
    address VARCHAR(20) NOT NULL,
    PRIMARY KEY(id),
    INDEX idx_name (name)
);
~~~

该语句在创建my_table表时，同时在字段name上创建了一个名为idx_name的的普通索引。

## 2.创建表之后创建索引

1. 修改表结构的方式添加索引
~~~sql
ALTER TABLE table_name ADD INDEX index_name (col_name [(length)] [ASC | DESC]);
-- 例如：
ALTER TABLE my_table ADD INDEX idx_address  (address);
-- 该语句在my_table表的address字段上添加了名为idx_address的普通索引。
~~~

2. 直接创建索引
~~~sql
CREATE INDEX index_name ON table(column[(length)] [ASC|DESC]);
-- 例如：
CREATE INDEX idx_name ON my_table(name);
~~~

可以使用如下DDL语句删除索引：
~~~sql
DROP INDEX index_name ON my_table
~~~     

# 2.唯一索引
与普通索引类似，不同的是要求索引列的值必须唯一，但是多个NULL值。与普通索引创建方式相似：

创建表时创建：
~~~sql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] table_name
(
    col_name column_definition,
    UNIQUE [INDEX|KEY] [index_name] [index_type] (col_name [(length)] [ASC |DESC])
);
        
index_type:
    USING {BTREE | HASH}
~~~
创建表后直接创建：
~~~sql
CREATE UNIQUE INDEX index_name ON table(column[(length)] [ASC|DESC]);
~~~
修改表结构方式创建：
~~~sql
ALTER TABLE table_name ADD UNIQUE indexName (column(length));
~~~
 

# 3.主键索引
主键索引不需要主动取创建，表创建主键后，自动在主键上创建，主键索引名称总PRIMARY，主键值的唯一和非空也就保证了主键索引的列值必须唯一且不能为空，所以索引可以看成是特殊的唯一索引。
~~~sql
CREATE TABLE my_table
(
    id INT(11) not NULL auto_increment,
    name VARCHAR(20) NOT NULL,
    address VARCHAR(20) NOT NULL,
    PRIMARY KEY(id)
);
~~~
该语句在创建表时创建了主键id，同时会在id上创建主键索引

# 4.组合索引
指在多个字段上创建的索引，在多个字段上创建的普通索引和唯一索引都是组合索引，组合实际是字段的组合，对于多个字段上的唯一索引，要求组合字段必须唯一。可以如建组合索引
~~~sql
CREATE TABLE my_table
(
    id INT(11) not NULL auto_increment,
    name VARCHAR(20) NOT NULL,
    address VARCHAR(20) NOT NULL,
    PRIMARY KEY(id),
    INDEX idx_name_address (name,address)
);
~~~
该语句在字段name和address两个字段上创建普通索引  
如果要在两个字段上创建唯一索引，只要将INDEX替换成UNIQUE即可  
也可以在创建表后创建组合索引 
~~~sql
    ALTER TABLE table_name ADD INDEX idx_name_address (name,address); 
    CREATE INDEX idx_name_address ON my_table(name,address)
~~~  

# 5.前缀索引
暂时未了解

# 6.全文索引
暂时未了解

# 7.哈希索引
哈希索引是基于哈希表实现的，只有精确匹配索引中所有列的查询，哈希所有才有效