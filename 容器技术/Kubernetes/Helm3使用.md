### Helm的本质是让k8s的应用管理（Deployment，SVC等）可配置，能动态生成资源清单文件，然后调用Kubectl自动创建K8s资源简化部署和管理

### Helm是官方提供的类似于YUM的包管理器，是部署环境的流程封装。

# Helm有两个重要的概念：chart 和 release

- chart是创建一个应用的信息集合（类似于一个镜像），包含各种K8s资源文件的打包。chart是应用部署的自包含逻辑单元，可以将chart想象成apt、yum中的软件安装包  
- release是chart的运行实例（类似于容器），代表了一个正在运行的应用。每当chart被安装到k8s集群，就生成一个release，并且release可以进行版本管理

Helm客户端：helm的客户端组件，通过heml客户端的组件可以和K8s apiserver通信  
Helm仓库：类似于Docker仓库一样，Helm也可以搭建私有仓库或者使用公有仓库

~~~shell
# 安装helm
# 参考https://blog.csdn.net/qq_41586875/article/details/120930760  注意与k8s的版本对应
# 选择一下版本
wget https://get.helm.sh/helm-v3.5.4-linux-amd64.tar.gz
wget https://get.helm.sh/helm-v3.5.2-linux-amd64.tar.gz
wget https://get.helm.sh/helm-v3.7.1-linux-amd64.tar.gz

tar -zxf helm-v3.7.1-linux-amd64.tar.gz 

mv linux-amd64/helm /usr/bin

rm -rf linux-amd64/

helm version

# 添加、列出、移除、更新、索引chart仓库
helm repo [add|list|remove|update|index] [自定义仓库命名] [仓库地址]
helm repo list
helm repo remove [自定义仓库命名]
helm repo update
helm repo index

# 根据关键字搜索chart包
helm search repo [搜索内容]
# 查看chart包的基本信息和详细信息，可用
helm show [all|chart|readme|values] [chart包名]
# 拉取远程仓库的chart包并解压缩到本地，--version指定版本--untar是解压缩
helm pull [chart包名] [--version] [--untar]
# 创建一个mychart，会在当前目录下创建一个mychart目录
helm create mychart
# 启动mychart的release，注意mychart目录可能是一个本地目录、本地压缩包、下载mychart包或目录的网络地址
helm install [自定义的release名称] [mychart目录]
# 显示release实例名的状态，显示已命名版本的状态
helm status [release名称]
# 查看已经创建的release实例
helm list
# 下载一个release
helm get manifest [自定义的release名称]
# 关闭mychart的release
helm uninstall [自定义的release名称]
# 不真正的运行只是测试是否可以运行
helm install [自定义的release名称] [mychart目录] --debug --dry-run
# 更新一个release实例
helm upgrade
# 回滚release实例
helm rollback
# 获取release实例的历史
helm history [release实例名]
# 将chart目录打包成chart存档文件
helm package [chart目录路径]
~~~
---

helm创建的chart目录
~~~
mychart/                        // chart包名称
├── charts                      // 存放子chart的目录
├── Chart.yaml                  // 保存chart的基本信息，这个变量文件可以被templates下的文件引用
├── templates                   // 模板文件目录，目录下是基本的资源模板文件
│   ├── deployment.yaml         // 存放deployment资源对象的文件
│   ├── _helpers.tpl            // 放置模板助手的文件，存放了一些templates可能会用到的模板，可以在整个chart中重复使用
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt               // 存放提示信息的文件，介绍chart帮助信息，helm install后展示给用户的提示信息
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests                   // 用于测试文件，如部署完一个web应用后根据链接发起请求看是否部署成功
│       └── test-connection.yaml
└── values.yaml                 // 用于模板渲染文件，定义templates目录下的yaml文件可能引用到的变量
~~~
---

内置对象

~~~
通过内置对象获取的信息引用需要使用{{}}包裹

.Release对象描述了版本发布自身的一些信息，它包含以下对象：
.Release.Name         表示release的名称
.Release.Namespace    表示release的命名空间
.Release.IsUpgrade    如果当前操作是升级或回滚的话，该值为true
.Release.IsInstall    如果当前操作是安装的话，该值为true
.Release.Revision     获取此次修订的版本号。初次安装时为1，每次升级或回滚都会递增
.Release.Service      获取渲染当前模板的服务为名称。一般都是Helm


.Values对象描述了value.yaml文件中内容，默认为空，使用Value对象可以获取到value.yaml文件中已定义的任何变量值
# 例如value.yaml文件内容如下
name1: test1
info:
  name2: test2
# 获取值的方式如下
.Value.name1
.Value.info.name2


.Chart对象用于获取Chart.yaml文件中的内容
.Chart.Name      获取Chart的名称
.Chart.Version   获取Chart的版本


.Capabilities对象提供了管理kubernetes集群相关的信息
.Capabilities.APIVersions                 返回kubernetes集群API版本信息集合
.Capabilities.APIVersions.Has.$version    用于检测指定的版本或资源在k8s集群中是否可用，例如apps/v1/Deployment
.Capabilities.KubeVersion和.Capabilities.KubeVersion.Version    用于获取k8s的版本号
.Capabilities.KubeVersion.Major             用于获取k8s的主版本号
.Capabilities.KubeVersion.Minor             用于获取k8s的小版本号


.Template对象用于获取当前模板的信息，它包含以下两个对象
.Template.Name      用于获取当前模板的名称和路径 mychart/templates/mytemplate.yaml
.Template.BasePath  用于获取当前模板的路径，例如 mychat/templates

~~~
 

 

 

 

 