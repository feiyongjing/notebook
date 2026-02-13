# Docker Compose 概述与安装？
- 使用 Docker 的时候，定义 Dockerfile 文件，然后使用 docker build、docker run 等命令操作容器。然而微服务架构的应用系统一般包含若干个微服务，每个微服务一般都会部署多个实例，如果每个微服务都要手动启停，那么效率之低，维护量之大可想而知
- 使用 Docker Compose 可以轻松、高效的管理容器，它是一个用于定义和运行多容器 Docker 的应用程序工具


### 安装参考
- https://docs.docker.com/compose/install/ 
-  https://blog.csdn.net/pushiqiang/article/details/78682323

### docker-compose使用
~~~shell
# 下载可执行程序
sudo curl -L https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

# 添加执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 验证安装
docker-compose --version

# 根据资源清单创建并启动容器
docker-compose [-f docekr-compose.yml] up [参数]
# -f 参数指定资源清单路径，不写会默认会在当前目录下查找 docker-compose.yml 的资源清单启动容器
# -d 参数设置启动的容器在后台运行

# 查看docker-compose启动的容器
docekr-compose ps

# 查看docker-compose资源清单中所使用的镜像
docekr-compose images

# 根据资源清单停止资源清单中的servicers容器
docker-compose [-f docekr-compose.yml] stop 

# 根据资源清单启动资源清单中的servicers容器
docker-compose [-f docekr-compose.yml] start

# 根据资源清单删除资源清单中的启动servicers容器和创建的network
docker-compose [-f docekr-compose.yml] down

# 在指定启动的service的容器中执行命令
docker-compose exec [service的名称] [命令]
# 可以用于进入容器，docker-compose exec [service的名称] bash

~~~
---


### docekr compose资源清单，参考：
### 使用参考
- https://www.runoob.com/docker/docker-compose.html   
- https://blog.csdn.net/pushiqiang/article/details/78682323
- https://zhuanlan.zhihu.com/p/387840381  
~~~yml
version: "3"                # 资源清单的版本号
services:                   # 定义有那些service的容器需要启动，下一节点的属性是service的名称
  web:                      # 定义service的名称
    build: .                # 定义镜像创建清单的 DockerFile 路径，如果同时指定了 build 和 image 那么会使用build提供的 DockerFile 创建镜像，image指定创建出来的新镜像镜像名称和版本
    image: myApp:1.0.0      
    networks:               # 定义使用那些docker网络
      - my-bridge
    ports:                  # 定义service容器启动的端口号映射
     - "8080:8080"
  redis:                    # 定义service的名称
    image: redis:alpine     # 直接定义镜像和版本，默认是docker hub 仓库拉取
    volumes:
      - redis_data:/data/db          # 方式一：使用设置好的卷进行挂载
      - /opt/data:/var/lib/mysql:ro  # 方式二：设置宿主机挂载数据卷到容器中，并且最后的ro表示文件只读权限，默认是rw读写权限
    ports:                           
      - "6379"              
    networks:               # 定义使用那些docker网络
      - my-bridge
    restart: always         # 故障自动重启，保障可用性
    deploy:
      replicas: 2           # 定义启动的容器数量
      update_config:        # 定义更新配置
        parallelism: 2      # 一次更新的容器数量
        delay: 10s          # 更新一组容器之间的等待时间
      restart_policy:       # 配置是否以及如何在退出时重新启动容器
        condition: on-failure  # none（不重启）, on-failure（启动失败重启）、any（总是重启），默认any
networks:                     # 设置自定义docker网络
  my-bridge:                  # 自定义的docker网络名称
    drivace: bridge           # 设置该docker的网络类型
volumes:                      # 设置的数据挂载卷，但是该数据卷一般是直接设置在service中
  test_data:                  # 这是匿名卷，由docker引擎自动指定数据卷的宿主机目录，一般不使用该方式
  redis_data: /opt/data/redis # 设置卷供其他容器进行挂载使用   
~~~

 