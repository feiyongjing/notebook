~~~shell
# ACL权限：在出现已经明确分配好了文件的所有者、所属组、其他人的权限下如果出现另外的特殊用户，而特殊的用户权限与现有的文件的所有者、所属组、其他人的权限都不一致的情况下应该如何分配权限？而给特殊用户分配ACL权限就是答案，要使用ACL权限功能必须对应的分区支持（一般都会支持），以下有查看分区是否支持ACL权限功能，以及在不支持ACL权限的情况下如何开启对应分区的ACL权限

# 显示目前在 Linux 系统上的文件系统磁盘使用情况统计
df [参数]
# -h参数表示使用人类可读的格式显示，有六列，分别是文件系统名称、文件系统容量、已经使用容量、剩余可用容量、以用容量的百分比、挂载点路径

# 查询指定分区详细文件系统信息（挂载点对应的文件系统信息）
dumpe2fs [参数] [文件系统名称]
# -h仅显示超级块中的信息，而不显示磁盘块组的详细信息，查看Default mount options的属性是否有acl（基本都有），如果有则对应挂载点分区支持ACL权限

# 临时开启分区挂载点的ACL权限（关机会失效）
mount -o remount,acl [挂载点]
# 重新挂载已经挂载的分区并增加ACL权限

# 永久开启分区挂载点ACL权限需要修改/etc/fstab配置文件，配置文件中第二列代表挂载点路径，根据挂载点路径找到对应的挂载点配置在第4列的defaults后添加逗号acl 然后保存退出
# 退出后重启系统（其实启动系统时会读取配置文件进行挂载分区）或者是mount -o remount [挂载点]重新进行挂载分区

# 查看文件目录的ACL权限
getfacl [文件]
# 如果文件的路径是绝对路径那么输出的第一行信息是要求删除根/，但是实际是不影响的
# 显示权限信息如下
# file: ocm-algorithm-install.sh 文件的名称
# owner: root  文件的所有者
# group: root  文件的所属组
user::rw-            #用户名为空表示的是文件的所有者，所有表示的是文件的所有者的权限
user:dirondhwab:r-x  #指定的dirondhwab用户的ACL权限，dirondhwab用户拥有读和执行权限
group::r--           #用户名为空表示的是文件的所有者，所有表示的是文件的所有者的权限
mask::r-x            #表示最大有效权限，除了文件所有者之外的用户和组都与这个权限进行相与得到的权限是用户或用户组得到的真正权限，一般情况下都是有rwx权限是不用管的
other::r--           #其他人的权限


# 设置文件的ACL权限
setfacl [参数] [文件]
# -m参数设置ACL权限，例如 setfacl -m u:dirondhwab:rx /mnt/test/ 给用户dirondhwab赋予/mnt/test/目录的rx权限，u改为g代表给dirondhwab组赋予权限，u改为m并且没有用户名表示设置最大的有效权限
# -x参数删除指定的ACL权限，例如 setfacl -m u:dirondhwab /mnt/test/ 给删除dirondhwab对/mnt/test/目录的ACL权限，u改为g代表删除dirondhwab组的ACL权限
# -b参数删除目录所有的ACL权限，删除所有用户以及所有组的ACL权限
# -R参数递归设置或取消ACL权限
# 设定默认ACL权限，例如 setfacl -m d:u:dirondhwab:rx -R /mnt/test/ 给用户dirondhwab赋予/mnt/test/目录的rx权限，同时之后新建的所有递归的子文件和子目录的都继承相同的ACL权限
# 删除默认ACL权限，例如 setfacl -m k:u:dirondhwab:rx -R /mnt/test/ 给用户dirondhwab赋予/mnt/test/目录的rx权限 

~~~