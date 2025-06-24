## 帮助命令
~~~shell
# 查看docker版本
docker version      

# 显示docker的系统信息，包括镜像和容器数量，以及docker镜像和容器的存储目录
docker info             
# docker info | grep "Root" 查看镜像和容器的存储目录
# docker 守护进程的默认配置文件是 /etc/docker/daemon.json 可能不存在需要手动创建配置自定义的设置

# 指定命令的帮助文档
docker [命令] --help  
~~~

## 镜像命令
~~~shell
# 查看主机上所有的镜像
docker images [args]
# 添加 -q 参数 显示的结果只有镜像的id                           

# 搜索指定的名字
docker search [镜像的名字]         

# 下载指定名字的镜像，默认是下载最新版本的
docker pull [镜像的名字]      

# 代码包构建成镜像
docker build -t [打包出新的镜像名字] -f [DokcerFile文件路径] [代码包目录（一般是jar包或前端dist包目录）]   

# 推送镜像到远程镜像仓库
docker push [镜像的名字]

# 删除指定id或名称的镜像，可多个删除
docker rmi -f [镜像id1] [镜像id2]                   

# 删除所有的镜像
docker rmi -f $(docker images -qa)  

# 创建一个新的镜像，镜像指向旧的镜像（同一个镜像id），注意使用docker rm [镜像id]会将新镜像和旧镜像都删除
docker tag  [旧的镜像id或名称] [新的镜像名称]  

# 镜像导出成tar文件
docker save -o [文件名称]  [镜像名称或id]    

# 导入tar文件成镜像
docker load -i [文件名称]                                 
~~~
 

## 容器命令
~~~shell
# 启动容器，-it表示进入交互式命令行
docker run [-it] [容器启动参数] [镜像的名字和版本] [命令]
# --name 为容器指定⼀个「名字」
# --restart=always 遇到错误⾃动重启，已经运行的容器也可以通过 docker container update --restart=always {容器id} 修改为总是重启
# --link [另一个容器的名称] 会自动添加另一个容器的DNS记录到启动的容器中，即可以直接在启动的容器中通过另一个容器的名称和端口去访问该容器的服务，注意这个连接是单向的，另一个容器不具备该能力
# --network [网络名称] 在指定的网络下运行容器（注意一个容器可以指定拥有多个网络环境），如果不设置默认是在 docker0 的网桥下运行容器。非默认docker0网桥网络下的容器之间是可以直接通过容器的名称和端口号进行通信访问服务（甚至可以访问没有开启端口映射到宿主机的容器服务），不需要手动添加 --link 参数
# -d 和 -id 分别指后台运行和交互式运行
# -p [本地端⼝:容器端⼝] 容器与物理机的端口映射
# -v [本地⽂件:容器⽂件] 容器与物理机的数据卷挂载
# -e NAME=VALUE 设置容器内部的环境变量
# 最后的命令会替换DockerFile指定的容器启动命令

# 查看Docker当前运行的容器
docker ps [args]                                                                          
# 添加 -a 参数 查看Docker当前所有的容器（包含未运行的容器）                                                                
# 添加 -q 参数 显示的结果只有容器的id                                                                   

# 根据id或者是名字删除指定容器，可删除多个，不能删除运行中的容器，删除运行中的容器要加-f
docker rm -f 容器id或容器名字                                       

# 强制删除所有的容器
docker rm -f $( docker ps -aq)                                        

# 进入交互式命令行执行命令
docker exec -it 容器id [命令]   
# 例如 docker exec -it 容器id bash 会进入容器的终端

# 在容器内部执行会退出容器
exit 

# 启动容器
docker start 容器id                                                          

# 重启容器
docker restart 容器id                                                       

# 停止当前正在运行的容器
docker stop 容器id                                                           

# 强制停止当前容器
docker kill 容器id                                                              

# 查看容器进程信息
docker top 容器id 
docker status 容器id 

# 查看指定容器的日志，-ft 是显示日志，--tail number是显示的日志条数，并且持续查看
docker logs -ft --tail number 容器id                              

# 获取容器/镜像的元数据，包含镜像的启动命令、容器的网络配置、容器的环境变量等，启动命令注意查看Cmd和Entrypoint指令
docker inspect [镜像或容器ID] 
# 检查镜像适用的架构 docker inspect [镜像或容器ID] | grep Architecture
# 注意GraphDriver.Data下的属性
#   LowerDir：这些是覆盖文件系统的只读层。对于容器来说这些宿主机目录下的每一个目录都对应一个镜像层的内部目录，这些目录组成了容器的根目录，容器中的这些目录是只读的
#   UpperDir：这是覆盖文件系统的读写层。对于容器来说这些宿主机目录下的目录是该容器在启动之后所做的更改
#   WorkDir：这是覆盖所需的目录，它需要一个内部使用的空目录
#   MergedDir：docker会通过Union File System（联合文件系统）将这些LowerDir下的目录和UpperDir下的目录联合挂载成完整的容器根目录
# 
~~~


# 网络
~~~shell
# 查看docker的网络列表，有三种类型的网络：bridge、host、none
docker network ls

# 创建docker网络，非默认docker0网桥网络下的容器之间是可以直接通过容器的名称和端口号进行通信访问服务，不需要手动添加 --link 参数
docker network create [参数] [自定义的网络名称]
# -d 参数设置管理网络的驱动程序，默认不设置是 bridge （网桥模式）

# 使指定容器连接到指定的docker网络中，注意一个容器可以连接多个docker网络
docker network connect [网络名称] [容器名称或id]

# 使指定容器断开与指定的ocker网络的连接
docker network [参数] disconnect [网络名称] [容器名称或id]
# -f 参数表示强制断开连接

# 删除所有未使用的网络
docker network prune 

# 删除一个或者多个网络
docker network rm [多个空格隔开的网络名称或ID]

# 查看网络详细信息
docker network inspect [docker的networkID]
~~~