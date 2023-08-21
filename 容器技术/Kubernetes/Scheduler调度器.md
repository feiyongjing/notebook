## Scheduler是K8s的调度器，Scheduler是单独的程序运行，启动后会一直监听API Server，获取PodSpec.NodeName为空的Pod，对每个Pod都会创建一个binding，表面该Pod应该放到哪个节点上。

### Scheduler调度器有很多考虑的问题

- 公平：如何保证每个节点都能被分配资源
- 资源高效利用：集群所有资源最大化被使用
- 效率：调度的性能要好，能够尽快地对大批量的Pod完成调度工作
- 灵活：允许用户根据自己的需求控制调度的逻辑
---

###Scheduler调度过程分为两个步骤，如果其中有一个步骤有错误，就直接返回错误

- 预选：过滤掉不满足条件的节点，将剩下符合条件的节点放入预选列表，如果预选列表为空，则Pod会一直是pending状态，不断重试预选，直到有节点通过预选
- 优选：对预选列表的节点按照优先级排序，然后选择其中优先级最高的节点
---

### 预先的算法，节点必须通过全部的预选算法才算通过预选

- PodFitsResources：节点上剩余的资源是否大于Pod请求的资源
- PodFitsHost：如果Pod指定了NodeName，检查节点名称是否和NodeName匹配
- PodFistsHostPorts：节点上已经使用的port是否和Pod申请的port冲突
- PodSelectorMatches：过滤掉和Pod指定的label不匹配的节点
- NoDiskConflict：已经mount的volume和Pod指定的volume不冲突，除非它们都是只读
---

### 优选是按照一些优先级选项计算出这些选项的优先级，计算出所有的选项组成一系列的键值对，键是优先级选项的名称，值是优先级选项的权重（该项的重要性）
### 优先级选项如下，优选会根据所有的优先级选项权重进行计算，得出最终的结果

- LeastRequestedPriority：通过计算CPU和Memory的使用率来决定权重，使用率越低权重越高。也就是该选项倾向于资源使用比例更低的节点
- BalancedResourceAllocation：节点上CPU和Memory使用率越接近，权重越高。该选项应该与LeastRequestedPriority选项一起使用，不要单独使用
- ImageLocalityPriority：倾向于已经有要使用镜像的节点。节点上镜像数量越大权重越高
---

### 自定义调度器使用（基本是不会去自定义调度器，因为官方的调度器已经是经过大量场景的验证，是可以满足大量场景的验证）

首先根据官方文档去自定义一个调度器。然后在Pod中使用自定义的调度器
~~~yaml
apiVersion: v1
kind: Pod
metadata:
  labels:       # pod的标签
    app: myapp-1.0  
  name: myapp   # pod的名称
spec:
  schedulername: test-scheduler      # 自定义的调度器名称，默认不写是官方的调度器default-scheduler
  containers:                        # mainC容器的创建方式，可以创建多个容器
  - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
    imagePullPolicy: IfNotPresent    # 镜像的下载策略，可以不写使用默认
    name: coredns1                   # 创建出的容器的名字
~~~
---
 

# 官方调度器的灵活体现
- 调度亲和性
- 容忍和污点
- 固定节点调度
---
# 调度亲和性分两类
- 节点亲和性：根据节点亲和策略直接选择节点运行Pod
- Pod亲和性：根据Pod亲和策略选择Pod，在被选择Pod的同一节点运行Pod

# 亲和性有两种策略：软策略和硬策略，不论是哪种策略都必须通过预选，硬策略和软策略可以同时使用

# 节点亲和性
### 硬策略（requiredDuringSchedulingIgnoredDuringExecution）：Pod强制要求在指定节点列表下，否则Pod处于pending状态挂起
~~~yaml
apiVersion: v1
kind: Pod
metadata:
  labels:       # pod的标签
    app: myapp-1.0  
  name: myapp   # pod的名称
spec:
  containers:                        # mainC容器的创建方式，可以创建多个容器
  - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
    imagePullPolicy: IfNotPresent    # 镜像的下载策略，可以不写使用默认
    name: coredns1                   # 创建出的容器的名字
  affinity:
    nodeAffinity:                                        # 节点亲和性
      requiredDuringSchedulingIgnoredDuringExecution:    # 节点亲和性硬策略
        nodeSelectorTerms:                               # 节点选择
          - matchExpressions:                            # 匹配节点
              - key: kubernetes.io/hostname              # 节点label标签的key,kubernetes.io/hostname代表节点名称的key
                operator: In                             # 键值的运算关系，在下面有详细解释
                values:
                  - k8s-node02                           # 节点label标签的value,kubernetes.io/hostname的vaule代表节点的名称
~~~
---

### 软策略（preferredDuringSchedulingIgnoredDuringExecution）：Pod推荐在指定节点列表下，不满足策略就按照默认的调度策略调度
~~~yaml
apiVersion: v1
kind: Pod
metadata:
  labels:       # pod的标签
    app: myapp-1.0  
  name: myapp   # pod的名称
spec:
  containers:                        # mainC容器的创建方式，可以创建多个容器
  - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
    imagePullPolicy: IfNotPresent    # 镜像的下载策略，可以不写使用默认
    name: coredns1                   # 创建出的容器的名字
  affinity:
    nodeAffinity:                                         # 节点亲和性
      preferredDuringSchedulingIgnoredDuringExecution:    # 节点亲和性软策略
        - weight: 1        # 软策略的权重，由于可能有多个软策略，所以有权重比较
          preference:      # 软亲和
            matchExpressions:                            # 匹配节点
              - key: kubernetes.io/hostname              # 节点label标签的key,kubernetes.io/hostname代表节点名称的key
                operator: In                             # 键值的运算关系，在下面有详细解释
                values:
                  - k8s-node02                           # 节点label标签的value,kubernetes.io/hostname的vaule代表节点的名称 

~~~
---
 

# Pod亲和性

- Pod亲和性是根据资源清单中topologyKey属性指定的亲和Pod所在的节点标签的key和value，在整个集群中查找带有相同节点标签key和value的节点
- Pod反亲和性是根据资源清单中topologyKey属性指定的亲和Pod所在的节点标签的key和value，在整个集群中查找不带有相同节点标签key和value的节点

- ## 硬策略（requiredDuringSchedulingIgnoredDuringExecution）：Pod强制要求在亲和指定Pod的节点列表下，否则Pod处于pending状态挂起
- ## 软策略（preferredDuringSchedulingIgnoredDuringExecution）：Pod推荐在亲和指定Pod的节点列表下，不满足策略就按照默认的调度策略调度

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  labels:       # pod的标签
    app: myapp-1.0  
  name: myapp   # pod的名称
spec:
  containers:                        # mainC容器的创建方式，可以创建多个容器
  - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
    imagePullPolicy: IfNotPresent    # 镜像的下载策略，可以不写使用默认
    name: coredns1                   # 创建出的容器的名字
  affinity:
    podAffinity:                                         # Pod亲和性
      requiredDuringSchedulingIgnoredDuringExecution:    # Pod亲和性硬策略
        - labelSelector:                                 # Pod亲和的Pod所在节点选择
            matchExpressions:                            # 匹配亲和的Pod
              - key: app                                 # Pod的label标签的key,app代表Pod标签的key
                operator: In                             # 键值的运算关系，在下面有详细解释
                values:
                  - v1                                   # Pod的app标签对应的value
          topologyKey: kubernetes.io/hostname            # kubernetes.io/hostname代表根据Pod亲和的Pod所在节点的名称选择节点运行Pod
    podAntiAffinity:                                     # Pod反亲和性，即匹配不亲和该Pod的节点上运行Pod
      preferredDuringSchedulingIgnoredDuringExecution:   # Pod亲和性软策略
        - weight: 1                                      # 软策略的权重，由于可能有多个软策略，所以有权重比较
          podAffinityTerm:
            labelSelector:                               # Pod亲和的Pod所在节点选择
              matchExpressions:                          # 匹配节点
                - key: app1                              # Pod的label标签的key,app1代表Pod标签的key
                  operator: In                           # 键值的运算关系，在下面有详细解释
                  values:
                    - v2                                 # Pod的app1标签对应的value
            topologyKey: xxx         # xxx是亲和Pod所在节点标签，节点标签key是xxx并且值与亲和Pod所在节点xxx标签一致的节点是符合要求的

~~~
---
 
# 容忍(Toleration)和污点（Taint）

亲和性是Pod的一种属性（软性或硬性要求），她使Pod被吸引到一类特定的节点，而污点Taint则相反，它使节点能够排除一类特点的Pod

每个节点都可以应用一个或多个污点Taint，有些Pod不接受容忍污点则这些Pod就不能运行在带有污点的节点上，如果将容忍应用在Pod上，则代表这些Pod可以被调度到带有污点的节点上。

污点的组成：key、value、effect。key和value作为污点的标签，其中value可以为空，effect描述污点的作用。

### effect支持的选项

- NoSchedule：表示K8s将不会将Pod调度到具有该污点的Node上
- PreferNoschedule：表示K8s将尽量避免将Pod调度到具有该污点的Node上
- NoExecute：不会将Pod调度到具有该污点的Node上，同时会将Node上已经存在的Pod驱逐出去
---

## 污点（Taint）设置

使用kubectl taint 命令可以给某个Node节点设置污点，Node被设置上污点之后就和Pod之间存在了一种相斥的关系，可以让Node拒绝Pod的调度执行，甚至将Node已经存在的Pod驱逐出去

~~~shell
# 设置污点
kubectl taint node "节点名称" key1=value1:NoSchedule

# 如果key没有value就省略，但是=不可省略
kubectl taint node "节点名称" key1=:NoSchedule

# 查看污点，在节点说明中查找Taint字段
kubectl describe node "节点名称"

#去除污点，在effect后添加-
kubectl taint node "节点名称" key1=value1:NoSchedule-

# 当有多个master或者需要master运行少量的Pod时，执行一下命令
kubectl taint node "节点名称" node-role.kubernetes.io/master=:PreferNoschedule

#注意一下k8s默认污点，它们是外部因素导致的，所以去除这些污点需要处理外部因素
# node.kubernetes.io/not-ready：节点未准备好，相当于节点状态Ready的值为False。
# node.kubernetes.io/unreachable：Node Controller访问不到节点，相当于节点状态Ready的值为Unknown
# node.kubernetes.io/out-of-disk：节点磁盘耗尽
# node.kubernetes.io/memory-pressure：节点存在内存压力
# node.kubernetes.io/disk-pressure：节点存在磁盘压力
# node.kubernetes.io/network-unavailable：节点网络不可达
# node.kubernetes.io/unschedulable：节点不可调度
# node.cloudprovider.kubernetes.io/uninitialized：如果Kubelet启动时指定了一个外部的cloudprovider，它将给当前节点添加一个Taint将其标记为不可用。在cloud-controller-manager的一个controller初始化这个节点后，Kubelet将删除这个Taint

~~~
---
 

## Pod设置容忍

注意Pod虽然设置了容忍，可以在容忍污点的节点上运行但是并不代表一定会在这些节点上运行，具体的Pod运行节点还是要看调度策略。由于DaemonSet会在每个节点都运行一个Pod，所以使用DaemonSet可以很方便的测试到Pod的容忍

~~~yaml
apiVersion: apps/v1
kind: DaemonSet      # 资源类型是DaemonSet控制器
metadata:
  name: frontend                 # 资源的名称，这里是DaemonSet控制器的名称
  labels:                        # 资源的标签
    app: myapp-1.0 
spec: 
  selector:                      # 定义标签匹配Pod对应的模板，只有匹配的Pod模板会创建对应的Pod
    matchLabels:                 # 匹配的标签key和value集合，不与matchExpressions一起使用
      tier: frontend             # 标签元素，是模板全部标签的一部分
  template:                      # 定义Pod模板
    metadata:                    # 定义Pod模板元数据
      labels:                    # 定义Pod模板的全部标签
        tier: frontend           # 定义Pod模板标签元素
        app: spring-k8s          # 定义Pod模板标签元素
    spec:                        # 定义Pod模板的期望值
      containers:                # 定义Pod模板的容器
      - name: php-redis          # 容器名
        image: wangyanglinux/myapp:v1   # 镜像的来源仓库和版本 
      tolerations:               # 设置Pod的容忍
        - key: "key1"            # 节点上污点的key，如果没有则容忍所有的污点
          operator: "Exists"     # 键值的运算关系，在下面有详细解释
          value: "value1"        # 节点上污点的value
          effect: "NoSchedule"   # 污点的类型，如果没有则容忍污点所有的类型
~~~
---
 
# 固定节点调度
固定节点调度有两种方式

方式一：设置Pod的Pod.spec.nodeName属性将Pod直接调度到指定的Node节点上，会跳过Scheduler调度器的调度策略，该匹配规则是强制匹配。如下使用DaemonSet控制器可以直观的看到Pod只会在设置的固定节点上运行一个Pod而其他节点都不会运行Pod

~~~yaml
apiVersion: apps/v1
kind: DaemonSet      # 资源类型是DaemonSet控制器
metadata:
  name: frontend                 # 资源的名称，这里是DaemonSet控制器的名称
  labels:                        # 资源的标签
    app: myapp-1.0 
spec: 
  selector:                      # 定义标签匹配Pod对应的模板，只有匹配的Pod模板会创建对应的Pod
    matchLabels:                 # 匹配的标签key和value集合，不与matchExpressions一起使用
      tier: frontend             # 标签元素，是模板全部标签的一部分
  template:                      # 定义Pod模板
    metadata:                    # 定义Pod模板元数据
      labels:                    # 定义Pod模板的全部标签
        tier: frontend           # 定义Pod模板标签元素
        app: spring-k8s          # 定义Pod模板标签元素
    spec:                        # 定义Pod模板的期望值
      nodeName: node-01          # 设置Pod只能在指定名称的节点上运行
      containers:                # 定义Pod模板的容器
      - name: php-redis          # 容器名
        image: wangyanglinux/myapp:v1   # 镜像的来源仓库和版本 
~~~
---

方式二：设置Pod的Pod.spec.nodeSelector，根据K8s的label-selector机制选择节点，由调度器调度策略匹配label，而后调度Pod到目标节点，该匹配规则属于强制约束

~~~yaml
apiVersion: apps/v1
kind: DaemonSet      # 资源类型是DaemonSet控制器
metadata:
  name: frontend                 # 资源的名称，这里是DaemonSet控制器的名称
  labels:                        # 资源的标签
    app: myapp-1.0 
spec: 
  selector:                      # 定义标签匹配Pod对应的模板，只有匹配的Pod模板会创建对应的Pod
    matchLabels:                 # 匹配的标签key和value集合，不与matchExpressions一起使用
      tier: frontend             # 标签元素，是模板全部标签的一部分
  template:                      # 定义Pod模板
    metadata:                    # 定义Pod模板元数据
      labels:                    # 定义Pod模板的全部标签
        tier: frontend           # 定义Pod模板标签元素
        app: spring-k8s          # 定义Pod模板标签元素
    spec:                        # 定义Pod模板的期望值
      nodeSelector:              # 只在节点上存在指定标签的节点运行Pod
        kubernetes.io/hostname: node-01  # 节点的标签，kubernetes.io/hostname对应的是节点名称，所以node-01就是节点名称
      containers:                # 定义Pod模板的容器
      - name: php-redis          # 容器名
        image: wangyanglinux/myapp:v1   # 镜像的来源仓库和版本 
~~~
---
