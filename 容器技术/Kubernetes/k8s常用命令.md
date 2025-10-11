1. 查看k8s集群节点
~~~shell
kubectl get node
~~~

2. 查看资源信息，查看指定pod的资源清单见第8条命令，查看pod的事件记录查看第10条命令，查看pod日志查看第11条
~~~shell
kubectl get [资源类型]
# 默认是查看的默认命名空间default下的资源
# k8s官方组件启动的资源在kube-system命名空间下
# 查看其他的命名空间加参数                                            -n "命名空间的名字"
# 查看所有的命名空间下的资源添加参数​                                 --all-namespaces
# 查看资源对应的标签加参数                                            --show-labels
# 筛选查看出带指定标签key或者是key=value的资源                        -l "标签的key或者是key=value"
# 查看pod的IP和所在的工作节点添加参数                                 -o wide
# 监控查看pod，资源发送变化时会打印变化                               -w 或者是 -watch  
# 如果需要查看多种资源则需要使用逗号分隔资源类型例如查看pod和svc使用    kubectl get pod,svc
~~~

3. 查看命名空间列表
~~~shell
# 查看所有的命名空间
kubectl get ns
~~~

4. 获取配置清单中apiVersion的可以指定所有版本
~~~shell
kubectl api-versions
~~~
 
5. 获取对应资源的apiVersion的版本，如下是获取pod的apiVersion版本，显示的结果是v1、也可以用于获取其他的资源apiVersion版本、如 Ingress 等
~~~shell
kubectl explain pod
~~~
 
6. 查看对应资源的元数据（metadata）有哪些属性可以设置，如下是查看pod的元数据（metadata）有哪些可以设置的
~~~shell
kubectl explain pod.metadata
~~~
 

7. 查看对应资源期望的状态（spec）有哪些属性可以设置
~~~shell
kubectl explain pod.spec 
~~~ 

8. 查看指定pod的资源清单并输出到pod.yaml文件

~~~shell
kubectl get pod -n [命名空间] [pod的name] -o yaml > [/root/pod.yaml]
# -n 参数指定命名空间，没有指定则是default命名空间
# -o 以指定的格式输出pod资源清单的文件，现在是yaml 也可以以json输出
# > 重定向输入指定文件中
~~~
 
9. 手动根据资源清单创建资源

~~~shell
kubectl create -n [命名空间] -f pod.yaml
# kubectl create 是用于pod、rc、rs资源的创建
# -n 参数指定资源创建在那个命名空间下
# -f 参数指定资源清单文件

kubectl apply -n [命名空间] -f [deployment.yaml] --record=true
# kubectl apply 是用于Deployment资源的创建，--record=true记录当前命令到Deployment控制器的版本历史中，--record=true在后续的更新中不要中途漏掉一次否则下一次记录的是上一次的命令
# -n 参数指定资源创建在那个命名空间下
# -f 参数指定资源清单文件
~~~
 

10. 查看资源的事件记录

~~~shell
kubectl describe -n [命名空间] [资源的类型] [资源的名称]
# 查看pod的事件记录     kubectl describe pod "pod的名称"
~~~
 

11. 查看pod的日志

~~~shell
kubectl logs -n [命名空间] [pod的名称]
~~~
 

12. 删除资源，不要删除svc类型下kubernetes名称的svc，这个svc被很多的kubernetes组件和插件使用，不能删除否则环境崩塌，更不要使用--all删除全部svc类型的资源

~~~shell
kubectl delete -n [命名空间] [资源的类型] [资源的名称]
# 删除非控制器控制的pod                         kubectl delete -n [命名空间] pod [pod的名称]
# 删除控制器控制的pod只需要删除控制器            kubectl delete -n [命名空间] [控制器的类型] [控制器的名称]
# 删除命名空间下的某一类型的所有资源(svc类型下不要使用--all删除)      kubectl delete -n [命名空间] [资源的类型] --all  
~~~
 

13. 进入容器内部

~~~shell
kubectl exec -it -n [命名空间] [pod的名称] -c [容器的名称] -- /bin/sh
# 如果pod中只有一个容器就不需要添加参数 -c "容器的名称"
~~~
 

14. 修改资源的标签，不要随便修改标签，修改pod的标签可能会导致pod不归控制器管理，控制器会认为Pod数量缺少，创建新的Pod并且不清理原来修改标签的Pod，所以修改标签需要谨慎
~~~shell
kubectl label -n [命名空间] [资源类型] [资源名称] app=myapp --overwrite
# app=myapp     修改的标签key和value
# --overwrite   允许修改标签
~~~
 

15. 修改控制器控制的Pod副本数量

~~~shell
kubectl scale -n [命名空间] --replicas=3 [控制器类型]/[控制器的名称]
# 控制器类型与控制器名称之间的/可以使用空格替换
# 一般情况下控制器的资源清单不写Pod的名称，
# 所以RS控制器类型的Pod名称就是RS控制器名称加pod的MD5值
# deployment控制器类型的pod名称是Deployment控制器的名称加RS控制器的名称加Pod的MD5值
# Pod副本的数量扩容和缩容不会创建新的RS控制器
~~~
 

16. 更新Deployment控制器控制的Pod容器镜像

~~~shell
kubectl set image deployment -n [命名空间] [deployment控制器的名称] 'Pod中的容器名'='镜像名称':'镜像版本' --record=true
# --record=true记录当前命令到Deployment控制器的版本历史中，方便查看和回滚
# --record=true在后续的更新中不要中途漏掉一次否则下一次记录的是上一次的命令
~~~
 

17. 回滚Deployment控制器

方式一：根据缓存在etcd里的RS版本回滚
~~~shell
kubectl rollout undo -n [命名空间] deployment/[deployment控制器的名称]
# 注意仅仅只是回退到上一版本，比如从4版本回退到3版本，如果在3版本上回退是不会回退到2版本而是进入4版本，如果需要回退到更老的版本则需要指定版本

kubectl rollout history -n [命名空间] deployment/[deployment控制器的名称] 
# 查看Deployment控制器的历史版本和创建更新命令记录，历史版本数量在资源清单不写是默认保留10个

kubectl rollout undo -n [命名空间] deployment/[deployment控制器的名称] --to-version=2 
# --to-version=2 回退到指定的历史版本2

~~~
方式二：根据保存在本地的历史资源清单文件回滚
~~~shell
kubectl apply -f [保存在本地的历史资源清单]
# 注意资源的名称不要发送变化
~~~
方式三：参考第18条命令

方式四：参考第19条命令

18. 通过编辑已经创建的资源缓存在etc的资源清单来更新资源

~~~shell
kubectl edit -n [命名空间] [资源类型] [资源名称]
# 编辑保存退出就更新了，如果退出不了则代表有些资源属性不允许修改
~~~
 
19. 通过命令对已经创建的资源的资源清单打补丁修改来更新资源

~~~shell
kubectl patch -n [命名空间] [资源类型] [资源名称] -p [需要修改的资源清单属性（是json格式而不是yaml）]
# 例如修改Deployment控制器控制的Pod镜像
kubectl patch deployment [deployment控制器的名称] -p '{"spec": {"template": {"spec": {"containers": [{"name": "web","image": "镜像名称:镜像版本"}]}}}}'
~~~
 

20. 金丝雀部署
~~~shell
kubectl patch deployment [deployment控制器的名称] -p '{"spec": {"strategy": {"rollingUpdate": {"maxSurge": "1","maxUnavailable": "0"}}}}'
# 允许控制器控制的Pod副本数量在更新的时候可以多一个，但是不允许在更新的时候可用Pod副本的数量少一个

kubectl set image deployment [deployment控制器的名称] 'Pod中的容器名'='镜像名称':'镜像版本' && kubectl rollout pause deployment [deployment控制器的名称]
# 进行更新整个控制器管理的Pod，但是由于设置了更新时允许可用Pod数量多一个但是一个都不允许少，所以控制器管理的Pod会多出一个并且多出来的哪个Pod是更新后的新版本而其他的Pod是旧版本，此时立刻停止更新。现在可以使用新版本的Pod进行测试新的版本是否稳定没有bug

kubectl rollout resume deployment [deployment控制器的名称]
# 若没有bug则进行全部Pod的更新

kubectl rollout status deployment [deployment控制器的名称]
# 查看是否更新成功

echo $?
# 通过判断上一条命令是否执行成功来判断是否更新成功，返回0代表成功

~~~
 

21. 滚动重启
~~~shell
# <deployment-name> 是需要重启的deployment
kubectl rollout restart deployment/[deployment-name]
~~~
 
22. 常见资源简写和版本查看
~~~shell
kubectl api-resources
# 列出 Kubernetes API 中所有可用的资源类型。命令的输出包含以下几列：
# `NAME`：资源类型的名称，例如 `pods`、`services`、`deployments` 等
# `SHORTNAMES`：资源类型的简写，例如 `po`（pods）、`svc`（services）、`deploy`（deployments）等
# `APIGROUP`：资源所属的 API 组。API 组是 Kubernetes API 的一个子集，例如 `apps`、`batch`、`extensions` 等
# `NAMESPACED`：这个列表示资源是否是命名空间级别的。如果是，那么你在创建、查看或删除这种资源时需要指定命名空间
# `KIND`：这是资源对象的种类，例如 `Pod`、`Service`、`Deployment` 等
# `VERBS`：这个列列出了你可以对这种资源执行的操作，例如 `get`、`list`、`create`、`delete` 等
~~~

23. 查看kubectl连接集群使用的config配置
~~~shell
kubectl config view
# 默认kubectl会读取~/.kube/config连接集群调用ApiServer
# kubectl config view 显示配置为空就需要按照以下方式获取配置

# k3s 集群（最常见）k3s 的默认配置文件路径为 /etc/rancher/k3s/k3s.yaml，复制到 ~/.kube/config 即可：
# 复制配置文件
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
# 修复权限（确保当前用户可读取）
sudo chown $USER:$USER ~/.kube/config
chmod 600 ~/.kube/config  # 安全权限，避免其他用户读取


#rke2 集群配置文件路径为 /etc/rancher/rke2/rke2.yaml，操作同上：
sudo cp /etc/rancher/rke2/rke2.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config


# 原生 Kubernetes（kubeadm）配置文件通常在 /etc/kubernetes/admin.conf：
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
~~~

24. kubeadm安装的集群使用kubectl连接不上报509证书过期
~~~shell
# kubeadm安装的集群证书和etcd的证书存储在 /etc/kubernetes/pki 目录下，这些证书的有效期默认是1年，根证书的有效期默认是10年
# 查看当前集群证书的有效期
kubeadm certs check-expiration

# 提前备份集群证书
cp -rf /etc/kubernetes/ /etc/kubernetes.bak

# 提前备份etcd数据目录
cp -rf /var/lib/etcd /var/lib/etcd.bak 

# 集群证书的有效期更新续期1年
kubeadm certs renew all

# 更新完证书需要重启 apiServer、controller-manager、scheduler、etcd，这些组件都在kube-system命名空间下 kubectl get pods -n kube-system 
kubectl delete pod -n kube-system [apiServer的pod名称] [controller-manager的pod名称] [scheduler的pod名称] [etcd的pod名称]

# 查看组件启动状态 
kubectl get pods -n kube-system 

# 再查看当前集群证书的有效期可以看到续期到一年了
kubeadm certs check-expiration
~~~
