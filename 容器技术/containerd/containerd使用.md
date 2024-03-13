# 安装 containerd
- 官网releases版本选择安装：https://github.com/containerd/containerd/releases 注意系统和cpu架构选amd64，即x86架构
- Containerd 提供了两个压缩包，一个叫 containerd-${VERSION}.${OS}-${ARCH}.tar.gz，另一个叫 cri-containerd-${VERSION}.${OS}-${ARCH}.tar.gz。其中 cri-containerd-${VERSION}.${OS}-${ARCH}.tar.gz 包含了所有 Kubernetes 需要的二进制文件。如果你只是本地测试，可以选择前一个压缩包；如果是作为 Kubernetes 的容器运行时，需要选择后一个压缩包。Containerd 是需要调用 runc 的，而第一个压缩包是不包含 runc 二进制文件的，如果你选择第一个压缩包，还需要提前安装 runc。所以我建议直接使用 cri-containerd 压缩包。


### 使用containerd
- 镜像、容器、名称空间参考：https://www.cnblogs.com/yinzhengjie/p/18058010
- cni网络参考：https://cloud.tencent.com/developer/article/2308392
~~~shell
# 安装containerd，注意选择含有 cri 插件（Container Runtime Interface 提供容器运行时接口）和cni插件的安装包
wget https://github.com/containerd/containerd/releases/download/v1.7.13/cri-containerd-cni-1.7.13-linux-amd64.tar.gz

# 查看压缩文件解压后有那些文件
tar -tf cri-containerd-cni-1.7.13-linux-amd64.tar.gz

# 直接将压缩包解压到系统的各个目录中
sudo tar -C / -xzf cri-containerd-cni-1.7.13-linux-amd64.tar.gz

# 检查path环境变量，是否包含 /usr/local/bin 和 /usr/local/sbin ，不包含请添加到path环境变量中
echo $PATH
vim ~/.bashrc  # 编辑当前用户的环境变量，使其生效                   
export PATH=$PATH:/usr/local/bin:/usr/local/sbin
source ~/.bashrc

# 启动 containerd 守护进程
sudo systemctl start containerd

# 查看 containerd 守护进程状态
sudo systemctl status containerd

# 重启 containerd 守护进程
sudo systemctl restart containerd

# 设置开机启动 containerd 守护进程
sudo systemctl enable containerd

# 关闭开机启动 containerd 守护进程
sudo systemctl disable containerd

# Containerd 的默认配置文件为 /etc/containerd/config.toml，我们可以通过命令来生成一个默认的配置
sudo mkdir /etc/containerd
sudo containerd config default > /etc/containerd/config.toml

# 在国内拉取公共镜像仓库的速度是极慢的，为了节约拉取时间，需要为 Containerd 配置镜像仓库的 mirror。Containerd 的镜像仓库 mirror 与 Docker 相比有两个区别：
# Containerd 只支持通过 CRI 拉取镜像的 mirror，也就是说，只有通过 crictl 或者 Kubernetes 调用时 mirror 才会生效，通过 ctr 拉取是不会生效的。
# Docker 只支持为 Docker Hub 配置 mirror，而 Containerd 支持为任意镜像仓库配置 mirror。
# 镜像加速的配置就在 /etc/containerd/config.toml 中plugins配置块 plugins."io.containerd.grpc.v1.cri".registry

# 注意如下配置是用于数据存储的
# root = "/var/lib/containerd"   # root用来保存持久化数据，包括 Snapshots, Content, Metadata 以及各种插件的数据。每一个插件都有自己单独的目录，Containerd 本身不存储任何数据，它的所有功能都来自于已加载的插件
# state = "/run/containerd"      # state 用来保存临时数据，包括 sockets、pid、挂载点、运行时状态以及不需要持久化保存的插件数据

# Containerd 是容器的守护者，一旦发生内存不足的情况，理想的情况应该是先杀死容器，而不是杀死 Containerd 守护进程。所以需要调整 Containerd 的 OOM 权重，减少其被 OOM Kill 的几率。最好是将 oom_score 的值调整为比其他守护进程略低的值。这里的 oom_socre 其实对应的是 /proc/<pid>/oom_score_adj，在早期的 Linux 内核版本里使用 oom_adj 来调整权重, 后来改用 oom_score_adj 了
# 在计算最终的 badness score 时，会在计算结果是中加上 oom_score_adj ,这样用户就可以通过该在值来保护某个进程不被杀死或者每次都杀某个进程。其取值范围为 -1000 到 1000。如果将该值设置为 -1000，则进程永远不会被杀死，因为此时 badness score 永远返回0。建议 Containerd 将该值设置为 -999 到 0 之间。如果作为 Kubernetes 的 Worker 节点，可以考虑设置为 -999。
# oom_score = 0


# systemd 配置 Containerd 作为守护进程运行配置文件
sudo vim /etc/systemd/system/containerd.service
# 这里有两个重要的参数：
#    Delegate : 这个选项是否允许 Containerd 以及运行时自己管理自己创建的容器的 cgroups。如果不设置这个选项，systemd 就会将进程移到自己的 cgroups 中，从而导致 Containerd 无法正确获取容器的资源使用情况。
#    KillMode : 这个选项用来处理 Containerd 进程被杀死的方式。默认情况下，systemd 会在进程的 cgroup 中查找并杀死 Containerd 的所有子进程，这肯定不是我们想要的。KillMode字段可以设置的值如下。我们需要将 KillMode 的值设置为 process，这样可以确保升级或重启 Containerd 时不杀死现有的容器
#        control-group（默认值）：当前控制组里面的所有子进程，都会被杀掉
#        process：只杀主进程
#        mixed：主进程将收到 SIGTERM 信号，子进程收到 SIGKILL 信号
#        none：没有进程会被杀掉，只是执行服务的 stop 命令。


# 查看版本，单机的 containerd 是使用的 ctr 进行管理容器和镜像的，而k8s中的 containerd 是使用的 crictr 进行管理容器和镜像的
sudo ctr version
sudo crictr version

# 查看镜像，下面几个命令一样，images、image、i 的意思是一样的，list、ls的意思是一样的
sudo ctr images list
sudo ctr i ls
# -n [namespace] 查看指定名称空间下的镜像

# 检查镜像的完整性，确保镜像可以
sudo ctr i check [镜像]
# -n [namespace] 指定名称空间

# 导出镜像
sudo ctr i export [导出的镜像文件，例如test.tar.gz] [镜像]
# -n [namespace] 指定名称空间

# 导入镜像
sudo ctr i import [导入的镜像文件，例如test.tar.gz]
# -n [namespace] 指定名称空间，名称空间不存在会自动创建

# 拉取镜像，注意镜像名无法简写，需要包含仓库地址，并且必须先下载镜像才能运行容器，无法和docker一样运行容器时下载镜像
sudo ctr i pull [参数] [镜像]
# 默认会下载符号当前系统架构的镜像
# --platform [系统架构] 会下载指定系统架构的镜像           
# --all-platforms 会下载所有系统架构的镜像
# -n [namespace] 指定名称空间，名称空间不存在会自动创建

# 推送镜像
sudo ctr i push [镜像]
# -n [namespace] 指定名称空间

# 删除镜像，delete, del, remove, rm 都可以删除镜像
sudo ctr i rm [镜像]
# -n [namespace] 指定名称空间

# 给镜像加个tag，即换个名称，注意新镜像名无法简写，需要包含仓库地址
sudo ctr i tag [镜像] [新的tag镜像名称]
# -n [namespace] 指定名称空间

# 将镜像挂载到主机目录上防止丢失
sudo ctr i mount [镜像] [主机挂载目录]
# -n [namespace] 指定名称空间

# 解除挂载到主机上的镜像
sudo ctr i unmount [主机挂载目录]



# 创建容器，并且必须先下载镜像才能运行容器，无法和docker一样运行容器时下载镜像
# 注意这只是创建容器，并没有运行容器，类似于启动安装好环境的虚拟机，但是内部真正需要启动的程序还没有运行
sudo ctr c create [镜像] [容器名称]
# -n [namespace] 指定名称空间
# --with-ns "pid:/proc/[PID]/ns/pid" 与其他容器共享名称空间，PID是其他容器中运行的主要进程PID（通过 ctr tasks ls | grep [容器名称] | awk '{print $2}' 查看其他容器中的主要进程PID），即在容器内可以看到其他和访问指定容器内运行的进程
# --mount type=bind,src=[宿主机挂载目录],dst=[容器内挂载目录],options=rbind:rw 设置挂载数据卷到宿主机，挂载的类型为 bind，即把宿主机的一个文件或目录挂载到容器中，options指定了挂载的设置，rbind 是递归绑定挂载，意味着如果源是一个目录，它会递归地绑定目录和其子目录。rw 表示挂载为读写模式

# 启动容器，请先确保 sudo ls /usr/local/bin/containerd-shim-runc-v2 -l 或者在 sudo ls /usr/bin/containerd-shim-runc-v2 -l 下可以找到这个可执行程序
sudo ctr task start -d [容器名称]
# -d 参数表示后台运行容器

# 直接创建容器并且启动容器，即上述两步操作
sudo ctr run -d --net-host [镜像] [容器名称]
# -d 参数表示后台运行容器
# --net-host 参数表示容器于宿主机共享网络资源（ip、端口之类的共享），即docker 的host网络类型
# -n [namespace] 指定名称空间

# 查看容器列表（包含静态容器和动态容器），下面几个命令一样，containers、container、c 的意思是一样的，list、ls的意思是一样的
sudo ctr container list
sudo ctr c ls

# 查看任务task，也就是容器中运行的主要进程，tasks、task、t的意思是一样的，list、ls的意思是一样的
# 注意显示的进程PID是宿主机中的PID号
sudo ctr task list
sudo ctr t ls

# 查看指定容器内的进程在宿主机中PID号，可以进一步的通过ps aux | grep [PID] 查看这些进程的启动命令
sudo ctr t ps [容器名称]
# -n [namespace] 指定名称空间

# 获取指定的容器详细信息
sudo ctr c info [容器名称]
# -n [namespace] 指定名称空间

# 在容器中执行命令
sudo ctr t exec --exec-id [id] [容器名称] [命令]
# ID是执行命令使用的ID号，该ID号无法在容器中同时被两个程序使用，其实一般情况下ID号随机指定，即可以使用 $RANDOM
# 进入容器 sudo ctr t exec --exec-id 1 [容器名称] /bin/sh
# -n [namespace] 指定名称空间

# 暂停运行的容器，后续通过 ctr t ls 可以看见容器主进程状态是 PAUSED
sudo ctr t pause [容器名称]
# -n [namespace] 指定名称空间

# 恢复暂停的容器，后续通过 ctr t ls 可以看见容器主进程状态是 RUNNING
sudo ctr t resume [容器名称]
# -n [namespace] 指定名称空间

# 杀死运行容器中的进程，后续通过 ctr t ls 可以看见容器主进程状态是 STOPPED
sudo ctr t kill [容器名称]
# -n [namespace] 指定名称空间

# 删除容器中的进程，后续通过 ctr t ls 看不到该容器的主进程了
# 注意容器中进程状态必须是STOPPED，即容器中的进程被杀死，否则无法删除容器内的进程
sudo ctr t rm [容器名称]
# -n [namespace] 指定名称空间

# 删除容器，delete, del, remove, rm 都可以删除容器
# 注意容器中不能有任何的进程（无论是否删除），否则无法删除容器
sudo ctr c rm
# -n [namespace] 指定名称空间


# 查看 namespace 列表，下面几个命令一样，namespaces、namespace、ns 的意思是一样的，list、ls的意思是一样的
sudo ctr namespace list
sudo ctr ns ls
# 默认的名称空间是 default
# docker引擎使用 containerd 作为容器管理的 名称空间是 k8s.io
# Kubernetes 默认使用的是 containerd 的 k8s.io 命名空间

# 创建 namespace , create、c 都可以创建namespace 
sudo ctr ns create [名称空间名称]
sudo ctr ns c [名称空间名称]

# 删除 namespace , remove、rm 都可以删除namespace 
# 注意删除名称空间时该名称空间内确保没有镜像和容器，否则无法删除
sudo ctr ns rm [名称空间名称]




# containerd cni 插件安装，如果一开始安装containerd时就已经装好就无需再安装 containerd cni 插件
# 官网安装：https://github.com/containernetworking/plugins/releases/
# wget https://github.com/containernetworking/plugins/releases/download/v1.4.1/cni-plugins-linux-amd64-v1.4.1.tgz

# 下载cni工具源码包
# 官网安装：https://github.com/containernetworking/cni/releases/
# wget https://github.com/containernetworking/cni/archive/refs/tags/v1.1.2.tar.gz
# 解压已下载cni工具源码包
# tar xf v1.1.2.tar.gz
# 重命名已下载cni工具源码包目录
# mv cni-1.1.2 cni
# 查看cni工具目录中包含的文件
# ls cni
#   cnitool             CONTRIBUTING.md  DCO            go.mod  GOVERNANCE.md  LICENSE   MAINTAINERS  plugins    RELEASING.md  scripts  test.sh
#   CODE-OF-CONDUCT.md  CONVENTIONS.md   Documentation  go.sum  libcni         logo.png  pkg          README.md  ROADMAP.md


# 安装jq，下面由shell脚本会使用jq
sudo apt install jq -y 

# 准备CNI网络配置文件，，其中包含名为cni0的网桥
vim /etc/cni/net.d/10-mynet.conf
#   {
#     "cniVersion": "1.0.0",
#     "name": "mynet",              # 创建的网络名称
#     "type": "bridge",             # 网络模式是 bridge （桥接）
#     "bridge": "cni0",             # 网桥名称
#     "isGateway": true,            # 是否是网关
#     "ipMasq": true,
#     "ipam": {                     # ip集
#       "type": "host-local",       
#       "subnet": "10.66.0.0/16",   # 网段设置
#       "routes": [
#         { "dst": "0.0.0.0/0" }    # 路由设置
#      ]
#     }
#   }

vim /etc/cni/net.d/99-loopback.conf
#   {
#     "cniVerion": "1.0.0",
#     "name": "lo",                 # 创建的网络名称
#     "type": "loopback"            # 网络模式是 loopback （回环网卡）
#   }

# 进入之前下载的cni工具目录下的scripts目录中执行，需要依赖exec-plugins.sh文件
cd cni/scripts/ 

# 执行脚本文件，基于/etc/cni/net.d/目录中的*.conf配置文件生成容器网络
# /opt/cni/bin 是 containerd cni 插件安装目录下的bin目录
sudo CNI_PATH=/opt/cni/bin ./priv-net-run.sh echo "Hello World"

# 在宿主机上查看是否生成指定网桥
ifconfig

# 创建并启动nginx容器
sudo ctr c create docker.io/library/nginx:1.21.6-alpine c1
sudo ctr t start -d c1

# 在宿主机中完成指定容器进程ID获取
pid=$(sudo ctr tasks ls | grep busybox | awk '{print $2}')
echo $pid

# 在宿主机中完成指定容器网络命名空间路径
netnspath=/proc/$pid/ns/net
echo $netnspath

# 注意进入之前下载的cni工具目录下的scripts目录中执行
# 执行脚本文件为容器添加网络配置
# /opt/cni/bin 是 containerd cni 插件安装目录下的bin目录
sudo CNI_PATH=/opt/cni/bin ./exec-plugins.sh add $pid $netnspath

# 进入容器，可以看到分配的网卡和ip地址，ip地址是宿主机生成的网桥地址段中分配的
# 并且可以 ping 通宿主机的局域网，而宿主机也可以 ping 通容器分配的ip
sudo ctr t exec --exec-id $RANDOM c1 sh
ifconfig

# 宿主机通过容器IP去访问容器nginx服务
curl http://[容器IP]

~~~





