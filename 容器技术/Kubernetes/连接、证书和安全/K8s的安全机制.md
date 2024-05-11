### K8s是一个分布式集群的管理工具，保证集群的安全性是一个重要的任务，API Server是集群内部各个组件通信的中介，也是外部控制的入口，所以K8s的安全机制基本就是围绕保护API Server设计的。K8s使用了认证（Authentication）、鉴权（Authorization）、准入控制（AdmissionControl）三步来保证API Server的安全。

- 认证：是否是内部成员
- 鉴权：判断是否有权限查看和执行操作
- 准入控制：不允许执行危险操作

# 认证（Authentication）

### 认证发展的三个阶段

- HTTP Token 认证：通过一个Token来识别合法用户
  - HTTP Token的认证是用一个很长的特殊编码方式的并且难以被模仿的字符串---Token来表达客户的一种方式。每一个Token对应一个用户名存储在API Server能访问的文件中。当客户端发起API调用请求时，需要在HTTP - Header 里放入Token
- HTTP Base认证：通过用户名+密码的方式认证
  - 用户名+密码使用BASE64算法进行编码后的字符串放在HTTP Request中的Header Authorization 域里发送给服务端，服务端收到后进行编码，获取用户名及密码
- 最严格的HTTPS证书认证：基于CA根证书签名的客户端身份认证方式
  - 通过一个CA架构给客户端和服务端各自发送一个证书，客户端和服务端各种交换证书进行验证，双方验证成功则通过认证
---

### 需要进行HTTPS认证的节点有两种类型

- K8s组件对API Server 的访问是通过HTTPS双向认证：kubectl、Colltroller Manager、Scheduler、kubelet、kube-proxy。其中Colltroller Manager、Scheduler和API Server在同一机器下不需要进行HTTPS认证直接使用API Server的非安全端口访问既--insecure-bind-address=127.0.0.1  而kubectl、kubelet、kube-proxy访问API Server就需要证书进行HTTPS双向认证
- K8s管理的Pod对容器的访问是通过Secret存储中的Service Account类型进行的认证（简称SA认证）：Pod（dashborad也是以Pod形式运行）
---

### 在K8s中有两对证书

- master节点下/etc/kubernetes/pki目录下的 ca.crt 文件和ca.key文件用于访问API Server的证书
- master节点下/etc/kubernetes/pki/etcd/目录下的 ca.crt 文件和ca.key文件用于访问ETCD的证书

### 两对证书分别对应

- 对于ETCD来说，ETCD是服务端，API Server是客户端
- 对于API Server来说，API Server是服务端，Colltroller Manager、Scheduler、kubectl、kubelet、kube-proxy是客户端，但是由于Colltroller Manager、Scheduler一般和API Server在同一个节点上所以它们不需要HTTPS证书认证
  - kubelat是集群自动颁发证书
  - kubectl，kube-proxy需要手动颁发证书
---

# 鉴权（Authorization）

### 鉴权是确认请求方有哪些资源的权限。API Server目前支持以下几种授权策略（通过API Server的启动参数 "--authorization-mode" 设置）

- AlwaysDeny：表示拒绝所有的请求，一般用于测试
- AlwaysAllow：允许接收所有请求，如果集群不需要授权流程，则可以采用该策略
- ABAC（Attribute-Based Access Control）：基于属性的访问控制，表示使用用户配置的授权规则对用户请求进行匹配和控制
- Webhook：通过调用外部REST服务对用户进行授权
- RBAC（Role-Based Access Control）：基于角色的访问控制，现行默认规则

## RBAC授权模式

### RBAC授权模式是基于角色的访问控制，在K8s1.5引入，现行版本成为默认标准。相对其他的访问控制方式，拥有以下优势：

- 对集群中的资源和非资源均拥有完整的覆盖
- 整个RBAC完全由几个API对象完成，同其他API对象一样，可以用kubectl或者API进行操作，这些API对各个编程语言都有SDK
- 可以在运行时进行调整，无需重启API Server
--- 

### 用户、组、SA和角色

- 用户、组、SA可以绑定角色，角色绑定权限
- 角色分为普通角色和集群角色，普通角色只能操作查看该角色命名空间下有权限的资源，集群角色可以操作查看集群下所有命名空间有权限的资源
- 普通用户和集群用户，普通用户只在该用户的命名空间下生效，集群用户在所有命名空间下生效
- 绑定角色有三种绑定
  - 用户roleBinding绑定普通角色，该用户可以操作查看绑定的普通角色所在命名空间下有权限的资源
  - 组ClusterRoleBinding绑定集群角色，该组用户可以操作查看绑定的集群角色所有命名空间下有权限的资源
  - 普通用户roleBinding绑定集群角色，该用户可以操作查看绑定的集群角色在该用户的命名空间下有权限的资源


### 用户和组

K8s并不会提供用户管理，User、Group、ServiceAccount指定的用户从证书中获取，K8s组件kubectl、kube-proxy或是其他自定义的用户在向CA申请证书时，需要提供一个证书请求文件，其中就有用户
~~~json
{
    "CN": "admin",        // 用户名  system:前缀是系统用户不可使使用
    "hosts": [],          // 规定只有那些IP可以使用这个证书，一个不写就是所有IP都可以使用
    "key": {
        "algo": "rsa",    // 加密算法
        "size": "2048"    // 加密长度
    },
    "names": [
        {
            "C": "CN",
            "ST": "HangZhou",
            "L": "XS",
            "0": "system:masters",  // Group组名，system:前缀是系统组不可创建，system:masters是系统内置组
            "OU": "System"
        }
    ]
}
~~~

kubelet使用TLS Bootstaping认证时，API Server可以使用Bootstrap Token或者是Token authentication file 验证 =token，无论哪一种，k8s都会为token绑定一个默认的User和Group 

Pod使用ServiceAccount 认证时，service-account-token中的JWT会保存User信息，有了用户信息，再创建一对角色/角色绑定（集群角色/集群角色绑定）资源对象，就可以完成权限绑定了

### Role 和 ClusterRole

在RBAC API中，Role表示一组规则权限，权限只会增加（累加权限），不存在一个资源一开始就有很多权限而通过RBAC对其进行减少的操作；Role可以定义在一个namespace中，如果想要跨namespace则可以创建ClusterRole，ClusterRole拥有以下用途

- 集群级别的资源控制（例如node访问控制）
- 非资源型endpoints (例如 /health 访问)
- 所有命名空间资源控制（例如pods）
---

Role资源清单
~~~yaml
apiVersion: rbac.authorization.k8s.io/v1       # 资源版本
kind: Role                                     # 资源类型
metadata: 
  namespace: default                           # 资源所在命名空间
  name: pod-reader                             # 资源名称
rules:
  - apiGroups: [""]                            # 权限来源的资源核心版本，比如给予pod读取权限，就是Pod资源的核心版本，空字符串就是核心的v1版
    resources: ["pods"]          # 赋予权限的资源类型，如果是子资源类型比如pod的日志则使用/分隔资源和子资源例如 pods/log
    verbs: ["get", "watch", "list"]            # 角色可以执行的动作，get是获取资源，watch是监听资源，list是列出资源
~~~

ClusterRole资源清单
~~~yaml
apiVersion: rbac.authorization.k8s.io/v1       # 资源版本
kind: ClusterRole                              # 资源类型
metadata: 
  name: secret-reader                          # 资源名称
rules:
  - apiGroups: [""]                            # 权限来源的资源核心版本，比如给予pod读取权限，就是Pod资源的核心版本，空字符串就是核心的v1版
    resources: ["secrets"]                     # 赋予权限的资源类型
    verbs: ["get", "watch", "list"]            # 角色可以执行的动作，get是获取资源，watch是监听资源，list是列出资源 
~~~

API URL例子
~~~
GET /api/v1/namespaces/{namespace}/pods/{pod}/log
~~~

用户roleBinding绑定普通角色
~~~yaml
apiVersion: rbac.authorization.k8s.io/v1       # 资源版本
kind: RoleBinding                              # 资源类型
metadata: 
  namespace: default                           # 命名空间
  name: read-pods                              # 资源名称
subjects:
  - kind: User                                 # 用户绑定角色，还可以是SA或者是组绑定角色
    name: jane                                 # 用户名，还可以是组名或者是SA的资源名称
    apiGroup: rbac.authorization.k8s.io        # 用户类型，组类型或者SA类型的核心版本
roleRef:
  kind: Role                                 # 绑定的角色类型，是普通角色
  name: pod-reader                           # 角色名称
  apiGroup: rbac.authorization.k8s.io        # 角色类型的核心版本
~~~

用户RoleBinding绑定集群角色
~~~yaml
apiVersion: rbac.authorization.k8s.io/v1       # 资源版本
kind: RoleBinding                              # 资源类型
metadata: 
  namespace: dev                               # 命名空间
  name: read-secrets                           # 资源名称
subjects:
  - kind: User                                 # 用户绑定角色，还可以是SA或者是组绑定角色
    name: dave                                 # 用户名，还可以是组名或者是SA的资源名称
    apiGroup: rbac.authorization.k8s.io        # 用户类型，组类型或者SA类型的核心版本
roleRef:
  kind: ClusterRole                            # 绑定的角色类型，是集群角色
  name: secret-reader                          # 角色名称
  apiGroup: rbac.authorization.k8s.io          # 角色类型的核心版本
~~~

组ClusterRoleBinding绑定集群角色
~~~yaml
apiVersion: rbac.authorization.k8s.io/v1       # 资源版本
kind: ClusterRoleBinding                       # 资源类型
metadata: 
  name: read-secrets                           # 资源名称
subjects:
  - kind: Group                                # 组绑定角色，还可以是SA或者是用户绑定角色
    name: manager                              # 组名，还可以是用户名或者是SA的资源名称
    apiGroup: rbac.authorization.k8s.io        # 用户类型，组类型或者SA类型的核心版本
roleRef:
  kind: ClusterRole                            # 绑定的角色类型，是集群角色
  name: secret-reader                          # 角色名称
  apiGroup: rbac.authorization.k8s.io          # 角色类型的核心版本
~~~
 

证书相关参考文章：https://juejin.cn/post/7016472622246395934#heading-4

 

创建一个新的用户登录到k8s集群，并设置它的权限
第一步：创建证书
第二步：将创建的证书转换成KubeConfig文件，KubeConfig中存在集群的访问地址、CA的私钥和公钥、用户
第三步：创建一个命名空间，在该命名空间下创建普通角色
第四步：kebectl根据KubeConfig登录到创建命名空间下

详细情况如下
~~~shell
# 在master节点下/etc/kubernetes/pki目录下创建证书
touch devuser.json

# 向devuser.json写入以下json,注意去除注释
# {
#     "CN": "devuser",        // 用户名  system:前缀是系统用户不可使使用
#     "hosts": [],            // 规定只有那些IP可以使用这个证书，一个不写就是所有IP都可以使用
#     "key": {
#         "algo": "rsa",      // 加密算法
#         "size": "2048"      // 加密长度
#     },
#     "names": [
#         {
#             "C": "CN",
#             "ST": "HangZhou",
#             "L": "HangZhou",
#             "0": "k8s",      // Group组名，system:前缀是系统组不可创建，system:masters是系统内置组
#             "OU": "System"
#         }
#     ]
# }

#下载证书生成工具
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl

wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo

chmod a+x /usr/local/bin/cfssl*

# 签发证书，注意如果需要修改用户名就修改devuser.json的用户名和命令最后的参数devuser
# 执行完命令后会出现 devuser-key.pem 私钥和devuser.pem 证书文件，文件名中的devuser是新创建的用户名
cfssl gencert -ca=ca.crt -ca-key=ca.key -profile=kubernetes devuser.json | cfssljson -bare devuser

# 设置集群环境变量和参数master节点API Server端口，参数设置完之后会出现devuser.kubeconfig文件
export KUBE_APISERVER="https://172.20.0.113:6443"
kubectl config set-cluster kubernetes --certificate-authority=ca.crt --embed-certs=true --server=${KUBE_APISERVER} --kubeconfig=devuser.kubeconfig

# 设置客户端认证参数
kubectl config set-credentials devuser --client-certificate=devuserpem --client-key=devuser-key.pem --emben-certs=true --kubeconfig=devuser.kubeconfig

# 设置上下文参数
kubectl config set-context kuberentes --cluster=kubernetes --user=devuser --namespace=dev --kubeconfig=devuser.kubeconfig

# 创建命名空间
kubectl create namespace dev

# 设置默认上下文
kubectl config use-context kubernetes --kubeconfig=dvuser.kubeconfig

# 用户devuser绑定集群角色admin, 可以使用--dry-run -o yaml查看绑定的资源清单
kubectl create rolebinding devuser-admin-binding --clusterrole=admin --user=devuser --namespace=dev

# 角色绑定完了之后，只要有devuser.kubeconfig这个文件就是devuser用户有集群角色admin的权限
# 利用Linux系统创建一个用户
# 将devuser放到该用户的家目录下 ~/.kube/config中，这样该用户登录Linux系统在K8s中就是devuser用户并且有集群角色admin权限
useradd feiyongjing
password feiyongjing  # 设置密码

mkdir /home/feiyongjing/.kube
cp -a devuser.kubeconfig /home/feiyongjing/.kube/config
chown feiyongjing.feiyongjing -R /home/feiyongjing/.kube  # 赋予该用户目录权限

# 使用Linux系统新创建的用户使用kubectl命令默认是在dev的命名空间，并且只有该命名空间的admin权限
~~~
 

# 准入控制（AdmissionControl）
准入控制是API Server的插件集合，通过添加不同的插件，实现额外的准入控制规则。甚至是API Server的一些注意功能都需要通过Admission Controllers实现，比如ServiceAccount

官方文档上有一份针对不同版本的准入控制器推荐列表，其中最新的1.14的推荐列表是：
~~~
NamespaceLifecycle, LimitRanger, ServiceAccount, DefaultStorageClass, DefaultTolerationSeconds, MutatingAdmissionWebhook, ValidatingAdmissionWebhook, ResourceQuota
~~~
 
推荐的几个插件功能

- NamespaceLifecycle：防止在不存在的namespace上创建对象，防止删除系统预置namespace，删除namespace时，连带删除它的所有资源对象
- LimitRanger：确保请求的资源不会超过资源所在Namespace的LimitRange的限制
- ServiceAccount：实现了自动化添加ServiceAccount
- ResourceQuota：确保请求的资源不会超过资源的ResourceQuota限制
 