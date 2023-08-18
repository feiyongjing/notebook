## 数据库操作
~~~shell
# 查看所有数据库
# show dbs没有显示数据库，是因为数据库是空的，没有表和数据
# 默认是有admin、config、local三个默认数据库的
show dbs;

# 切换使用数据库，如果数据库不存在则会创建该数据库
use [数据库名称];

# 显示当前数据库
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
~~~shell
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

# 指定集合下删除文档数据
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
# 第二参数是update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的更新的数据
# 第三参数是一些更新策略选项
# upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
# multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
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
# projection ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。
# 以易读的方式来读取数据，可以使用 pretty() 方法
db.[集合名称].find(query, projection).pretty()

# 以下是一些查询条件与Mysql的查询一一对应
db.[集合名称].find({"by":"菜鸟教程"}).pretty()	    where by = '菜鸟教程'
db.[集合名称].find({"likes":{$lt:50}}).pretty()	    where likes < 50
db.[集合名称].find({"likes":{$lte:50}}).pretty()	where likes <= 50
db.[集合名称].find({"likes":{$gt:50}}).pretty()	    where likes > 50
db.[集合名称].find({"likes":{$gte:50}}).pretty()	where likes >= 50
db.[集合名称].find({"likes":{$ne:50}}).pretty()	    where likes != 50
db.[集合名称].find({"likes":{$ne:50}}).pretty()	    where likes != 50
db.[集合名称].find({ $and: [{ price: {$gt:10} }, { price: {$lt:20} }] })   where price > 10 and price < 20
db.[集合名称].find({ $or: [{ author: '余华' }, { author: '曹雪芹' }] })    where author = '余华' or author = '曹雪芹'

~~~