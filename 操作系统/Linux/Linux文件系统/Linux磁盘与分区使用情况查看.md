# 磁盘内部的数据组成
- 数据区：存放文件的内容
- 组织区：存放文件的信息等
  - inode表：记录了每个inode节点对应文件所占用的块位置，读取文件结束的本质是根据文件的长度读取磁盘数据而文件的末尾并没有标识
  - 块组位图表：由多个块（一般是4096字节）组成的块组，每一个 bit 位都描述当前分区中一个块的使用情况，df 命令统计磁盘使用快的原因就是检查的块位图表，而不是去具体的统计文件大小


~~~shell
# 显示目前在 Linux 系统上的文件系统磁盘使用情况统计
df [参数] [挂载点]
# 挂载点可以加，也可以不加，不加显示的是所以的文件系统
# -h参数表示使用人类可读的格式（显示容量单位，默认不加-h参数容量的单位是Kb）显示，有六列，分别是文件系统名称、文件系统容量、已经使用容量、剩余可用容量、以用容量的百分比、挂载点路径。
# -a参数表示all，显示所有的文件系统信息，包括特殊的文件系统，例如/proc 、/sysfs等
# -T参数表示Type，会多一列显示文件系统的类型。文件系统有ext3、ext4（一般的硬盘或U盘文件系统）、iso9660（光盘文件系统）、NTFS（移动硬盘文件系统）
# -m参数表示以Mb显示文件系统容量、已经使用容量、剩余可用容量的大小
# -k参数表示以Kb显示文件系统容量、已经使用容量、剩余可用容量的大小

# ls -h可以查看文件的大小，但是如果是目录就显示的是目录下的第一级子目录名称和子文件名称所占用的大小
# 查看目录下的文件夹的大小
du [参数] [文件或目录]
# -h参数表示使用人类可读的格式（显示容量单位，默认不加-h参数容量的单位是Kb）显示
# -a参数表示all，默认不加-a参数只显示子目录和所有递归的子目录大小，而不显示子文件和所有递归的子文件大小
# -s参数表示统计整个目录文件夹的大小，包括子目录、所有递归的子目录、子文件、所有递归的子文件之和

# 注意du命令只统计的是文件和目录大小，而df命令统计的不仅文件和目录大小还包括命令和程序占用的空间（最常见的就是文件已经被删除但是程序并没有释放空间），所以df命令看到的磁盘使用量才是真实的


# 查看设备分区和挂载点详情
lsblk

#查询本机可以识别的硬盘和分区
[root@localhost ~]# fdisk -l
                                                     #第一块硬盘识别
Disk /dev/sda:32.2 GB, 32212254720 bytes             #硬盘文件名和硬盘大小
255 heads, 63 sectors/track, 3916 cylinders          #共255个磁头、63个扇区和3916个柱面
Units = cylinders of 16065 *512 = 8225280 bytes      #每个柱面的大小
Sector size (logical/physical): 512 bytes/512 bytes  #每个扇区的大小
I/O size (minimum/optimal): 512 bytes/512 bytes Disk identifier: 0x0009e098
Device Boot Start End Blocks ld System               #表格头一共7列：设备文件名、启动分区、起始柱面、终止柱面容量、分区的大小、分区ID、系统
/dev/sda1 * 1 26 204800 83 Linux                     #分区1的信息，linux的标准分区ID是83
Partition 1 does not end on cylinder boundary.       #分区1没有占满硬盘
/dev/sda2 26 281 2048000 82 Linux swap / Solaris     #分区2的信息，这是一个swap交换分区，swap交换分区ID是83
Partition 2 does not end on cylinder boundary        #分区2没有占满硬盘
/dev/sda5 281 3917 29203456 5 Linux                  #分区3的信息，这是扩展分区，扩展分区ID是5

Disk /dev/sdb: 21.5 GB, 21474836480 bytes            #第二个硬盘识别，这个硬盘没有分区
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes/512 bytes Disk identifier: 0x00000000


# 查询指定分区详细文件系统信息（挂载点对应的文件系统信息）
dumpe2fs [参数] [文件系统名称]
# -h仅显示超级块中的信息，而不显示磁盘块组的详细信息，查看Default mount options的属性是否有acl（基本都有），如果有则对应挂载点分区支持ACL权限
# 以下是执行dumpe2fs /dev/vda1显示的信息
dumpe2fs 1.39 (29-May-2006)
Filesystem volume name:   <none>             <==这个是文件系统的名称(Label)
Last mounted on:          /                  <==挂载点
Filesystem UUID:          4f4de78b-e926-4e41-9ab6-0643a8eed46f   <== 硬盘的唯一识别码UUID
Filesystem features:      has_journal ext_attr resize_inode dir_index
  filetype needs_recovery sparse_super large_file
Default mount options:    user_xattr acl <==默认挂载的参数
Filesystem state:         clean          <==这个文件系统是没问题的(clean)
Errors behavior:          Continue
Filesystem OS type:       Linux
Inode count:              2560864        <==inode的总数
Block count:              2560359        <==block的总数
Free blocks:              1524760        <==还有多少个 block 可用
Free inodes:              2411225        <==还有多少个 inode 可用
First block:              0
Block size:               4096           <==每个 block 的大小啦！
Filesystem created:       Fri Sep  5 01:49:20 2008
Last mount time:          Mon Sep 22 12:09:30 2008
Last write time:          Mon Sep 22 12:09:30 2008
Last checked:             Fri Sep  5 01:49:20 2008
First inode:              11
Inode size:               128            <==每个 inode 的大小
Journal inode:            8              <==底下这三个与下一小节有关
Journal backup:           inode blocks
Journal size:             128M

Group 0: (Blocks 0-32767) <==第一个 data group 内容, 包含 block 的启始/结束号码
  Primary superblock at 0, Group descriptors at 1-1  <==超级区块在 0 号 block
  Reserved GDT blocks at 2-626
  Block bitmap at 627 (+627), Inode bitmap at 628 (+628)
  Inode table at 629-1641 (+629)                     <==inode table 所在的 block
  0 free blocks, 32405 free inodes, 2 directories    <==所有 block 都用完了！
  Free blocks:
  Free inodes: 12-32416                              <==剩余未使用的 inode 号码
Group 1: (Blocks 32768-65535)
#由于数据量非常的庞大，这里省略了一部分输出信息，以上信息大致可分为 2 部分。前半部分显示的是超级块的信息，包括文件系统名称、已使用以及未使用的 inode 和 block 的数量、每个 block 和 inode 的大小，文件系统的挂载时间等。另外，Linux 文件系统（EXT 系列）在格式化的时候，会分为多个区块群组（block group），每 个区块群组都有独立的 inode/block/superblock 系统。此命令输出结果的后半部分，就是每个区块群组的详细信息（如 Group0、Group1）。
~~~