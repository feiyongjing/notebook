# docker安装MongoDB
~~~shell
# 启动MongoDB，设置端口、数据文件挂载、用户名密码设置
docker run -itd --name mongodb -p 27017:27017 -v /docker_volume/mongodb/data:/data/db -e MONGO_INITDB_ROOT_USERNAME=mongo -e MONGO_INITDB_ROOT_PASSWORD=123456 mongo:6.0.18


docker run -itd --name mongodb -p 27017:27017 -v 'E:\project\test\mongo-tset-data':/data/db -e MONGO_INITDB_ROOT_USERNAME=mongo -e MONGO_INITDB_ROOT_PASSWORD=123456 mongo:6.0.18

# 进入容器使用命令行测试或者是使用其他连接方式可以尝试去连接MongoDB，--authenticationDatabase admin 是MongoDB 创建用户时默认会关联到某个数据库（通常是 admin 库，超级用户必须创建在 admin 库），连接时必须指定用户关联的库
docker exec -it mongodb bash
mongosh --host=127.0.0.1 --port=27017 -u mongo -p 123456 --authenticationDatabase admin
# 或者不进入容器直接连接
docker exec -it mongodb mongosh --host=127.0.0.1 --port=27017 -u mongo -p 123456 --authenticationDatabase admin

# 连接上MongoDB后查看所有数据库，默认是有admin、config、local三个默认数据库的
show dbs
~~~

## MongoDB中常见的数据类型  
~~~
Srting      字符串  
Number      数字  
Boolean     布尔  
Arroy       数组  
Date        日期  
Buffer      Buffer对象  
Mixed       任意对象   nodejs使用mongoose.Schema.Types.Mixed表示  
ObjectId    对象ID，一般用于存储其他文档的id，相当于mysql表的外键，nodejs使用mongoose.Schema.Types.ObjectId  
Decimal128  高精度数据 nodejs使用mongoose.Schema.Types.Decimal128  
~~~