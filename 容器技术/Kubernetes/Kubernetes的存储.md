# Kubernetes的存储类型有4种：configmap类型、Secret类型、volume类型、PersistentVolume类型

- configMap存储普通的配置信息
- Secret存储需要加密的配置信息
- volume
- PersistentVolume类型存储持久化的数据
 
# configmap类型
configmap类型存储的是单个属性或者是整个配置文件或者是JSON二进制对象，它们都是KV对

configMap资源的创建方式
1. 根据目录来创建configmap，configmap在命令行的缩写是cm
~~~shell
# 针对目录来创建configmap，--from-file指定的目录下所有文件会在ConfigMap组成一组键值对，key是文件的名称，value是文件的内容
kubectl create configmap 'configmap资源的名称' --from-file='目录路径'

# 查看configmap的yaml文件
kubectl create configmap 'configmap资源的名称' --from-file='目录路径' --dry-run -o yaml
kubectl get cm 'configmap资源的名称' -o yaml 
~~~
2. 根据文件来创建configmap，configmap在命令行的缩写是cm
~~~shell
# 针对文件来创建configmap，--from-file指定的文件会在ConfigMap组成一个键值对，key是文件的名称，value是文件的内容
kubectl create configmap 'configmap资源的名称' --from-file='文件路径'
~~~
3. 根据命令直接指定KV对，configmap在命令行的缩写是cm
~~~shell
# --from-literal直接指定一对KV对
kubectl create configmap 'configmap资源的名称' --from-literal=name=张三 --from-literal=password=123456 
~~~
4. 根据资源清单创建
~~~shell
apiVersion: v1                # configMap的版本
kind: ConfigMap               # 创建的资源类型
data:                         # 配置的数据，这是第一和第二种生成configMap的资源清单
  game.txt: |                 # 配置的key,也就是配置文件的名称，使用|符号另起一行代表文件的内容，注意文件内容的换行
    version=1.0               # 配置的value第一行,也就是配置文件的内容，使用xxx=aaa的kv对是天然的环境变量结构能被当作环境变量被读取
    name=root                 # 配置的value第二行
    password=123456           # 配置的value第三行
  url.txt: |                  # 配置的key,也就是配置文件的名称，使用|符号另起一行代表文件的内容，注意文件内容的换行
    dev=https://xxx.com/xxx   # 配置的value第一行,也就是配置文件的内容，使用xxx=aaa的kv对是天然的环境变量结构能被当作环境变量被读取
    prod=https://aaa.com/aaa  # 配置的value第二行
metadata:                     # 元数据
  name: myappConfig           # configmap资源的名称
---
apiVersion: v1                # configMap的版本
kind: ConfigMap               # 创建的资源类型，这是第三种生成configMap的资源清单
data:                         # 配置的数据
  version: 1.0                # key是version,value是1.0
  name: root                  # key是name,value是root
  password: 123456            # key是password,value是123456
metadata:                     # 元数据
  name: myappUrlConfig        # configmap资源的名称
~~~
--- 

### pod中使用configMap做环境变量(一旦启动容器就无法再次修改配置的环境的变量了，需要修改configMap重启和重启Pod，即无法热更新 
### pod中使用volume注入进行configMap做环境变量(即热更新，但是仅仅只是更新容器内的配置文件，应用程序不会更新)

### 注意是volume注入，而不是挂载，k8s的volume的注入与docker的文件挂载相同的地方是都能实现热更新，不同的是docker的文件挂载在实时的立即更新属于IO密集型，在工作节点多的情况下占用的资源过多，而k8s的volume的注入不是实时的立即更新，而是保证一段时间后读取volume的更新再注入到容器的文件中，在工作节点多的情况下占用的资源相对少一些

### 注意通过kubectl edit编辑configMap资源进行热更新时，如果在etcd中configMap资源清单metadata.annotations中没有data配置属性则只修改data下的配置属性，如果metadata.annotations中有配置的属性就需要metadata.annotations中data配置的属性和data下的属性一起进行修改

~~~yaml
apiVersion: v1                # configMap的版本
kind: ConfigMap               # 创建的资源类型，这是第三种生成configMap的资源清单
data:                         # 配置的数据
  version: 1.0                # key是version,value是1.0
  name: root                  # key是name,value是root
  password: 123456            # key是password,value是123456
metadata:                     # 元数据
  name: myappConfig           # configmap资源的名称 
---
apiVersion: v1                # Pod资源的版本
kind: Pod                     # 创建的资源类型
metadata:                     # 元数据
  name: myapp                 # Pod资源的名称
spec:
  volumes:                             # 设置volume映射的配置文件
    - name: config-volume              # volume的名称，根据volume的名称映射的配置文件会被映射到容器内部目录下
      configMap:                       # 设置volume映射的配置资源是configMap
        name: myappConfig              # 设置volume映射的configMap资源名称
  containers:                          # 容器的创建方式
    - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
      imagePullPolicy: IfNotPresent    # 镜像的下载策略，可以不写使用默认
      name: coredns1                   # 创建出的容器的名字
      volumeMounts:                    # 将volume映射到容器内的目录中
        - name: config-volume          # volume的名称，根据volume的名称映射的配置文件会被映射到容器内部目录下
          mountPath: /etc/config       # 容器内部的目录，该目录下有根据volume的名称映射的配置文件
~~~
---
 
修改deployment控制器的资源清单中Pod的配置注解来进行更新Pod的配置
~~~shell
kubectl patch deployment 'deployment控制器的名称' --patch '{"spec": {"template": {"metadata": {"annotations": {"version/config": "配置的版本号"}}}}}'
# 配置的版本号随便修改，只要与原版本号不同就会触发Pod更新配置文件
~~~
--- 

# Secret类型
### Secret类型存储解决了密码、token、密钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者Pod spec中。Secret可以以Volume或者环境变量的方式使用

## Secret有三种类型
- Service Account 类型用来访问Kubernetes API ，由Kubernetes自动创建，并且会自动挂载到Pod的 /run/secrets/kubernetes.io/serviceaccount 目录中
- Opaque：base64编码格式的Secret，用来存储密码、密钥等。注意Opaque类型不提供配置信息的安全保证。
- kubernetes.io/dockerconfigjson：用来存储私有docker仓库的认证信息

## Service Account 类型
默认情况下，每个namespace都要一个ServiceAccount，如果Pod在创建时没有指定ServiceAccount，就会使用Pod所属的namespace的ServiceAccount 
 进入Pod内部cd 切换目录到 /run/secrets/kubernetes.io/serviceaccount 中 查看该目录下的文件

~~~shell
cd /run/secrets/kubernetes.io/serviceaccount   #切换目录到 /run/secrets/kubernetes.io/serviceaccount 中 
ls     #查看该目录下的文件
# ca.crt     namespace  token

# token是JWT,Json web token,用于单点认证apiServer验证Pod的是否合法，
# ca.crt证书是用于Pod认证apiServer发送的证书是否有效
# namespace是当前Pod所在的命名空间

# 双向认证
# 既每个Pod向API Server发送token和namespace，API Server会通过toekn确认该Pod是集群所管理的，如果是的API Server认证通过，会根据nameSpace给Pod发送ca.crt证书
# Pod会比对API Server返回的ca.crt证书和自己的ca.crt证书是否一致，如果一致则Pod认证通过
~~~
--- 

## Opaque 类型
Opaque中存储的配置是真正的密码等信息进行base64编码后的字符串，该字符串不保证密码的安全，因为要拿到编码后的字符是可以进行反编码拿到真正的密码

和configMap类型的存储一样Opaque类型的存储也一样是key和value的结构，但是注意的是value必须是原数据进行base64编码后得到的字符串，因为在Opaque类型的配置在注入Pod时会自动的进行base64的解码，所以容器内部使用配置是原数据

base64编码和解码方式如下
~~~shell
$ echo -n "admin" | base64             # 对admin字符串进行base64编码
YWRtaW4=                               # 进行base64编码后输出的字符串
$ echo -n "fyj123456" | base64         # 对admin字符串进行base64编码
ZnlqMTIzNDU2                           # 进行base64编码后输出的字符串

$ echo -n "YWRtaW4=" | base64 -d       # 对base64编码进行解码
admin                                  # 进行base64解码后输出的原字符串
$ echo -n "ZnlqMTIzNDU2" | base64 -d   # 对base64编码进行解码
fyj123456                              # 进行base64解码后输出的原字符串
~~~
--- 

Pod中使用Volume注入Opaque 类型存储的配置
~~~yaml
apiVersion: v1                # Secret的版本
kind: Secret                  # 创建的资源类型
type: Opaque                  # Secret资源下的类型设置，现在是Opaque类型
data:                         # 配置的数据
  name: YWRtaW4=              # key是name,value是root进行base64编码后的字符串
  password: ZnlqMTIzNDU2      # key是password,value是fyj123456进行base64编码后的字符串
metadata:                     # 元数据
  name: myappSecret           # Secret资源的名称 
---
apiVersion: v1                # Pod资源的版本
kind: Pod                     # 创建的资源类型
metadata:                     # 元数据
  name: myapp                 # Pod资源的名称
spec:
  volumes:                             # 设置volume映射的配置文件
    - name: config-volume              # volume的名称，根据volume的名称映射的配置文件会被映射到容器内部目录下
      secret:                          # 设置volume映射的配置资源是secret
        secretName: myappSecret        # 设置volume映射的secret资源名称
  containers:                          # 容器的创建方式
    - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
      imagePullPolicy: IfNotPresent    # 镜像的下载策略，可以不写使用默认
      name: coredns1                   # 创建出的容器的名字
      volumeMounts:                    # 将volume映射到容器内的目录中
        - name: config-volume          # volume的名称，根据volume的名称映射的配置文件会被映射到容器内部目录下
          mountPath: /etc/config       # 容器内部的目录，该目录下有根据volume的名称映射的配置文件

~~~
--- 

Pod中使用环境变量引用Opaque 类型存储的配置
~~~yaml
apiVersion: v1                # Secret的版本
kind: Secret                  # 创建的资源类型
type: Opaque                  # Secret资源下的类型设置，现在是Opaque类型
data:                         # 配置的数据
  name: YWRtaW4=              # key是name,value是root进行base64编码后的字符串
  password: ZnlqMTIzNDU2      # key是password,value是fyj123456进行base64编码后的字符串
metadata:                     # 元数据
  name: myappSecret           # Secret资源的名称 
---
apiVersion: v1                # Pod资源的版本
kind: Pod                     # 创建的资源类型
metadata:                     # 元数据
  name: myapp                 # Pod资源的名称
spec:
  containers:                          # 容器的创建方式
    - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
      imagePullPolicy: IfNotPresent    # 镜像的下载策略，可以不写使用默认
      name: coredns1                   # 创建出的容器的名字
      env:                             # 容器的环境变量，引入一个环境变量数组，每个数组元素就是一个环境变量
        - name: USERNAME               # 指定容器的环境变量名称
          valueFrom:                   # 指定容器的环境变量的值从哪里来
            secretKeyRef:              # 容器的环境变量的值从secret类型的资源中引用
              name: myappSecret        # 指定secret类型的资源的名称
              key: name                # 指定secret类型资源中的key,key对应的值就是环境变量的值
        - name: PASSWORD               # 指定容器的环境变量名称
          valueFrom:                   # 指定容器的环境变量的值从哪里来
            secretKeyRef:              # 容器的环境变量的值从secret类型的资源中引用
              name: myappSecret        # 指定secret类型的资源的名称 
              key: password            # 指定secret类型资源中的key,key对应的值就是环境变量的值
      envFrom:                      # 容器的环境变量，引入一个secret数组，每个数组secret元素中的所有key和value就是环境变量的key和value
        - configMapKeyRef:             # 容器的环境变量的值从secret类型的资源中引用
            name:  myappUrlConfig      # 指定secret类型的资源的名称 
~~~
--- 

# kubernetes.io/dockerconfigjson类型
pod从私有镜像仓库拉取镜像需要登陆私有镜像仓库，而kubernetes.io/dockerconfigjson存储登陆私有镜像仓库的私密信息

创建secret资源下kubernetes.io/dockerconfigjson类型的资源
~~~shell
kubectl create secret docker-registry myregistrykey --docker-server="私有镜像仓库服务的域名" --docker-username="私有镜像仓库登陆的用户名" --docker-password="私有镜像仓库登陆的密码" --docker-email="用户邮箱" 

# myregistrykey是创建的secret资源的名称
~~~
---
在Pod在引用kubernetes.io/dockerconfigjson类型的资源从私有镜像仓库拉取镜像
~~~yaml
apiVersion: v1                # Pod资源的版本
kind: Pod                     # 创建的资源类型
metadata:                     # 元数据
  name: myapp                 # Pod资源的名称
spec:
  imagePullSecrets:           # 从Secret资源中引入私有镜像仓库的信息
    - name myregistrykey      # Secret资源的名称
  containers:                          # 容器的创建方式
    - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
      name: coredns1                   # 创建出的容器的名字
~~~
--- 

# volume常见的类型
## emptyDir卷

当Pod分配给工作节点时，首先创建emptDir卷，并且只要Pod在该节点运行，该卷就会存在，该卷在最初是空的，Pod中的容器可以读取和写入emptDir卷中相同的文件，该卷可以挂载到每个容器相同或者不同的路径中。当出于任何原因从节点中删除Pod时，emptDir中的数据将被永久删除
~~~yaml
apiVersion: v1                # Pod资源的版本
kind: Pod                     # 创建的资源类型
metadata:                     # 元数据
  name: myapp                 # Pod资源的名称
spec:
  volumes:                             # 设置volume映射的配置文件
    - name: config-volume              # volume的名称，根据volume的名称映射的配置文件会被映射到容器内部目录下
      emptyDir: {}                     # 创建emptDir类型的volume
  containers:                          # 容器的创建方式
    - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
      name: coredns1                   # 创建出的容器的名字
      volumeMounts:                    # 将volume映射到容器内的目录中
        - name: config-volume          # volume的名称，根据volume的名称映射的配置文件会被映射到容器内部目录下
          mountPath: /etc/config       # 容器内部的目录，该目录下有根据volume的名称映射的配置文件
    - image: k8s.gcr.io/coredns:1.3.2  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
      name: coredns2                   # 创建出的容器的名字
      volumeMounts:                    # 将volume映射到容器内的目录中
        - name: config-volume          # volume的名称，根据volume的名称映射的配置文件会被映射到容器内部目录下
          mountPath: /etc/test/        # 容器内部的目录，由于vulume是emptyDir类型所有该目录下是空的
~~~
--- 

## hostPath卷

hostPath卷将主机节点的文件系统或目录挂载到集群中

hostPath卷除了path属性之外还有type属性，它的值有如下选择
~~~
""                     |   空字符串（默认是空字符串），用于向后兼容，这意味着在挂载hostPath卷之前不会执行任何检查。
DirectoryOrCreate      |   如果在工作节点给定的路径上没有东西存在，那么会在那里创建一个空目录，权限是0755，与Kubelet具有相同的组和所有权
Directory              |   工作节点给定的路径下必须是一个目录
FileOrCreate           |   如果工作节点给定的路径上没有任何东西存在，那么会根据需要创建一个空文件，权限是0644，与Kubelet具有相同的组和所有权
File                   |   工作节点给定的路径下必须存在文件
Socket                 |   工作节点给定的路径下必须存在UNIX套接字
CharDevice             |   工作节点给定的路径下必须存在字符设备
BlockDevice            |   工作节点给定的路径下必须存在快设备
~~~
 使用hostPath类型的卷请注意：

- 请确保每个工作节点文件系统路径中的路径和内容全部保持一致，否则Pod在不同节点的行为会有所不同
- 按照type属性检查工作节点目录是发符合要求，如果不符合那么由于hostPath卷的Pod是无法正常启动的
- Kubenetes按照计划添加资源感知调度时，无法考虑hostPath使用的资源
- 在工作节点上创建的文件和目录只能由root写入，所有在Pod内部的容器需要以root身份运行进程或者是修改工作节点上的文件权限才能写入hostPath卷

~~~yaml
apiVersion: v1                # Pod资源的版本
kind: Pod                     # 创建的资源类型
metadata:                     # 元数据
  name: myapp                 # Pod资源的名称
spec:
  volumes:                             # 设置volume映射的配置文件
    - name: config-volume              # volume的名称，根据volume的名称映射的配置文件会被映射到容器内部目录下
      hostPath:                        # hostPath的卷
        type: Directory                # hostPath卷的类型
        path: /data                    # hostPath指定的工作节点路径
  containers:                          # 容器的创建方式
    - image: k8s.gcr.io/coredns:1.3.1  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
      name: coredns1                   # 创建出的容器的名字
      volumeMounts:                    # 将volume映射到容器内的目录中
        - name: config-volume          # volume的名称，根据volume的名称映射的配置文件会被映射到容器内部目录下
          mountPath: /etc/config       # 容器内部的目录，该目录下有根据volume的名称映射的配置文件
    - image: k8s.gcr.io/coredns:1.3.2  # 创建容器使用的镜像,在yml中表示数组的-符号前对于上一行来说两个空格可以加，也可以不加
      name: coredns2                   # 创建出的容器的名字
      volumeMounts:                    # 将volume映射到容器内的目录中
        - name: config-volume          # volume的名称，根据volume的名称映射的配置文件会被映射到容器内部目录下
          mountPath: /etc/test/        # 容器内部的目录
~~~
 

# PersistentVolume类型

Pod创建PVClaim，PVClaim中有PV的默认配置，如PV的读写策略、存储大小、回收策略、类。PVClaim根据这些配置去寻找符合条件的PV

PV分两种静态PV和动态PV，静态PV的存储大小是固定，动态PV是根据需求动态的申请存储大小，动态PV只在云平台有。

### PVClaim会保护数据的持久性，使PV不会因为Pod的突然崩溃而丢失，所以要删除PV，需要先删除Pod，再删除PVClaim，最后删除PV

PersistentVolume类型在K8s中是以插件形式实现的，使用不同的PV都需要下载对应的插件，K8s目前支持以下插件类型：
- GCEPersistentDisk   AWSElasticBlockStore    AzureFile   AzureDisk  FC(Fibre Channel)
- FlexVolume   Flocker  NFS   ISCSI  RBD(Ceph Block Device)  CephFS
- Cinder(OpenStack block storage)  Glusterfs  VsphereVolume  Quobyte  Volumes
- HostPath  VMware Photon  Portworx Volumes  ScaleIO Volumes  StorageOS

使用一台额外的服务器做NFS服务器存储持久化的文件，K8s集群和NFS服务器安装NFS插件（注意在每个节点上都需要安装NFS插件）

~~~shell
yum install -y nfs-common nfs-utils rpcbind  # K8s集群和NFS服务器安装NFS插件

mkdir /nfsdata  # NFS服务器创建目录

chmod 666 /nfsdata  # NFS服务器目录赋予读写权限

chown nfsnobody /nfsdata  # NFS服务器目录的所有者设置为nfsnobody

vim /etc/exports # 编辑NFS服务器挂载的目录设置

/nfsdata *(rw,no_root_squash,no_all_squash,sync)  # 在/etc/exports写入这个，/nfsdata目录可以被其他人远程挂载，可以写多行挂载多个目录
    
systemctl start rpcbind  # NFS服务器启动rpcbind
systemctl start nfs      # NFS服务器启动nfs

systemctl enable rpcbind  # NFS服务器开机自启动rpcbind
systemctl enable nfs      # NFS服务器开机自启动nfs

mkdir /test       # 使用其他服务器创建目录用于挂载nfs设置的挂载目录进行测试nfs服务是否可以连接挂载

mount -t nfs "nfs服务器IP":"nfs服务器上/etc/exports中挂载的绝对路径" /test/  # nfs服务器上挂载的绝对路径会挂载到/tets/目录下

umount /test/  # 解除/test/目录的挂载
~~~

## PV和PVC的资源清单

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
 

### 访问策略是根据供应商提供的PV类型来设置，如NFS支持单节点读写、多节点读、多节点读写，但是访问策略在资源清单中只能写其中的一种

- ReadWriteOnce  该卷可以被单个节点读写挂载，命令行简写为RWO
- ReadOnlyMany    该卷可以被多个节点只读挂载，命令行简写为ROX
- ReadWriteMany   该卷可以被多个节点读写挂载，命令行简写为RWX

### 持久卷的回收策略

- Retain 保留存储的PV和PV中存储的内容，并且该PV不能被其他的PVC使用，只能由运维管理员来手动删除回收
- Recycle 保留存储的PV，删除PV中的内容（就是执行rm -rf 删除存储的内容），在删除PV内容后该PV可以被其他的PVC所使用，目前只有HostPath支持
- Delete 不报错存储内容，并且直接删除PV，目前只有云供应商支持

### PV卷的状态

- Available （可用）  一块空闲的PV卷还没有被PVC绑定
- Bound  （以绑定）  PV卷已经被PVC声明绑定
- Released （以释放）PV卷已经声明被删除，但是还未被集群重新声明可以被PVC使用
- Failed (失败)  PV卷的自动回收失败
 
### 如果删除与PVC绑定的Pod，那么与原来Pod绑定的PV会是Released 状态，使用原来的Pod清单新创建的Pod可以直接使用这个状态的PV，但是如果使用新的Pod清单则无法使用原来的PV。如果要使用原来的PV则需要先删除PVC然后去查看PV中存储的内容是否需要备份保存，备份保存后将PV的卷恢复为Available，方式有两种

- 先去查看PV中存储的内容是否需要备份保存，备份保存后先删除PVC，然后再删除PV，重新创建PV和PVC
- 先去查看PV中存储的内容是否需要备份保存，备份保存后先删除PVC，再kubectl edit 编辑PV的资源清单删除资源清单中spec.claimRef内的全部属性（其实就是清楚PV绑定的PVC信息，也就是该PV会恢复为Available （可用）  一块空闲的PV卷还没有被PVC绑定），重新创建PVC去绑定PV
 