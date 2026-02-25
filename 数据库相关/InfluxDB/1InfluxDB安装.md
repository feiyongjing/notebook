# InfluxDB是什么？
关系型数据库虽然承载软件系统基本内容的铺垫，却难以承载大量设备传输过来的数据内容，但时序数据库（Time Series DBMS）解决了这一难题，InfluxDB是最流行的开源时序库，当然国产也有优秀的时序数据库，例如TDengine，时序数据库适合：IoT、监控、日志、指标业务场景
并且InfluxDB 3.x 基于 IOx 引擎重构，原生支持标准 SQL 查询（PostgreSQL 兼容），这是官方最推荐的查询方式，适配度最高、功能最完整

# docker安装InfluxDB
~~~shell
# 手动进入容器生成管理员token，InfluxDB 1.x版本是有用户的，3.x版本是靠token来决定权限的
docker run -it --rm influxdb:3-core bash
influxdb3 create token --admin --offline --output-file /home/influxdb3/admin-token.txt
# 查看token，手动复制到宿主机的文件中
cat /home/influxdb3/admin-token.txt
{
  "token": "apiv3_Bw72Ge4qGOVwfNFdVH6gW2qUVlbhYaTOrIt6rsyHgTfQcuputyS8mMfBoGTYVeCeXJb6N-AftSt4hUSuMnlVag",
  "name": "_admin"
}

# 启动InfluxDB，更多详细配置请执行 docker run --rm influxdb:3-core influxdb3 serve --help-all 查看详细信息
# data、tokens、plugins
docker run -d \
  --name influxdb3-core \
  -p 8181:8181 \
  -v $PWD/influxdb3/data:/var/lib/influxdb3/data \
  -v %PWD%/influxdb3/tokens:/var/lib/influxdb3/tokens \
  -v $PWD/influxdb3/plugins:/var/lib/influxdb3/plugins \
  influxdb:3-core \
  influxdb3 serve \
  --node-id=node0 \
  --object-store=file \
  --data-dir=/var/lib/influxdb3/data \
  --plugin-dir=/var/lib/influxdb3/plugins \
  --admin-token-file=/var/lib/influxdb3/tokens/admin-token.txt  
# node-id 指定节点ID
# object-store 指定对象存储后端为本地文件系统
# data-dir 指定数据存储目录
# plugin-dir 指定插件加载目录
# admin-token-file 指定token所在文件


# Git Bash 执行（加前缀禁用路径转换）
MSYS_NO_PATHCONV=1 docker run -d \
  --name influxdb3-core \
  -p 8181:8181 \
  -v /e/project/test/influxdb3/data:/var/lib/influxdb3/data \
  -v /e/project/test/influxdb3/tokens:/var/lib/influxdb3/tokens \
  -v /e/project/test/influxdb3/plugins:/var/lib/influxdb3/plugins \
  --user root \
  influxdb:3-core influxdb3 serve \
  --node-id=node0 \
  --object-store=file \
  --data-dir=/var/lib/influxdb3/data \
  --plugin-dir=/var/lib/influxdb3/plugins \
  --admin-token-file=/var/lib/influxdb3/tokens/admin-token.txt

# 连接测试
docker exec -it influxdb3-core bash
~~~

### docker compose启动
~~~yml
name: influxdb3
services:
  influxdb3-core:
    container_name: influxdb3-core
    image: influxdb:3-core
    ports:
      - 8181:8181
    command:
      - influxdb3
      - serve
      - --node-id=node0
      - --object-store=file
      - --data-dir=/var/lib/influxdb3/data
      - --plugin-dir=/var/lib/influxdb3/plugins
      - --admin-token-file=/var/lib/influxdb3/tokens/admin-token.txt 
    volumes:
      - type: bind
        source: E:\project\test\influxdb3\data
        target: /var/lib/influxdb3/data
      - type: bind
        source: E:\project\test\influxdb3/plugins
        target: /var/lib/influxdb3/plugins
      - type: bind
        source: E:\project\test\influxdb3/tokens
        target: /var/lib/influxdb3/tokens

~~~


# InfluxDBStudio客户端安装（只支持1.x版本的服务端进行连接）
到 https://github.com/meverett/InfluxDBStudio/releases 下载压缩包解压缩就可以使用了

# 安装InfluxDB 3 Explorer Web UI
~~~shell
docker run --detach \
  --name influxdb3-explorer \
  -p 8888:80 \
  -v $(pwd)/db:/db:rw \
  -v $(pwd)/config:/app-root/config:ro \
  influxdata/influxdb3-ui:1.4.0 \
  --mode=admin
# /db 存储数据库文件和应用数据
# /app-root/config 存储应用配置文件

# 通过 http://localhost:8888 访问Web界面，连接本机docker启动的服务端可以使用 http://host.docker.internal:8181，token使用上述管理员的token
docker run --detach --name influxdb3-explorer -p 8888:80 -v 'E:\project\test\influxdb-explorer-ui/db':/db:rw -v 'E:\project\test\influxdb-explorer-ui/config':/app-root/config:ro influxdata/influxdb3-ui:1.4.0 --mode=admin
~~~
