### Kubernetes是一个分布式资源管理系统，特点是消耗的资源少、开源、节点弹性伸缩、负载均衡

### Kubernetes架构组成图:https://ln-normal.oss-cn-shanghai.aliyuncs.com/u-5a367e0bab644113c6001dc9%2F625296fcab64412e4504ce93_UnTitled?Expires=1692615355&OSSAccessKeyId=LTAISK6X9HhMsiQX&Signature=zBCIA2LYv5VL06ZDCx9Zueryi98%3D

Kubernetes属于主从分布式架构，主要由Master Node（主节点）和Worker Node（工作节点）组成，以及包括客户端命令行工具kubectl、web UI和其它插件。

- Master Node：作为主节点，对集群进行调度管理；Master Node由API Server、Scheduler、Cluster State Store（etcd）和Controller-Manger Server所组成

  - API Server：API Server主要用来处理REST的操作，确保它们生效，并执行相关业务逻辑，以及更新etcd（或者其他存储）中的相关对象。API Server是所有REST命令的入口，它的相关结果状态将被保存在etcd（或其他存储）中。API Server的基本功能包括：REST语义，监控，持久化和一致性保证，API 版本控制，放弃和生效、内置准入控制语义，同步准入控制钩子，以及异步资源初始化API注册和发现，API Server也作为集群的网关。默认情况，客户端通过API Server对集群进行访问，客户端需要通过认证，并使用API Server作为访问Node和Pod（以及service）的堡垒和代理/通道。
  - Scheduler：scheduler是调度器，为容器自动选择运行的主机。依据请求资源的可用性，服务请求的质量等约束条件，scheduler监控未绑定的pod，并将其绑定至特定的node节点。Kubernetes也支持用户自己提供的调度器，Scheduler负责根据调度策略自动将Pod部署到合适Node中，调度策略分为预选策略和优选策略，Pod的整个调度过程分为两步：

    - 预选Node：遍历集群中所有的Node，按照具体的预选策略筛选出符合要求的Node列表。如没有Node符合预选策略规则，该Pod就会被挂起，直到集群中出现符合要求的Node。
    - 优选Node：预选Node列表的基础上，按照优选策略为待选的Node进行打分和排序，从中获取最优Node。

  - Cluster State Store（etcd）：Cluster State Store代表集群的存储状态，存储在etcd中。etcd是一个简单的、分布式的、一致的key-value存储服务，主要被用来共享配置和服务发现。etcd提供了一个CRUD操作的REST API，以及提供了作为注册的接口，以监控指定的Node。集群的所有状态都存储在etcd实例中，并具有监控的能力，因此当etcd中的信息发生变化时，就能够快速的通知集群中相关的组件。
  - Controller-Manger Server：Controller-Manager Serve用于执行大部分的集群层次的功能，它既执行生命周期功能(例如：命名空间创建和生命周期、事件垃圾收集、已终止垃圾收集、级联删除垃圾收集、node垃圾收集)，也执行API业务逻辑（例如：pod的弹性扩容）。控制管理提供自愈能力、扩容、应用生命周期管理、服务发现、路由、服务绑定和提供。Kubernetes默认提供Replication Controller、Node Controller、Namespace Controller、Service Controller、Endpoints Controller、Persistent Controller、DaemonSet Controller等控制器。

- Worker Node：作为真正的工作节点，运行业务应用的容器；Worker Node包含kubelet、kube proxy和Container Runtime；
kubelet：Kubelet是Kubernetes中最主要的控制器，它是Pod和Node API的主要实现者，Kubelet负责驱动容器执行层。在Kubernetes中，应用容器彼此是隔离的，并且与运行其的主机也是隔离的，这是对应用进行独立解耦管理的关键点。在Kubernets中，Pod作为基本的执行单元，它可以拥有多个容器和存储数据卷，能够方便在每个容器中打包一个单一的应用，从而解耦了应用构建时和部署时的所关心的事项，已经能够方便在物理机/虚拟机之间进行迁移。
API准入控制可以拒绝Pod，或者为Pod添加额外的调度约束，但是Kubelet才是Pod是否能够运行在特定Node上的最终裁决者，而不是scheduler或者DaemonSet。

  - kubelet默认情况使用cAdvisor进行资源监控。负责管理Pod（包括其他k8s组件pod）、容器的生命周期、镜像、数据卷等，实现集群对节点的管理，并将容器的运行状态汇报给Kubernetes API Server。

  - kube proxy：基于一种公共访问策略（例如：负载均衡），服务提供了一种访问一群pod的途径。此方式通过创建一个虚拟的IP来实现，客户端能够访问此IP，并能够将请求代理发送至Pod。
  每一个Node都会运行一个kube-proxy，kube proxy通过iptables规则引导访问至服务IP，并将重定向至正确的后端应用，通过这种方式kube-proxy提供了一个高可用的负载均衡解决方案。
  服务发现主要通过DNS实现。
  在Kubernetes中，kube proxy负责为Pod创建代理服务；引到访问至服务；并实现请求服务到Pod的路由和转发，以及通过应用的负载均衡。

  - Container Runtime：每一个Node都会运行一个Container Runtime，其负责下载镜像和运行容器。Kubernetes本身并不提供容器运行时环境，但提供了接口，可以插入所选择的容器运行时环境。kubelet使用Unix socket之上的gRPC框架与容器运行时进行通信，kubelet作为客户端拉取镜像，而CRI shim作为服务器提供镜像。
  protocol buffers API提供两个gRPC服务，ImageService和RuntimeService。
  ImageService提供拉取、查看、和移除镜像的RPC。
  RuntimeSerivce则提供管理Pods和容器生命周期管理的RPC，以及与容器进行交互(exec/attach/port-forward)。容器运行时能够同时管理镜像和容器（例如：Docker和Rkt），并且可以通过同一个套接字提供这两种服务。
  在Kubelet中，这个套接字通过–container-runtime-endpoint和–image-service-endpoint字段进行设置。
  Kubernetes CRI支持的容器运行时包括docker、rkt、cri-o、frankti、kata-containers和clear-containers等。

- kubectl：用于通过命令行与主节点的API Server进行交互，而对Kubernetes进行操作，实现在集群中进行各种资源的增删改查等操作；
- web UI：与kubectl一样都是对API Server进行交互，而对Kubernetes进行操作，实现在集群中进行各种资源的增删改查等操作；
- CoreDNS插件：为Pod集群的SVC（svc是service的简写）创建一个域名ID的对应关系解析
- Dashboard插件：Kubernetes的仪表盘
- Ingress Controller插件：官方只能实现4层代理，而Ingress Controller插件可以实现7层代理
- Pederation插件：提供一个可以跨集群中心多K8s同一管理功能
- Prometheus插件：提供K8s集群的监控能力
- ELK插件：提供K8s集群日志统一分析接入平台
 