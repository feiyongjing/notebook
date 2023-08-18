### 注意MongoDB启动后必须设置用户密码否则无法使用数据库连接工具进行连接

~~~shell
# 只允许拉取最新版本，拉取其他版本会出现奇怪的问题
docker pull mongo:latest

# /data/db是容器内的默认目录
# –auth：需要密码才能访问容器服务，但是第一次是不需要密码直接登录设置管理员账号密码
# 容器暴露27017端口服务
docker run -itd --name mongo -v /docker_volume/mongodb/data:/data/db -p 27017:27017 mongo:4.4 --auth

# 进入容器设置密码并进入到默认的admin数据库
docker exec -it mongo mongo admin

# user:‘root’ 设置用户名为root
# pwd:‘123456’ 设置密码为123456
# role:‘userAdminAnyDatabase’ 只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
# db: ‘admin’ 可操作的数据库
# ‘readWriteAnyDatabase’ 赋予用户读写权限
db.createUser({ user:'root',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},'readWriteAnyDatabase']});

# 参考 https://developer.aliyun.com/article/999689
# MONGO_INITDB_ROOT_USERNAME ： mongodb 的用户名，这设置为mongo
# MONGO_INITDB_ROOT_PASSWORD： mongodb 的密码， 这里设置为123456
docker run -d -p 27017:27017 --name mongodb -v /docker_volume/mongodb/data:/data/db -e MONGO_INITDB_ROOT_USERNAME=mongo -e MONGO_INITDB_ROOT_PASSWORD=123456 mongo

# 查看所有数据库
# show dbs没有显示数据库，是因为数据库是空的，没有表和数据
# 默认是有admin、config、local三个默认数据库的
show dbs
~~~