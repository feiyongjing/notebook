# centOS6管理服务
~~~shell
# centOS6管理服务
# 查看npm包安装的服务
chkconfig --list
# 显示如下
netconsole      0:off   1:off   2:off   3:off   4:off   5:off   6:off
network         0:off   1:off   2:on    3:on    4:on    5:on    6:off
# 0到6分别代表系统的运行级别，off和on表示在该系统运行级别是否自启动
# 0表示系统关机
# 1表示单用户模式，该模式只启动最少最核心的服务，这个模式是用来做修复使用的
# 2表示不完全多用户，该模式没有NFS服务并且没有网络的命令行
# 3表示完全多用户，该模式是完整的命令行，我们一般操作的都是这个运行级别，系统开机会去读/etc/inittab，而其中默认配置的运行级别就是3，当然在有些系统中会在/etc/inittab标识不用配置修改运行级别而是使用systemctl get-default和systemctl set-default target来查看和修改运行级别
# 4这个级别未分配
# 5表示图像界面
# 6表示重启系统

# 查看系统的运行级别
runlevel
# 一般显示的是 N 3 其中N代表null,开机就进入现在的系统运行级别，如果不是N而是数字表示前一个系统的运行级别，3代表当前的运行级别

# 改变运行级别
init [运行级别数字]

# rpm的服务分两种，第一种是独立服务，第二种是xinetd服务
# 独立服务是独立处理单一需求的服务，mysql是独立服务
# xinetd服务能够同时监听多个指定的端口，在接受用户请求时，它能够根据用户请求的端口的不同，启动不同的网络服务进程来处理这些用户请求。可以把xinetd看做一个管理启动服务的管理服务器，它决定把一个客户请求交给哪个程序处理，然后启动相应的守护进程。xinetd无时不在运行并监听它所管理的所有端口上的服务。当某个要连接它管理的某项服务的请求到达时，xinetd就会为该服务启动合适的服务器。但是xinetd服务比较少见所以就不多详细记录了，有兴趣可以参考一下 http://c.biancheng.net/view/1054.html 进行xinetd服务的启动管理
# 一般独立服务和xinetd服务的默认配置如下
# 独立服务的启动脚本在/etc/rc.d/init.d/目录下，而xinetd服务的启动脚本在/etc/xinetd.d/目录下
# 独立服务的配置文件在/etc/目录下，而xinetd服务的配置文件在/etc/xinetd.conf/目录下
# 初始化环境配置文件位置在/etc/sysconfig/目录下
# 服务产生的数据放在/var/lib/目录下
# 服务产生的日志放在/var/log/目录下

# 操作rpm软件包安装的独立服务，或者使用绝对路径/etc/rc.d/init.d/服务名称作为执行命令，并且它们的参数都一致 
service [软件名] [参数]
# status参数表示显示软件服务的状态
# start参数表示启动软件服务，其实一般是去/etc/rc.d/init.d/（/etc/init.d/是它的软链接）目录下查找对应rpm安装软件的启动脚本并执行这个脚本
# restart参数表示重启软件服务
# stop参数表示关闭软件服务

# 源码包的启动方式，当然也可以将源码包的执行脚本的路径作为原文件在/etc/rc.d/init.d/目录下创建它的软链接再使用和rpm包启动一样的service命令启动源码包软件
源码包的执行脚本的路径 [参数]
# status参数表示显示软件服务的状态
# start参数表示启动软件服务
# restart参数表示重启软件服务
# stop参数表示关闭软件服务

# 设置服务的系统开机自启动
# 修改/etc/rc.d/rc.local文件或者它的软连接/etc/rc.local，在里面直接添加独立服务的启动命令就行了
# 比如在/etc/rc.local有默认的touch /var/lock/subsys/local这条命令，用于每次启动系统都更新/var/lock/subsys/local文件的修改时间，而以该文件的修改时间来作为系统的最后一次开机时间
~~~
---

# centOS7管理服务
~~~shell
# centOS7管理服务
# Systemd 是一组命令，涉及到系统管理的方方面面

# 查看系统启动耗时
systemd-analyze                                                                                       

# 查看每个服务的启动耗时
systemd-analyze blame

# 查看指定服务启动过程
systemd-analyze critical-chain [服务名称]
# 例如mysql 查看 systemd-analyze critical-chain mysqld.service


# Systemd 可以管理所有系统资源。不同的资源统称为 Unit（单位）
# Unit 一共分成12种。如下
# Service unit：系统服务，比如mysql、influxdb等服务
# Target unit：多个 Unit 构成的一个组
# Device Unit：硬件设备
# Mount Unit：文件系统的挂载点
# Automount Unit：自动挂载点
# Path Unit：文件或路径
# Scope Unit：不是由 Systemd 启动的外部进程
# Slice Unit：进程组
# Snapshot Unit：Systemd 快照，可以切回某个快照
# Socket Unit：进程间通信的 socket
# Swap Unit：swap 文件
# Timer Unit：定时器

# 列出正在运行的 Unit
systemctl list-units [参数]
# 不加参数列出的是运行的 Unit的信息
# 显示的列有UNIT、LOAD、ACTIVE、SUB、DESCRIPTION
# UNIT列是单元名称，后缀代表不同的单元类型
# LOAD列表示单元是否正确加载
# ACTIVE列表示高级单元激活状态，即SUB的泛化。
# SUB列表示低级单位激活状态，值取决于单位类型。
# DESCRIPTION表示单元的描述
# systemctl list-units --all列出所有Unit，包括没有找到配置文件的或者启动失败的
# systemctl list-units --all --state=inactive列出所有没有运行的 Unit
# systemctl list-units --type=service列出所有正在运行的、类型为 service 的 Unit

# 显示单个 Unit 的状态
sysystemctl status [unit的名称]
# 一般用于显示系统服务，比如mysql、influxdb等服务的状态。例如systemctl status mysqld，不加后缀名sysystemctl会默认后缀名为.service
# 除了status可以看到Unit 的状态还有以下3个命令可以在shell脚本中方便判断Unit 的状态
# 显示某个 Unit 是否正在运行
systemctl is-active [unit的名称]
# 显示某个 Unit 是否处于启动失败状态
systemctl is-failed [unit的名称]
# 显示某个 Unit 服务是否建立了启动链接，即是否设置了开机自启动
systemctl is-enabled [unit的名称]

# 立即启动、停止、重启、杀死、开机自启动、撤销开机自启动一个服务，不加后缀名sysystemctl会默认后缀名为.service
systemctl [参数] [服务名称]
# start参数启动服务
# stop参数停止服务
# restart参数重启服务
# kill参数杀死服务与该服务的子进程
# enable参数激活开机启动
# disable参数撤销开机启动

# 每一个 Unit 都有一个配置文件，告诉 Systemd 怎么启动这个 Unit 
# /etc/systemd/system/                               — 这是自定义服务单元文件的推荐位置。文件放在这里后，系统启动时会加载这些单元。
# /lib/systemd/system/ 或 /usr/lib/systemd/system/   — 这是系统安装时默认的服务单元文件目录，通常不要在这里放置自定义服务单元。
# Systemd 默认从上述目录读取原始配置文件 
# 而从/etc/systemd/system/multi-user.target.wants/读取自启动配置文件，但是，里面存放的大部分系统安装时默认的服务单元文件都是符号链接，指向原始配置文件
systemctl cat docker  # 查看服务的Unit 配置文件内容和配置文件目录


# 而systemctl enable命令用于在上面两个目录之间，建立符号链接关系。所有下面第一条命令等同与第二命令
systemctl enable xxx.service   # 不加后缀名sysystemctl会默认后缀名为.service
ln -s '/etc/systemd/system/xxx.service' '/etc/systemd/system/xxx.target.wants/xxx.service'
# 如果配置文件里面设置了开机启动，systemctl enable命令相当于激活开机启动。
# 与之对应的，systemctl disable命令用于在两个目录之间，撤销符号链接关系，相当于撤销开机启动。

# 查看Unit的依赖关系，即一个Unit A依赖于Unit B，就意味着 Systemd 在启动 A 的时候，同时会去启动 B
systemctl list-dependencies [unit的名称]
# 上面命令的输出结果之中，有些Unit的依赖是 Target 类型Unit，默认不会展开显示。如果要展开 Target，就需要使用--all参数

# 列出所有Unit的配置文件
systemctl list-unit-files
# --type参数列出指定类型的配置文件，例如systemctl list-unit-files --type=service
# 显示的结果有两列，UNIT FILE显示Unit的配置文件名和STATE配置文件的状态。配置文件的状态有enabled：已建立启动链接即设置为开机自启动、disabled：没建立启动链接即没有设置为开机自启动、static：该配置文件没有[Install]部分（无法执行），只能作为其他配置文件的依赖、masked：该配置文件被禁止建立启动链接即禁止设置为开机自启动
# 注意，从配置文件的状态无法看出，该 Unit 是否正在运行。这必须执行前面提到的systemctl status命令
# 一旦修改配置文件，就要让 SystemD 重新加载配置文件，然后重新启动，否则修改不会生效。
# 重新加载一个服务的配置文件
systemctl reload [服务名称]
# 重载所有修改过的配置文件
systemctl daemon-reload
# 重启一个服务
systemctl restart [服务名称]

# 配置文件就是普通的文本文件，可以在/usr/lib/systemd/system/或者是它的软链接目录下找到对应的配置文件直接用文本编辑器打开。或者是如下命令查看
systemctl cat [配置文件名称]
# 同样配置文件名称不加后缀名sysystemctl会默认后缀名为.service
# 配置文件分成几个区块。每个区块的第一行，是用方括号表示的区别名，比如[Unit]。注意，配置文件的区块名和字段名，都是大小写敏感的。
# 每个区块内部是一些等号连接的键值对。
# 注意，键值对的等号两侧不能有空格。
# [Unit]区块通常是配置文件的第一个区块，用来定义 Unit 的元数据，以及配置与其他 Unit 的关系。它的主要字段如下。
##      Description：简短描述
##      Documentation：文档地址
##      Requires：当前 Unit 依赖的其他 Unit，如果它们没有运行，当前 Unit 会启动失败
##      Wants：与当前 Unit 配合的其他 Unit，如果它们没有运行，当前 Unit 不会启动失败
##      BindsTo：与Requires类似，它指定的 Unit 如果退出，会导致当前 Unit 停止运行
##      Before：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之后启动
##      After：如果该字段指定的 Unit 也要启动，那么必须在当前 Unit 之前启动
##      Conflicts：这里指定的 Unit 不能与当前 Unit 同时运行
##      Condition...：当前 Unit 运行必须满足的条件，否则不会运行
##      Assert...：当前 Unit 运行必须满足的条件，否则会报启动失败
# [Install]区块通常是配置文件的最后一个区块，用来定义如何启动，以及是否开机启动。它的主要字段如下。
##      WantedBy：它的值是一个或多个 Target，当前 Unit 激活时（enable）符号链接会放入/etc/systemd/system目录下面以 Target 名 + .wants后缀构成的子目录中
##      RequiredBy：它的值是一个或多个 Target，当前 Unit 激活时，符号链接会放入/etc/systemd/system目录下面以 Target 名 + .required后缀构成的子目录中
##      Alias：当前 Unit 可用于启动的别名
##      Also：当前 Unit 激活（enable）时，会被同时激活的其他 Unit
# [Service]区块用来 Service 的配置，只有 Service 类型的 Unit 才有这个区块。它的主要字段如下。
##      Type：定义启动时的进程行为。它有以下几种值。
##      Type=simple：默认值，执行ExecStart指定的命令，启动主进程
##      Type=forking：以 fork 方式从父进程创建子进程，创建后父进程会立即退出
##      Type=oneshot：一次性进程，Systemd 会等当前服务退出，再继续往下执行
##      Type=dbus：当前服务通过D-Bus启动
##      Type=notify：当前服务启动完毕，会通知Systemd，再继续往下执行
##      Type=idle：若有其他任务执行完毕，当前服务才会运行
##      ExecStart：启动当前服务的命令
##      ExecStartPre：启动当前服务之前执行的命令
##      ExecStartPost：启动当前服务之后执行的命令
##      ExecReload：重启当前服务时执行的命令
##      ExecStop：停止当前服务时执行的命令
##      ExecStopPost：停止当其服务之后执行的命令
##      RestartSec：自动重启当前服务间隔的秒数
##      Restart：定义何种情况 Systemd 会自动重启当前服务，可能的值包括always（总是重启）、on-success、on-failure、on-abnormal、on-abort、on-watchdog
##      TimeoutSec：定义 Systemd 停止当前服务之前等待的秒数
##      Environment：指定环境变量
# Unit 配置文件的完整字段清单，请参考官方文档https://www.freedesktop.org/software/systemd/man/systemd.unit.html

# 启动计算机的时候，需要启动大量的 Unit。如果每一次启动，都要一一写明本次启动需要哪些 Unit，显然非常不方便。Systemd 的解决方案就是 Target。
# 简单说，Target 就是一个 Unit 组，包含许多相关的 Unit 。启动某个 Target 的时候，Systemd 就会启动里面所有的 Unit。从这个意义上说，Target 这个概念类似于"状态点"，启动某个 Target 就好比启动到某种状态。
# 传统的centOS6的init启动模式里面，有 RunLevel 的概念，跟 Target 的作用很类似。不同的是，RunLevel 是互斥的，不可能多个 RunLevel 同时启动，但是多个 Target 可以同时启动。
# 查看当前系统的所有 Target
systemctl list-unit-files --type=target

# 查看一个 Target 包含的所有 Unit依赖
systemctl list-dependencies [Target名称]

# 查看系统启动时的默认 Target
systemctl get-default

# 设置启动时的默认 Target
systemctl set-default [Target名称]

# 切换 Target 时，默认不关闭前一个 Target 启动的进程，
# systemctl isolate 命令改变这种行为，
# 关闭前一个 Target 里面所有不属于后一个 Target 的进程
systemctl isolate [Target名称]

# Target 与 传统centOS6的 RunLevel 运行级别的对应关系如下
Runlevel 0   -> poweroff.target   代表电源断电关机
Runlevel 1   -> rescue.target     代表单用户救援模式
Runlevel 2   -> multi-user.target 代表不完全的多用户模式
Runlevel 3   -> multi-user.target 代表完全的多用户模式
Runlevel 4   -> multi-user.target 未设定
Runlevel 5   -> graphical.target  代表图像界面
Runlevel 6   -> reboot.target     代表重启

# 它与init进程的主要差别如下。
# 默认的 RunLevel（在/etc/inittab文件设置）现在被默认的 Target 取代，位置是/etc/systemd/system/default.target，通常符号链接到graphical.target（图形界面）或者multi-user.target（多用户命令行）。
# 启动脚本的位置，以前是/etc/init.d目录，符号链接到不同的 RunLevel 目录 （比如/etc/rc3.d、/etc/rc5.d等），现在则存放在/lib/systemd/system和/etc/systemd/system目录。
# 配置文件的位置，以前init进程的配置文件是/etc/inittab，各种服务的配置文件存放在/etc/sysconfig目录。现在的配置文件主要存放在/lib/systemd目录，在/etc/systemd目录里面的修改可以覆盖原始设置。
~~~
---
 