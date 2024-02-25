# 能力（capabilities）是什么？
为了执行权限检查，Linux 区分两类进程：特权进程(其有效用户标识为 0，也就是超级用户 root)和非特权进程(其有效用户标识为非零)。 特权进程绕过所有内核权限检查，而非特权进程则根据进程凭证(通常为有效 UID，有效 GID 和补充组列表)进行完全权限检查。

以常用的 passwd 命令为例，修改用户密码需要具有 root 权限，而普通用户是没有这个权限的。但是实际上普通用户又可以修改自己的密码，这是怎么回事？在 Linux 的权限控制机制中，有一类比较特殊的权限设置，比如 SUID(Set User ID on execution)，因为程序文件 /bin/passwd 被设置了 SUID 标识，所以普通用户在执行 passwd 命令时，进程是以 passwd 的所有者，也就是 root 用户的身份运行，从而修改密码。

SUID 虽然可以解决问题，却带来了安全隐患。当运行设置了 SUID 的命令时，通常只是需要很小一部分的特权，但是 SUID 给了它 root 具有的全部权限。因此一旦 被设置了 SUID 的命令出现漏洞，就很容易被利用。也就是说 SUID 机制在增大了系统的安全攻击面。

Linux 引入了 capabilities 机制对 root 权限进行细粒度的控制，实现按需授权，从而减小系统的安全攻击面。本文将介绍 capabilites 机制的基本概念和用法。

从内核 2.2 开始，Linux 将传统上与超级用户 root 关联的特权划分为不同的单元，称为 capabilites。Capabilites 作为线程(Linux 并不真正区分进程和线程)的属性存在，每个单元可以独立启用和禁用。如此一来，权限检查的过程就变成了：在执行特权操作时，如果进程的有效身份不是 root，就去检查是否具有该特权操作所对应的 capabilites，并以此决定是否可以进行该特权操作。比如要向进程发送信号(kill())，就得具有 capability CAP_KILL；如果设置系统时间，就得具有 capability CAP_SYS_TIME。

# 查看可以设置那些能力以及拥有的能力、设置能力(设置和查看文件的能力需要root权限)
~~~shell
# 查看可以设置那些特权能力给普通用户
man capabilities
# 以下是通过man capabilities 查看的常见能力名称和描述
#   CAP_AUDIT_CONTROL	启用和禁用内核审计；改变审计过滤规则；检索审计状态和过滤规则
#   CAP_AUDIT_READ	允许通过 multicast netlink 套接字读取审计日志
#   CAP_AUDIT_WRITE	将记录写入内核审计日志
#   CAP_BLOCK_SUSPEND	使用可以阻止系统挂起的特性
#   CAP_CHOWN	修改文件所有者的权限
#   CAP_DAC_OVERRIDE	忽略文件的 DAC 访问限制，即忽略文件的读写执行权限，即使没有权限也可以操作文件
#   CAP_DAC_READ_SEARCH	忽略文件读及目录搜索的 DAC 访问限制，即忽略文件的读权限，即使没有权限也可以读文件
#   CAP_FOWNER	忽略文件属主 ID 必须和进程用户 ID 相匹配的限制
#   CAP_FSETID	允许设置文件的 setuid 位
#   CAP_IPC_LOCK	允许锁定共享内存片段
#   CAP_IPC_OWNER	忽略 IPC 所有权检查
#   CAP_KILL	允许对不属于自己的进程发送信号
#   CAP_LEASE	允许修改文件锁的 FL_LEASE 标志
#   CAP_LINUX_IMMUTABLE	允许修改文件的 IMMUTABLE 和 APPEND 属性标志
#   CAP_MAC_ADMIN	允许 MAC 配置或状态更改
#   CAP_MAC_OVERRIDE	覆盖 MAC(Mandatory Access Control)
#   CAP_MKNOD	允许使用 mknod() 系统调用
#   CAP_NET_ADMIN	允许执行网络管理任务
#   CAP_NET_BIND_SERVICE	允许绑定其他服务到小于 1024 的系统服务专用端口
#   CAP_NET_BROADCAST	允许网络广播和多播访问
#   CAP_NET_RAW	允许使用原始套接字
#   CAP_SETGID	允许改变进程的 GID
#   CAP_SETFCAP	允许为文件设置任意的 capabilities
#   CAP_SETPCAP	参考 capabilities man page
#   CAP_SETUID	允许改变进程的 UID
#   CAP_SYS_ADMIN	允许执行系统管理任务，如加载或卸载文件系统、设置磁盘配额等
#   CAP_SYS_BOOT	允许重新启动系统
#   CAP_SYS_CHROOT	允许使用 chroot() 系统调用
#   CAP_SYS_MODULE	允许插入和删除内核模块
#   CAP_SYS_NICE	允许提升优先级及设置其他进程的优先级
#   CAP_SYS_PACCT	允许执行进程的 BSD 式审计
#   CAP_SYS_PTRACE	允许跟踪任何进程
#   CAP_SYS_RAWIO	允许直接访问 /devport、/dev/mem、/dev/kmem 及原始块设备
#   CAP_SYS_RESOURCE	忽略资源限制
#   CAP_SYS_TIME	允许改变系统时钟
#   CAP_SYS_TTY_CONFIG	允许配置 TTY 设备
#   CAP_SYSLOG	允许使用 syslog() 系统调用
#   CAP_WAKE_ALARM	允许触发一些能唤醒系统的东西(比如 CLOCK_BOOTTIME_ALARM 计时器)


# getcap 命令查看文件的 capabilities 属性（需要root权限可以查看）
sudo getcap [文件路径]
# 例如 getcap /bin/ping 查看ping的capabilities 属性

# setcap 命令用来设置程序文件的 capabilities 属性（添加、减少、等于那些capabilities属性）
sudo setcap [需要操作的 capabilities 属性+-集合] [文件路径]
# -r 参数可以清除掉文件的所有 capabilities 属性
# 例子1：先去除/bin/ping的SUID权限，这样其他用户无法使用ping命令了，再执行 sudo setcap cap_net_admin,cap_net_raw+ep /bin/ping 其他用户又获得了ping命令的执行权限
# 上述命令将允许执行网络任务和允许使用原始套接字添加到ep集合中，注意ep集合是e集合和p集合，下面会有集合的详细解释

# 例子2：执行 sudo setcap cap_chown=eip /bin/chown 其他用户获得了chown的执行权限，可以修改文件的所有者

# 例子3：执行 sudo setcap cap_dac_override=eip /bin/cat 其他用户又获得了cat查看文件时跳过文件的读写执行权限检查，可以直接查看文件内容
~~~


## 程序文件的 capabilities集合
在上面的示例中我们通过 setcap 命令修改了程序文件 /bin/ping 的 capabilities。在可执行文件的属性中有三个集合来保存三类 capabilities，它们分别是：
- Permitted：在程序文件执行时，Permitted 集合中的 capabilites 自动被加入到进程的 Permitted 集合中。
- Inheritable：Inheritable 集合中的 capabilites 会与进程的 Inheritable 集合执行与操作，以确定进程在执行 execve 函数后哪些 capabilites 被继承。
- Effective：Effective 只是一个 bit。如果设置为开启，那么在执行 execve 函数后，Permitted 集合中新增的 capabilities 会自动出现在进程的 Effective 集合中。
---

## 进程的 capabilities集合
进程中有五种 capabilities 集合类型，分别是：
- Permitted
- Inheritable
- Effective
- Bounding
- Ambient
### 进程的capabilities集合信息查看
/proc/[pid]/status 文件中包含了进程的五个 capabilities 集合的信息，我们可以通过下面的命名查看当前进程的 capabilities 信息：
~~~shell
# 查看pid是1的进程capabilities集合信息
cat /proc/1/status | grep 'Cap'
#   CapInh: 0000000000000000
#   CapPrm: 0000001fffffffff
#   CapEff: 0000001fffffffff
#   CapBnd: 0000001fffffffff
#   CapAmb: 0000000000000000

#但是这中方式获得的信息无法阅读，我们需要使用 capsh 命令把它们转义为可读的格式：
capsh --decode=0000001fffffffff
~~~