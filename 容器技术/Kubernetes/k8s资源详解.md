### k8s中所有的内容都抽象为资源，资源实例化之后叫对象。

### k8s中存在那些资源？

- 名称空间级别的资源：在不同的命名空间下是互相隔离的
  1. 工作负载型资源：Pod、ReplicaSet、Deployment等
  2. 服务发现及负载均衡型资源：Service(svc)、Ingress等
  3. 配置与存储型资源：Volumc、CSI等
  4. 特殊类型的存储卷：ConfigMap、Secre等
- 集群级别的资源：针对整个集群级别的资源
  1. Namespace、Node、ClusterRole、ClusterRoleBinding
- 元数据型的资源：参考型的资源
  1. HPA、PodTemplate、LimitRange

在Kubernets中，Pod作为基本的执行单元，它可以拥有多个容器和存储数据卷，能够方便在每个容器中打包一个单一的应用，从而解耦了应用构建时和部署时的所关心的事项，已经能够方便在物理机/虚拟机之间进行迁移。

 

# Pod的类型
- 自主式的Pod（不被控制器管理的Pod）：Pod一但死亡不会被重启拉起来，即使无法满足满足指定副本数的期望值也不会拉起来
- 控制器管理的Pod：由控制器控制Pod的生命周期

# Pod的控制器类型
1. ReplicationController（RC）、ReplicaSet（RS）、Deployment
   - ReplicationController简称RC，用来确保容器应用的副本数始终保持在用户定义的副本数，即如果有容器异常退出、会自动创建新的Pod来替代；而如果异常多出来的容器也会自动回收。在新版本的K8s中建议使用ReplicaSet来取代ReplicationController，原因是RC不支持Pod的标签筛选，RS支持
   - ReplicaSet简称RS，和ReplicationController没有本质的不同，只是名字不一样，并且ReplicaSet支持集合式的标签筛选
   - 虽然ReplicaSet可以独立使用，但一般还是建议使用Deployment来自动管理ReplicaSet，比如ReplicaSet不支持rolling-update（滚动更新） 但Deployment支持
   - Deployment的滚动更新原理是创建一个新的ReplicaSet，然后由新的ReplicaSet创建新版本的Pod，再杀掉老ReplicaSet创建的老版本Pod，交替执行直到所有的Pod都是最新的版本，但是不会关闭老的ReplicaSet，原- 因是可能出现新的Pod有bug，需要回滚，然后无需创建老的ReplicaSet就能进行滚动回滚
2. Horizontal Pod Autoscaling  （HPA）
   - Horizontal Pod Autoscaling  简称为HPA，仅适用于Deployment和ReplicaSet，在V1版本中仅支持根据Pod的CPU利用率扩容和缩容，在vlalpha版本还能支持根据内存和用户自定义的metric进行扩容和缩容
3. StatefulSet
   - StatefulSet是为了解决有状态服务的问题（对应Deployments 和 ReplicaSets 是为无状态服务而设计），其应用场景包括：
     - 稳定的持久化存储，即Pod重新调度后还是能访问到相同的持久化数据，基于PVC来实现
     - 稳定的网络标志，即Pod重新调度后其PodName和HostName不变，基于Headless Service (即没有Cluster IP的 Service)来实现
     - 有序部署，有序扩展，即Pod是有顺序的，在部署或扩展的时候依据定义的顺序依次进行（即在下一个Pod运行之前所有之前的Pod必须是 Running 或 Ready状态），基于 init containers 来实现
     - 有序收缩、有序删除
4. DaemonSet
   - DaemonSet确保全部（或者一些，即排除被打了污点的Node）Node上每一个Node仅运行一个Pod的副本。当有Node加入集群时，也会为它们新增一个Pod。当有Node从集群移除时，这些Pod也会被回收。删除DaemonSet将会删除它创建的所有Pod，创建n个DaemonSet则在（所有或者一些，即排除被打了污点的Node）Node上运行n个Pod副本   
   - 使用DaemonSet的一些典型用法：
     - 运行集群存储daemon，例如在每个Node上运行 glusterd、ceph。
     - 在每个Node上运行日志收集daemon，例如fluentd、logstash。
     - 在每个Node上运行监控daemon，例如Promethus Node Exporter
   - 注意master节点不会运行Pod副本，原因是master是有污点不能被调度
5. Job、Gron Job
   - Job负责批处理任务，即仅执行一次的任务，它保证批处理任务的一个或多个Pod成功结束
   - 删除CronJob是不会删除CronJob创建的Job
   - Gron Job管理基于时间的Job，即：
     - 在给定的时间只运行一次
     - 周期性的在给定时间点运行
 

# svc详解（service）
Kubernetes 'Service' 定义了一种抽象：一个Pod的逻辑分组，一种可以访问他们的策略既微服务，这一组 ’Pod‘ 能被 ’Service‘ 访问到，通常是通过 ’Label Selector‘  匹配
如果一个Pod的标签集合是一个svc的标记集合的父集，则可以通过svc来访问这个Pod，一个svc可以匹配访问多个Pod

## svc的类型
- ClusterIP：默认类型，自动分配一个仅ClusterIP内部可以访问的虚拟IP，使用场景是用于管理集群服务，只提供一个虚拟IP地址加端口可以在内部访问服务
  - 可以通过编辑etcd中的资源清单升级为NodePort类型
- NodePort: 在ClusterIP的基础上为Service在每个Node节点上绑定一个端口，可以通过Node节点IP加绑定的端口在外部访问该服务 
  - NodePort对外提供服务的端口是配置清单中的  spec.ports[0].nodePort  属性，注意可能是数组的其他元素，该端口必定是30000之上的端口
  - 通过设置ClusterIP内部的端口映射到物理网卡的端口，在外部可以通过物理网卡的端口访问服务
  - 不建议通过编辑etcd中的资源清单降级为ClusterIP类型，如果非要进行降级建议删除svc资源重新通过资源清单创建NodePort类型的svc
- LoadBalancer：在NodePort的基础上，借助cloud provider创建一个外部负载均衡器，并将请求转发到NodeIP:NodePort
  - 在私有云上需要自己搭建LVS
  - 在公有云（阿里云、百度云、腾讯云、AWS等）上由于云主机中内核的IPVS模块被移除了，只能购买它们的LAAS服务
  - LoadBalancer对外提供服务的端口是配置清单中的  spec.ports[0].nodePort  属性，注意可能是数组的其他元素，该端口必定是30000之上的端口
- ExternalName：把集群外部的服务引入到集群内部来，在集群内部直接使用。没有任何类型代理被创建，这只有kubernetes 1.7或更高版本的kube-dns 才支持
 
# Ingress详解

Ingress 也是为了解决在集群之外，访问集群内部Service服务的问题

实际上，将service的type设置为nodePort或LoadBalancer，也能实现将集群内部的服务暴露给外部访问。但是它们都有对应的缺点，

nodePort和LoadBalancer实现将集群内部的服务暴露给外部访问的缺点

- 如果只是单独的直接通过Node节点IP加绑定的物理网卡端口在外部访问服务，则只能单点访问，无法分布式部署
- 如果需要实现分布式部署则需要考虑使用额外的负载均衡，目前有两种负载均衡方案，
  - 第一种是将NodePort类型的svc替换为LoadBalancer类型的svc，LoadBalancer类型的svc的缺点是每一个LoadBalancer类型的svc都会在k8s集群外部自动创建一个LoadBalancer的负载均衡器，实际上是成本比较高，并且只能在云平台使用
  - 第二种是在k8s集群内部创建一个负载均衡器的Pod，并且创建一个与该Pod匹配的NodePort类型的svc暴露Node节点IP加绑定的物理网卡端口访问负载均衡服务，而负载均衡服务负责进行服务转发和代理到不同的Pod集群的svc上，缺点是当添加或者减少Pod集群服务需要修改负载均衡转发代理规则然后手动重启负载均衡服务

Ingress，是K8s的一个资源对象，定义了一系列路由转发规则（或反向代理规则），它规定了外部进来的HTTP/HTTPS请求应该被转发到哪个Service上。

简单的说，Ingress就是一段nginx服务的反向代理配置，它能根据请求中不同的Host和URL路径，将请求转发到不同的Service的Pod上。

Ingress与手动部署反向代理服务器的思路是一样的。如果反向代理服务器使用的是Nginx程序，Ingress Controller 会将 Ingress 规则变化生成一段Nginx的配置，然后将这个配置写到Ingress Controller管理下的Nginx Pod中，然后自动重启加载路由规则。

架构图：https://img2020.cnblogs.com/blog/1395193/202009/1395193-20200923212820372-429939181.png

必须具有 ingress 控制器【例如 ingress-nginx，即ingress-nginx的镜像】才能满足 Ingress 的要求。仅创建 Ingress 资源无效。

参考：https://cloud.tencent.com/developer/article/1718482  获取ingress-nginx的镜像和创建Ingress资源
