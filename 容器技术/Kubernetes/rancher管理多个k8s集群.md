### Rancher支持集中化认证、权限控制、监控和管理多个K8s集群

### Rancher提供了一个简单直接的用户界面给开发工程师方便的管理它们的应用程序

### 解决了多机房、跨区域的容器调度，目前主流方案还是部署多套集群，多套环境，对于多套环境的应用程序的管理

 

下载rancher镜像，注意要2.5以上才可以管理多集群

~~~shell
docker pull rancher/rancher:v2.5.8
~~~
 

启动rancher

~~~shell
# 注意提交创建挂载的本地目录存储数据卷
docker run -d --privileged=true -p 80:80 -p 443:443 -v /mnt/d/rancher_data:/var/lib/rancher/ --restart=always --name "容器名" rancher/rancher:v2.5.8

访问启动rancher容器的主机IP就可以设置管理员账号密码了
~~~
 

 