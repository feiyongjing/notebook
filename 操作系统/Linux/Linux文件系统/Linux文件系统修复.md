~~~shell
# 先使用mount检查设备文件是否挂载成功
mount [分区设备文件] [挂载的目录]
mount -a 
# mount -a 检查安装 /etc/fstab 文件配置的自动挂载是否有问题

# 修复文件系统之前需要先保证解除文件系统的挂载
umount [分区设备文件]

# 备份数据
dd if=/dev/sdX of=/path/to/backup.bin bs=64K conv=noerror,sync status=progress
# if设置的是设备文件目录
# of设置的是备份的文件，会将设备文件备份打包成一个文件
# bs=64K 设置块大小（可以调整以优化性能）
# conv=noerror,sync 参数让 dd 在遇到读取错误时不会停止，并在必要时填充输入块
# status=progress 参数会在运行时打印备份数据的进度和磁盘IO速度
# 使用 nohup 命令 和 在命令最后加 & 设置程序挂到后台运行，并且会在当前目录下生成nohup.out存放程序的标准输出和标准错误
# 当然也可以在最后的 & 之前手动设置程序的标准输出和标准错误到指定文件
# 例如 nohup dd if=/dev/sdX of=/path/to/backup.bin bs=64K conv=noerror,sync status=progress > myout.txt 2>&1 & 
# 可以使用 jobs -l 查看后台运行的任务，注意每个任务都有一个任务编号，通过任务编号可以杀死后台任务，例如kill -9 %1 杀死任务编号是1的后台任务

# ext4等ext系列文件系统修复，注意提前备份数据
# 在文件系统正常的情况下不要使用，否则会出现问题
fack [参数] [分区设备文件名]
# -a参数表示不用显示用户提示，自动修复文件系统
# -y参数，与-a参数的作用一致，不过有些文件系统只支持-y参数

# xfs文件系统修复，注意提前备份数据
xfs_repair [分区设备文件，例如/dev/sdd]
# 如果修复失败就先执行xfs_repair -L /dev/sdd(-L是修复xfs文件系统的最后手段，慎重选择，它会清空日志，会丢失用户数据和文件)
# 再次执行xfs_repair /dev/sdd 尝试修复文件系统
# 最后执行xfs_check /dev/sdd 检查文件系统是否修复成功，如果xfs_check命令不存在就手动使用 mount挂载设备文件检查是否挂载成功
~~~