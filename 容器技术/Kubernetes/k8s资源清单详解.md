## 在k8s中，一般使用yaml格式的文件来创建符合我们预期期望的pod，这样的yaml文件我们一般称为资源清单

### 资源清单格式
~~~yaml
apiVersion: group/apiversion          # 组和apiversion组成，该属性由kind资源类型的值来决定，通过 "kubectl explain 资源类型" 获取group和apiversion 如果没有给定group 名称，那么默认为core核心版本，
kind: pod               # 资源类型，注意要写资源类型的全称，不要写缩写
metadata:             # 资源元数据
  name:
  namespace:
  lables：
  annotations:    # 主要的目的是方便用户阅读查找
spec:   # 期望的状态

status:  # 当前状态，本字段由Kubernetes 自身维护，用户不能定义
~~~
 

# Pod简化版的资源清单，需要其他的再添加
~~~yaml
apiVersion: v1
kind: Pod
metadata:
  labels:       # pod的标签
    app: myapp-1.0  
  name: myapp   # pod的名称
  namespace: kube-system # pod所在的命名空间
spec:
  initContainers:                    # initC容器的创建方式，可以创建多个容器
  - image: xxx                       # 创建容器使用的镜像
    name: coredns1                   # 创建出的容器的名字
    commend: ["/bin/sh","-c","echo Hello"]    # initC容器的退出命令
  containers:                        # mainC容器的创建方式，可以创建多个容器
  - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
    imagePullPolicy: IfNotPresent    # 镜像的下载策略，可以不写使用默认
    name: coredns1                   # 创建出的容器的名字
    readinessProde:                  # 容器的就绪探针
      httpGet:                       # 就绪探针探测方式选择httt的get请求
        prod: 80                     # 就绪探针访问的端口
        path: /index.html            # 就绪探针访问的路径
      initialDelaySeconds: 1         # 容器初始化多久后开始进行就绪探测，单位是秒
      periodSeconds: 3               # 就绪探测失败后重试的间隔时间，单位是秒
    livenessProde:                   # 容器的存活探针
      httpGet:                       # 存活探针探测方式选择httt的get请求
        prod: 80                     # 存活探针访问的端口
        path: /index.html            # 存活探针访问的路径
      initialDelaySeconds: 1         # 容器初始化多久后开始进行存活探测，单位是秒
      periodSeconds: 3               # 探测存活后重试的间隔时间，单位是秒
    lifecycle:                       # 定义容器的生命周期
      postStart:                     # 定义容器启动时的钩子
        exec:                        # 定义钩子的类型是命令
          commend:  ["/bin/sh","-c","echo hello world"]    # 定义钩子执行的命令      
      postStop:                      # 定义容器关闭前的钩子
        exec:                        # 定义钩子的类型是命令
          commend:  ["/bin/sh","-c","echo goodbye"]        # 定义钩子执行的命令 
  - image: xxx:1.3.1                 # 创建容器使用的镜像
    name: coredns2                   # 创建出的容器的名字 
~~~
 
# RC控制器(简化版)
~~~yaml
apiVersion: apps/v1
kind: ReplicationController      # 资源类型是RC控制器
metadata:
  name: frontend                 # 资源的名称，这里是RC控制器的名称
  labels:                        # 资源的标签
    app: myapp-1.0  
spec: 
  replicas: 3                    # 资源的副本数量
  selector:                      # 定义标签匹配Pod对应的模板，只有匹配的Pod模板会创建对应的Pod
    app: nginx                   # 标签元素，是模板全部标签的一部分
  template:                      # 定义Pod模板
    metadata:                    # 定义Pod模板元数据
      labels:                    # 定义Pod模板的全部标签
        app: nginx               # 定义Pod模板标签元素
    spec:                        # 定义Pod模板的期望值
      containers:                # 定义Pod模板的容器
      - name: php-redis          # 容器名
        image: wangyanglinux/myapp:v1   # 镜像的来源仓库和版本
        env:                     # 容器的环境变量
        - name: GET_HOST_FROM    # 环境变量名称
          value: dns             # 环境变量的值
          name: zhangsan         # 环境变量的名称
          value: "123"           # 环境变量的值
        ports:                   # 容器暴露的端口集合
        - containerPort: 80      # 容器暴露的端口
~~~
 

# RS控制器（简化版）
~~~yaml
apiVersion: apps/v1
kind: ReplicaSet      # 资源类型是RS控制器
metadata:
  name: frontend                 # 资源的名称，这里是RS控制器的名称
  labels:                        # 资源的标签
    app: myapp-1.0 
spec: 
  replicas: 3                    # 资源的副本数量
  selector:                      # 定义标签匹配Pod对应的模板，只有匹配的Pod模板会创建对应的Pod
    matchLabels:                 # 匹配的标签key和value集合，不与matchExpressions一起使用
      tier: frontend             # 标签元素，是模板全部标签的一部分
#    matchExpressions:           # 其他的标签匹配方式，不与matchLabels一起使用
#      - key: app                # 指定匹配标签的key
#        operator: Exists        # 指定标签的匹配方式，有In、NotIn、Exists、DoesNotExists。具体见下面解释
  template:                      # 定义Pod模板
    metadata:                    # 定义Pod模板元数据
      labels:                    # 定义Pod模板的全部标签
        tier: frontend           # 定义Pod模板标签元素
        app: spring-k8s          # 定义Pod模板标签元素
    spec:                        # 定义Pod模板的期望值
      containers:                # 定义Pod模板的容器
      - name: php-redis          # 容器名
        image: wangyanglinux/myapp:v1   # 镜像的来源仓库和版本
        env:                     # 容器的环境变量
        - name: GET_HOST_FROM    # 环境变量名称
          value: dns             # 环境变量的值
          name: zhangsan         # 环境变量的名称
          value: "123"           # 环境变量的值
        ports:                   # 容器暴露的端口集合
        - containerPort: 80      # 容器暴露的端口
~~~
 
### spec.selector.matchExpressions[0] .operator 的值有In、NotIn、Exists、DoesNotExists四个选择，In表示标签的值在某个列表中、NotIn表示标签的值不在某个列表中、Exists表示某个标签存在、DoesNotExists表示某个标签不存在
~~~yaml
matchExpressions:            # 其他的标签匹配方式
  - key: app                 # 指定匹配标签的key
    operator: In             # 指定标签的匹配方式
    values:                  # 标签值匹配列表，既标签匹配key为上述key也就是app，值是spring-k8或者是xxx的Pod模板
      - spring-k8s           # 标签匹配值
      - xxx                  # 标签匹配值
~~~
 

# deployment控制器（简化版）
~~~yaml
apiVersion: apps/v1
kind: Deployment      # 资源类型是deployment控制器
metadata:
  name: frontend                 # 资源的名称，这里是deployment控制器的名称
  labels:                        # 资源的标签
    app: myapp-1.0 
spec: 
  replicas: 3                    # 资源的副本数量
  revisionHistoryLimit: 0        # etcd最多保存的RS历史版本数量，设置为0则禁止版本回滚，默认不写是10
  template:                      # 定义Pod模板
    metadata:                    # 定义Pod模板元数据
      labels:                    # 定义Pod模板的全部标签
        tier: frontend           # 定义Pod模板标签元素
        app: spring-k8s          # 定义Pod模板标签元素
    spec:                        # 定义Pod模板的期望值
      containers:                # 定义Pod模板的容器
      - name: php-redis          # 容器名
        image: wangyanglinux/myapp:v1   # 镜像的来源仓库和版本
        env:                     # 容器的环境变量
        - name: GET_HOST_FROM    # 环境变量名称
          value: dns             # 环境变量的值
          name: zhangsan         # 环境变量的名称
          value: "123"           # 环境变量的值
        ports:                   # 容器暴露的端口集合
        - containerPort: 80      # 容器暴露的端口
  strategy:                      # 
    type: RollingUpdate          # 部署更新的类型有Recreate重建（删除一个启动一个）和RollingUpdate滚动更新（按照更新策略滚动更新）
    rollingUpdate:               # 滚动更新
      maxSurge: 25%              # 指定在更新的时候最多超出的资源副本数有多少，两种指定方式：直接指定个数和百分比指定
      maxUnavailable: 25%        # 指定在更新的时候最多有多少资源副本不可用，两种指定方式：直接指定个数和百分比指定
~~~
 
# StatefulSet控制器（用于有状态服务）
~~~yaml
apiVersion: v1                # 资源的版本
kind: PersistentVolume        # PersistentVolume资源类型
metadata:                     # 元数据
  name: myPV                  # 资源的名称
spec:
  capacity:                   # 容量
    storage: 5Gi              # 存储容量
  volumeMode: Filesysteam     # 文件系统挂载
  accesModes:                 # 访问策略
    - ReadWriteOnce           # 设置为单节点读写挂载，命令行可以简写为RWO，访问策略在资源清单中只能写一种
  persistentVolumeReclaimPolicy: Retain # 持久卷的回收策略，保留存储的PV和PV中存储的内容，并且该PV不能被其他的PVC使用
  storageClassName: nfs       # 存储类名称
  nfs:                        # 使用nfs插件来实现PV卷
    path: /tmp                # nfs挂载路径
    server: 172.17.0.2        # nfs服务器的地址IP
---
apiVersion: v1                # 资源的版本
kind: Service                 # svc资源类型
metadata:                     # 元数据
  name: mySvc                 # 资源的名称
  lables: 
    app: mySvc                # svc的标签
spec:
  ports:
    - port: 80                # 服务暴露的端口
      name: web               # 暴露端口的名称
  clusterIP: None             # 无头服务
  selector: 
    app: mySvc                # 标签
---
apiVersion: apps/v1           # 资源的版本
kind: StatefulSet             # 资源类型
metadata:
  name: web                   # 资源名称，StatefulSet控制器控制的Pod名称是控制器的名称加从0开始的递增数字
spec:
  selector:
    matchLables:
      app: mySvc              # 标签
  serviceName: "mySvc"        # 无头服务svc的资源名称
  replices: 3                 # pod副本数量
  template:
    metadata:
      lables:
        app: mySvc            # Pod的标签
    spec:
      containers:
        - name: myapp         # 容器名称
          images: myapp:v1    # 容器镜像
          ports:
            - containerPort: 80  # 容器暴露端口
              name: web          # 容器暴露端口的名称
          volumeMounts:          # 卷挂载
            - name: www          # 卷的名称，根据名称去查找名称相同的PVC，将volume映射到容器内的目录中
              mountPath: /root/test  # 容器内挂载的路径
  volumeClaimTemplates:          # PVC的模板
    - metadata:
        name: www                # PVC的名称
      spec:
        accessModes: ["ReadWriteOnce"]  # PVC的访问策略，设置为单节点读写挂载，根据PVC的访问策略匹配相同访问策略的PV
        storageClassName: "nfs"         # PVC的类名，根据PVC的类目匹配相同类名的PV
        resources:
          requests:
            storage: 1Gi                # PVC匹配的PV大小，实际PV的大小只能相等或者是大于，有多个则选择最接近PV的匹配
~~~
 
# DaemonSet（简化版）
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
#    matchExpressions:           # 其他的标签匹配方式，不与matchLabels一起使用
#      - key: app                # 指定匹配标签的key
#        operator: Exists        # 指定标签的匹配方式，有In、NotIn、Exists、DoesNotExists。具体见下面解释
  template:                      # 定义Pod模板
    metadata:                    # 定义Pod模板元数据
      labels:                    # 定义Pod模板的全部标签
        tier: frontend           # 定义Pod模板标签元素
        app: spring-k8s          # 定义Pod模板标签元素
    spec:                        # 定义Pod模板的期望值
      containers:                # 定义Pod模板的容器
      - name: php-redis          # 容器名
        image: wangyanglinux/myapp:v1   # 镜像的来源仓库和版本 
~~~
 
# Job控制器（简化版）
~~~yaml
apiVersion: batch/v1
kind: Job                        # 资源类型是Job控制器
metadata:
  name: frontend                 # 资源的名称，这里是Job控制器的名称
spec: 
  completions: 1                 # 标志Job结束需要运行成功的Pod数量，默认是1个
  parallelism: 1                 # 标志可以同时并行运行的Pod数量，默认是1个
  activeDeadlineSeconds: 1       # 标志Pod失败的最大重试时间，单位是秒
  template:                      # 定义Pod模板
    metadata:                    # 定义Pod模板元数据
      name: frontend             # 定义Pod的名称
    spec:                        # 定义Pod模板的期望值
      containers:                # 定义Pod模板的容器
      - name: php-redis          # 容器名
        image: wangyanglinux/myapp:v1   # 镜像的来源仓库和版本 
      restartPolicy: Never       # 设置重启策略，由于是批处理任务所有不能使用默认的Always一旦Pod死亡就重启，Never是从不重启，OnFailure 是关闭退出异常时重启
~~~
 
# Cron Job控制器（简化版）
~~~yaml
apiVersion: batch/v1beta1
kind: CronJob                    # 资源类型是CronJob
metadata:
  name: frontend                 # 资源的名称，这里是CronJob的名称
spec: 
  schedule: "*/1***"               # 必填选项，CronJob调度的周期
  startingDeadlineSeconds: 3       # 可选项，Job任务执行时间单位秒，在该时间内没有启动执行完任务则认为Job任务执行失败
  concurrencyPolicy: Allow         # Job并行运行策略，有三种策略：Allow、Forbid、Replace。Allow禁止并行运行Job、Forbid是必须等待前一个Job运行完才运行下一个、Replace是如果前一个Job没有运行完，则直接取消前一个Job，并运行当前Job。默认是Allow
  suspend：false                   # 若为true则挂起所有还未开始执行的Job，默认是false
  successfulJobsHistoryLimit: 3    # 可选项，保留Job执行成功的历史副本
  failedJobsHistoryLimit: 1        # 可选项，保留Job执行失败的历史副本
  jobTemplate: 
    completions: 1                 # 标志Job结束需要运行成功的Pod数量，默认是1个
    parallelism: 1                 # 标志可以同时并行运行的Pod数量，默认是1个
    activeDeadlineSeconds: 1       # 标志Pod失败的最大重试时间，单位是秒
    template:                      # 定义Pod模板
      metadata:                    # 定义Pod模板元数据
        name: frontend             # 定义Pod的名称
      spec:                        # 定义Pod模板的期望值
        containers:                # 定义Pod模板的容器
        - name: php-redis          # 容器名
          image: wangyanglinux/myapp:v1   # 镜像的来源仓库和版本 
        restartPolicy: OnFailure   # 设置重启策略，由于是批处理任务所有不能使用默认的Always一旦Pod死亡就重启，Never是从不重启，OnFailure 是关闭退出异常时重启
~~~
 
# SVC资源清单（简化版）
~~~yaml
apiVersion: v1
kind: Service                    # 资源类型是svc
metadata:
  name: frontend                 # 资源的名称，这里是svc的名称
  namespace: default             # 资源在哪个命名空间下创建
spec: 
  type: ClusterIP                # svc的类型，默认是ClusterIP类型，如果是无头服务就不需要设置type，默认是ClusterIP类型，并且需要设置clusterIP: None
  selector:                      # 标签匹配器
    app: myappv1                 # 匹配POD的标签，标签key是app值是myappv1的标签
    release: stabel              # 匹配POD的标签，标签key是release值是stabel的标签
  ports:                         # 指定端口集合
  - name: http                   # 端口映射名称
    port: 80                     # 指定svc的端口
    targetPort: 80               # 映射的容器内部端口
~~~
 
# Ingress资源清单（包含deployment控制器和SVC资源清单），注意需要提前安装ingress-controller，也就是ingress-controller
# k8s官网ingress和ingress-controller资料参考：https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers/
# nginx ingress github官网参考：https://github.com/kubernetes/ingress-nginx/blob/main/README.md#readme
# nginx ingress 官网安装参考：https://kubernetes.github.io/ingress-nginx/deploy/

~~~yaml
apiVersion: v1                        # svc的创建，svc匹配标签key是app，value是my-app1的Pod
kind: Service
metadata:
  name: my-service-1
  namespace: default
spec:
  selector:
    app: my-app1
  ports:
  - name: my-app1
    port: 8080
    targetPort: 8080
  type: ClusterIP
---
apiVersion: apps/v1                   # Deployment的创建，会创建2个my-app1的Pod
kind: Deployment
metadata:
  name: my-app1
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app1
  template:
    metadata:
      labels:
        app: my-app1
    spec:
      containers:
      - name: my-app1
        image: my-app1:1.7.9
        ports:
        - name: http
          containerPort: 8080 
---
apiVersion: v1                              # svc的创建，svc匹配标签key是app，value是my-app2的Pod
kind: Service
metadata:
  name: my-app2
  namespace: default
spec:
  selector:
    app: my-app2
  ports:
  - name: my-app2
    port: 8081
    targetPort: 8081
  type: ClusterIP
---
apiVersion: apps/v1                       # Deployment的创建，会创建1个my-app2的Pod
kind: Deployment
metadata:
  name: my-app2
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app2
  template:
    metadata:
      labels:
        app: my-app2
    spec:
      containers:
      - name: my-app2
        image: my-app2:1.7.9
        ports:
        - name: http
          containerPort: 8081
---
apiVersion: extensions/v1beta1            # Ingress的资源版本
kind: Ingress                             # Ingress资源类型
metadata:                                 # 元数据
  name: ingress-demo-0915                 # 资源名称
  namespace: default                      # 资源所在命名空间
spec:                                     # 期望
  rules:                                  # 路由规则
    - host: my.test.com                   # 将来去往my.test.com域名的请求代理对应的服务上
      http:
        paths:
          - backend:
              serviceName: my-service-2   # 将来去往my.test.com域名并且路径是/web2的请求代理到my-service-2上
              servicePort: 8081
            path: /web2                   # 注意路径重叠覆盖，若有路径覆盖则路由规则放在前面
          - backend:
              serviceName: my-service-1   # 将来去往my.test.com域名并且路径是/web1的请求代理到my-service-1上
              servicePort: 8080
            path: /web1
    - host: a.test.com                    # 将来去往my.test.com域名的请求代理对应的服务上
      http:
        paths:
          - backend:
              serviceName: my-service-2
              servicePort: 8081
~~~
 
# Ingress-Nginx实现BasicAuth认证
参考地址：https://cloud.tencent.com/developer/article/1718482

~~~shell
# 安装httpd
yum install -y httpd              

# 设置访问Ingress代理路径的密码和再次输入密码生成auth文件，注意foo是用户名可以设置其他的用户名
htpasswd -c auth foo              

# 生产secret类型的资源，资源名称是basic-auth
kubectl create secret generic basic-auth --from-file=auth  

# 查看资源清单
kubectl get secret basic-auth -o yaml 

# Ingress资源清单添加3个注解
# metadata:
#   annotations:
#     nginx.ingress.kubernetes.io/auth-type: basic
#     nginx.ingress.kubernetes.io/auth-secret: basic-auth
#     nginx.ingress.kubernetes.io/auth-realm: 'Authentication Required - foo'

# 访问Ingress代理路径会需要用户名和密码
~~~
