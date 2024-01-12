## 帮助命令
~~~shell
# 查看docker版本
docker version      

# 显示docker的系统信息，包括镜像和容器数量
docker info             

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
# 启动容器，-it和sh表示进入容器
docker run [-it] 容器启动参数 镜像的名字和版本 [sh]
# --name 为容器指定⼀个「名字」
# --restart=always 遇到错误⾃动重启
# -d 和 -id 分别指后台运行和交互式运行
# -p [本地端⼝:容器端⼝] 容器与物理机的端口映射
# -v [本地⽂件:容器⽂件] 容器与物理机的数据卷挂载
# -e NAME=VALUE 设置容器内部的环境变量

# 查看Docker当前运行的容器
docker ps [args]                                                                          
# 添加 -a 参数 查看Docker当前所有的容器（包含未运行的容器）                                                                
# 添加 -q 参数 显示的结果只有容器的id                                                                   

# 根据id或者是名字删除指定容器，可删除多个，不能删除运行中的容器，删除运行中的容器要加-f
docker rm -f 容器id或容器名字                                       

# 强制删除所有的容器
docker rm -f $( docker ps -aq)                                        

# 进入容器里面
docker exec -it 容器id bash   

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

~~~

## 日志
~~~shell
# 查看指定容器的日志，-ft 是显示日志，--tail number是显示的日志条数
docker logs -ft --tail number 容器id                              

# 查看容器进程信息
docker top 容器id                                                            
~~~

# 其他
~~~shell
# 获取容器/镜像的元数据，包含镜像的启动命令，启动命令注意查看Cmd和Entrypoint指令
docker inspect [镜像或容器ID]
~~~