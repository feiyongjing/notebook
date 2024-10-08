~~~shell
# 修改文件系统属性，注意对root也会生效
chattr [+-=] [参数] [文件或目录名]
# -----------e- ./.bashrc  显示的结果，前面一串都是文件的隐藏属性（也叫文件系统权限）,具体属性如下
#     a:让文件或目录仅提供附加用途，只允许以追加方式读写文件
#     A:当文件被访问时，它的atime(访问时间)记录不会被修改。这为系统避免了一定数量的磁盘I/o。
#     c:将文件或目录压缩后存放。文件会被内核自动压缩到磁盘上。从这个文件读取未压缩的数据。对该文件的写入会在将日期存储到磁盘之前压缩它们。
#     C:属性设置为“C”的文件将不会copy-on-write(写时复制)。此属性仅在支持copy-on-write的文件系统上支持。
#     d:将目录排除在倾倒操作之外。
#     D:目录被修改时，这些修改被同步写入磁盘;这相当于应用于文件子集的“dirsync”挂载选项。
#     e:表示文件使用区段来映射磁盘上的块。不能使用chattr移除。
#     E:用来表示压缩文件有压缩错误。虽然可以用Isattr显示，但chttr不能设置或重置它。
#     h:表示文件以文件系统块大小为单位存储其块，而不是以扇区为单位，并表示该文件大于2TB。虽然可以通过Isattr显示，但chattr不能设置或重置
#     i:文件不能被修改:不能被删除或重命名，不能创建到该文件的链接，也不能向该文件写入数据。
#     I:文件或目录使用散列树对目录进行索引。不能使用chattr设置或重置它，但可以用lsattr显示。
#     j:如果文件系统是用data=ordered或data=writeback选项挂载的，那么带有“j”属性的文件在被写到文件本身之前，它的所有数据都被写到ext3或ext4日志中。当用“data-journal”#     选项挂载文件系统时，所有文件数据都已经被记录，这个属性没有作用。
#     N:表示该文件将数据内联存储在inode本身中。不能使用chattr来设置或重置，但可以通过lsattr来查看。
#     s:允许一个文件被安全地删除文件被删除时，它的块被归零并写回磁盘。
#     S:即时更新文件或目录。文件被修改时，这些修改将同步写入磁盘。
#     t:带有“t”属性的文件在与其他文件合并的文件末尾不会有部分块片段(对于那些支持tail-merging的文件系统)。这是
#     对于像LILO这样直接读取文件系统且不支持tail-merging的文件的应用程序来说是必需的。
#     T:目录将被视为目录层次结构的顶部。这是给块分配器的一个提示由ext3和ext4使用，该目录下的子目录是不相关的，因此为了分配目的应该分散开来。
#     u:预防意外删除。若文件删除，系统会允许你在以后恢复这个被删除的文件
#     X:可以直接访问压缩文件的原始内容。不可以使用chattr重置或设置，可以使用lsattr显示。
# chattr的参数如下
# +-=表示增加、减少、等于权限
# -R参数，递归设置权限
# 一般是设置如下的i权限或者a权限
# 如果对文件设置i权限，那么不允许删除文件、修改文件名、移动文件、添加和修改文件内容，但是可以查看和拷贝即使是-p参数拷贝也可以修改拷贝文件。如果对目录设置i权限，那么可以修改目录下文件的内容，不能修改目录名、移动目录、在目录下删除文件和建立文件，但是可以拷贝即使是-p参数拷贝也可以修改拷贝文件。
# 如果对文件设置a权限，那么不允许删除文件、修改文件名、移动文件、修改文件内容，但是可以添加数据到文件中、可以拷贝，即使是-p参数拷贝也可以修改拷贝文件。如果对目录设置a权限，那么可以修改目录下文件的内容，不能修改目录名、移动目录，但是可以在目录下删除文件和建立文件和拷贝即使是-p参数拷贝也可以修改拷贝文件。



# 查看文件系统的属性
lsattr [参数] [文件名或目录]
# -a参数代表all显示所有的文件和目录
# -d参数，当查看的是目录时添加-d参数代表只看目录的属性，而不看目录下的子文件和子目录
# 文件的i权限在第1列的第5个字符显示，或者添加-l参数在第2列中有Immutable显示表示不可变的
# 文件的a权限在第1列的第6个字符显示，或者添加-l参数在第2列中有 Append_Only显示表示只允许添加  

~~~