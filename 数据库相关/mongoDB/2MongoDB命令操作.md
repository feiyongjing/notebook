## 数据库操作
~~~shell
# 查看所有数据库，默认是有admin、config、local三个默认数据库的
show dbs;
# admin数据库：从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
# config数据库：当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息
# local数据库：这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合


# 创建或切换使用数据库(数据库不存在则会创建该数据库，存在就进行切换，空数据库未插入数据会在会话结束之后被清除)
# 注意空数据库不会被持久化显示，必须插入至少一条数据后才会出现在 show dbs 结果中
use [数据库名称];

# 显示当前使用的数据库
db

# 删除当前数据库
db.dropDatabase()
~~~

## 集合操作（集合相当于Mysql的表）
~~~shell
# 查看当前数据库下的集合列表，集合相当于mysql的表
show collections

# 创建集合，集合相当于mysql的表
# 参数user是集合名称
db.createCollection("user")

# 删除当前数据库下的指定集合，集合相当于mysql的表
db.[集合名称].drop()

# 修改当前数据库下的指定集合名称，集合相当于mysql的表
# 参数xxxx是新的集合名称
db.[集合名称].renameCollection("xxxx")
~~~

## 文档操作（文档相当于Mysql数据表中的一行数据）
- 如果操作文档时所属的集合不存在也会一并创建集合
~~~s
# 向指定集合中插入文档数据
db.[集合名称].insert({"name":"张三", "age": "18"})

# 向指定集合中插入一条文档数据
# 第二个参数是一些策略选项
# writeConcern：写入策略，默认为 1，即要求确认写操作，0 是不要求。
db.[集合名称].insertOne({"name":"张三", "age": "18"},{ writeConcern: <document>})

# 向指定集合中插入多条文档数据
# 第二个参数是一些策略选项
# writeConcern：写入策略，默认为 1，即要求确认写操作，0 是不要求。
# ordered：指定是否按顺序写入，默认 true，按顺序写入。
db.[集合名称].insertMany([{},{}],{writeConcern: <document>,ordered: <boolean>})

# 指定集合下删除文档数据，请注意 db.[集合名称].remove({})会删除集合下的全部文档
# 第一参数（可选）是删除的文档的条件。
# 第二参数是一些更新策略选项
# justOne : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
# writeConcern :（可选）抛出异常的级别。
db.[集合名称].remove(
   {"name":"张三"},
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)

# 指定集合下修改文档数据
# 第一参数是update的查询条件，类似sql update查询内where后面的条件。
# 第二参数是update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的更新的数据，注意一定要{$set:{'age': '20'}}，如果直接是{'age': '20'}会覆盖更新，就是其他属性全部删除只有age这个属性了。递增操作可以使用{$inc: { age: 1 }}，每次执行age加1
# 第三参数是一些更新策略选项
# upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入插入数据，true为插入，默认是false，不插入。
# multi : 可选，默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
# writeConcern :可选，抛出异常的级别。
db.[集合名称].update(
   {"name": "张三"},
   {$set:{'age': '20'}},
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)


# 查看指定集合中的文档数据
# query ：可选，使用查询操作符指定查询条件
# projection ：可选，使用投影操作符指定返回的键。默认省略查询时返回文档中所有键值。1指定字段返回，0表示指定字段不返回，例如 {name: 1, age: 0}表示返回name字段，age字段不返回
# 以易读的方式来读取数据，可以使用 pretty() 方法
db.[集合名称].find(query, projection).pretty()

# 以下是一些查询条件与Mysql的查询一一对应
db.[集合名称].find({"by":"菜鸟教程"}).pretty()	    where by = '菜鸟教程'
db.[集合名称].find({"by":"菜鸟教程", "author":"余华"}).pretty()	    where by = '菜鸟教程'
db.[集合名称].find({"likes":{$lt:50}}).pretty()	    where likes < 50
db.[集合名称].find({"likes":{$lte:50}}).pretty()	  where likes <= 50
db.[集合名称].find({"likes":{$gt:50}}).pretty()	    where likes > 50
db.[集合名称].find({"likes":{$gte:50}}).pretty()	  where likes >= 50
db.[集合名称].find({"likes":{$ne:50}}).pretty()	    where likes != 50
db.[集合名称].find({ $and: [{ price: {$gt:10} }, { price: {$lt:20} }] })  where price > 10 and price < 20
db.[集合名称].find({ $or: [{ author: '余华' }, { author: '曹雪芹' }] })    where author = '余华' or author = '曹雪芹'
db.[集合名称].find({author: {$in:['余华', '曹雪芹']}})                     where author in ('余华','曹雪芹')
db.[集合名称].find({author: {$nin:['余华', '曹雪芹']}})                    where author not in ('余华','曹雪芹')

# 模糊查询，支持js的正则表达式
db.[集合名称].find({"by":"/你的正则表达式/"})
# 例如 {"by":"/哈哈/"} 查询by字段包含“哈哈”的所有文档
# 例如 {"by":"/^嗷嗷/"} 查询by字段以“嗷嗷”开头的所有文档


# 查询集合中满足条件（条件参考上面的find查询条件）的文档条数
db.[集合名称].count(query)

# 分页查询，skip是指前面的跳过多少条数据，limit是指需要查询的数据条数，如下是查询第9~12条数据
db.[集合名称].find().skip(8).limit(4)

# 排序，sort设置查询的数据按照属性进行排序，1表示升序，-1表示降序，
# 多个属性的排序是按照sort中声明排序属性的先后顺序进行排序的，例如{age: 1, user_id: -1}是先按照age升序然后age一致的按照user_id降序
db.[集合名称].find().sort({{age: 1, user_id: -1}})
~~~

## 索引操作
~~~s
# 查看集合的所有索引，默认有一个索引key是_id的索引 
db.[集合名称].getIndexes();


# 创建索引，keys设置根据那些字段创建索引（设置索引是1升序索引还是-1降序索引），options设置索引（一般就unique设置唯一索引，name设置索引名称，一般索引名称也不用设置，会根据字段等信息自动生成）
db.[集合名称].createIndex(keys, options);
db.[集合名称].createIndex({ age: 1 });                        # 创建单字段升序索引
db.[集合名称].createIndex({ name: 1, age: -1 });              # 创建多字段复合索引
db.[集合名称].createIndex({ email: 1 }, { unique: true });    # 创建唯一索引

# 创建部分索引，如下是age字段大约25的部分文档进行创建索引
db.[集合名称].createIndex(
{ age: 1 },
{ partialFilterExpression: { age: { $gt: 25 } } }
);

# 创建TTL索引（120秒后过期）
db.[集合名称].createIndex({ createtime: 1 }, { expireAfterSeconds: 120 });


# 删除指定索引
db.[集合名称].dropIndex("indexName");
# 默认索引 _id 之外的所有索引都删除
db.[集合名称].dropIndexes();
~~~

## 查询性能分析
~~~s
# explain 用于查询解释和性能分析的方法，可以在find()、aggregate() 和 count() 等查询操作的结果上调用 explain() 方法
db.[集合名称].find(query, projection).explain()
# explain有三种模式，一般默认就够使用了
#     queryPlanner (默认) ： 只列出所有可能执行的方案，不会执行实际的语句，显示已经胜出的方案winningPlan（最佳查询计划）。
#     executionStats ： 只执行winningPlan方案，并输出结果。
#     allPlansExecution ：执行所有的方案，并输出结果

# 返回结果：
# queryPlanner：查询优化器的决策和统计信息。
# winningPlan：优化器选择的最佳执行计划。
# executionStats：执行计划的统计信息，如扫描的文档数量、查询时间等。
# serverInfo：MongoDB 服务器的信息

# 主要就是看queryPlanner.winningPlan.stage参数值
#   COLLSCAN：全表扫描
#   IXSCAN：索引扫描
#   FETCH：根据索引去检索指定document
#   SHARD_MERGE：将各个分片返回数据进行merge
#   SORT：表明在内存中进行了排序
#   LIMIT：使用limit限制返回数
#   SKIP：使用skip进行跳过
#   IDHACK：针对_id进行查询
#   SHARDING_FILTER：通过mongos对分片数据进行查询 ---服务器是分片的才有。
#   COUNT：利用db.coll.explain().count()之类进行count运算
#   COUNTSCAN： count不使用Index进行count时的stage返回
#   COUNT_SCAN： count使用了Index进行count时的stage返回
#   SUBPLA：未使用到索引的$or查询的stage返回
#   TEXT：使用全文索引进行查询时候的stage返回
#   PROJECTION：限定返回字段时候stage的返回
~~~



