### 用于配置集群访问信息的文件叫作 kubeconfig 文件，在开启了 TLS 的集群中，每次与集群交互时都需要身份认证，生产环境一般使用证书进行认证，其认证所需要的信息会放在 kubeconfig 文件中。此外，K8s 的组件都可以使用 kubeconfig 连接 apiserver，client-go 、operator、helm 等其他组件也使用 kubeconfig 访问 apiserver。

### 使用kubeconfig文件来组织有关集群、用户、命名空间和身份认证机制的信息。kubectl 命令行根据使用kubeconfig文件来查找选择集群所需的信息，并与集群的API服务进行通信。

# kubectl是操作k8s的一个客户端工具，只要为kubectl提供链接apiserver的配置(kubeconfig)，kubectl可以在任何地方操作该集群。
### 默认情况下，kubectl 在 $HOME/.kube 目录下查找名为config的文件，可以通过设置KUBECONFIG环境变量或设置--kubeconfig参数来指定其他Kubeconfig文件。
### kubectl加载配置文件的顺序：  
1. kubectl 默认连接本机的 8080 端口
2. 从 $HOME/.kube 目录下查找文件名为 config 的文件
3. 通过设置环境变量 KUBECONFIG 或者通过设置去指定其它 kubeconfig 文件
4. 通过kubectl --kubeconfig=kubeconfig.yaml 命令参数指定kubeconfig 文件

## 参考：https://developer.aliyun.com/article/1069902
## 参考：https://www.cnblogs.com/securitybob/p/13964534.html

## 外部通过公网IP的kubeconfig连接集群，参考：https://blog.csdn.net/DANTE54/article/details/105297228
## 注意最终的kubeconfig，如果是在内网使用就修改配置的apiserveer为内网地址