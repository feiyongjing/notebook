# Linux磁盘LVM分区及扩容操作参考：https://blog.csdn.net/Ruishine/article/details/117133185

# 参考  
https://blog.csdn.net/qq_34291570/article/details/126800373

# 例子中有扩展root分区、创建新的data分区以及将原来的home分区挂载回去
~~~shell
#查看分区
df -h

#卸载/home分区
umount /home
#如果出现 home 存在进程，使用 fuser -m -v -i -k /home 终止 home 下的进程，最后使用 umount /home 卸载 /home）

# 安装fuser命令
yum install psmisc

#删除分区，腾出磁盘空间
lvremove /dev/mapper/centos-home

# 给/root增加磁盘空间
lvextend -L +100G /dev/mapper/centos-root
# 扩展root文件系统
xfs_growfs  /dev/mapper/centos-root

# 创建新的data分区并划分磁盘空间
lvcreate -L 570G -n data centos
# 重新创建home分区并划分剩余的磁盘空间
lvcreate -l +100%free -n home centos

# 给新的分区创建文件系统格式化
mkfs.xfs /dev/centos/home
mkfs.xfs /dev/centos/data

# 手动挂载新的分区
mount /dev/centos/home /home
mkdir /data
mount /dev/centos/data /data

# 将挂载信息写入/etc/fstab让电脑开机自动挂载
echo “/dev/mapper/centos-data /data  xfs  defaults  0 0” >>/etc/fstab
~~~

### 非LVM文件系统分区:  https://zhuanlan.zhihu.com/p/83340525