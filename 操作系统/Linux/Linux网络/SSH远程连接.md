### SSH是用于远程管理的服务，端口号是22端口，在Linux中的守护进程是sshd，

### 服务端的服务是/usr/sbin/sshd 配置文件是/etc/ssh/sshd_config，修改完配置文件需要执行systemctl restart sshd 重启sshd服务

### 客户端的服务是/usr/bin/ssh 配置文件是/etc/ssh/ssh_config

# 远程连接和传输文件
~~~shell
#客户端远程登录服务端，第一次登录会询问是否下载服务器加密的公钥，直接回答yes
ssh 用户名@服务端IP
ssh -l 用户名 服务端IP

# 下载远程服务器的文件到本地，如果是下载文件夹添加-r参数
scp 用户名@服务端IP:/root/test.txt .

# 本地文件上传到远程服务器，如果是上传文件夹添加-r参数
scp ./test.txt 用户名@服务端IP:/root/

# 进入交换式命令，可以查看本地和远程的文件并且上传下载
sftp 用户名@服务端IP
# 在交换式命令下使用help可以查看交互式命令的帮助
# ls 查看服务器的文件
# cd 切换服务器的目录
# lls 查看本地的文件
# lcd 切换本地的目录
# get 下载，例如 get [远程服务器文件路径] [本地文件路径]
# put 上传，例如 put [本地文件路径] [远程服务器文件路径]
~~~
---
 

# 服务端配置文件介绍
~~~
Port 22 # SSH服务的端口号，一般都是注释的，默认就是22
ListenAddress 0.0.0.0 # 设置sshd服务器绑定的IP地址,即只开启绑定IP的ssh远程服务，0.0.0.0是默认sshd服务器所有的网卡IP都绑定
HostKey /etc/ssh/xxx  # 指定ssh私钥存放的位置
SyslogFacility AUTHPRIV # 记录日志，默认等级是INFO
PermitRootLogin yes # 是否允许远程登陆root用户
PubkeyAuthentication yes # 是否使用公钥验证
AuthorizedKeysFile      .ssh/authorized_keys  # 指定公钥保存针对用户家目录的相对位置
PasswordAuthentication yes # 是否允许使用密码登录
PermitEmptyPasswords no    # 是否允许使用空密码登录﻿​
~~~
---
 

# 使用ssh-keygen生成密钥对（可以用于Git和SSH密钥登录）
~~~
ssh-keygen -t rsa
然后依序输入：
密钥存放位置或密钥文件名
输入密钥锁码
设置密钥锁码后指即使有私钥也要验证你设置的锁码，可以有效防止私钥被盗用，不设置的话可以留空
重复输入密钥锁码
然后就会在当前目录下生成一对密钥文件（有的系统是在.ssh下生成）
（.pub后缀的就是公钥）
~~~
---
 

# SSH密钥登录设置
~~~
1. 在服务器上创建新的用户： useradd $username
2. 在新用户的home目录下创建一个.ssh的文件夹，该文件夹的权限是700，owner是$username
3. 将新用户的公钥上传到服务器上，拷贝到~/.ssh/目录下，并重命名为authorized_keys
4. 将authorized_keys文件权限修改为644，同时修改所有者为$username

多个人使用不同的密钥登录Linux同一用户

使用cat pubilc.ras >> ~/.ssh/authorized_keys  将多个公钥都追加到文件的末尾，拥有与公钥对应的私钥就可以密钥登录 
~~~
---
 

 