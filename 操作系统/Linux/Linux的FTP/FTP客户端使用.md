## FTP客户端使用（SFTP客户端使用类似）
~~~shell
# 连接到 FTP 服务器
ftp ftp.example.com
# -v 显示命令执行过程
# -d 详细显示命令执行过程，便于排错或分析程序执行的情形
# 系统会提示你输入用户名和密码。如果 FTP 服务器允许匿名访问，你可以使用 anonymous 作为用户名，并使用你的电子邮件地址作为密码。
# 如果服务端不是默认的21端口，需要使用 ftp -p [host] [port] 设置服务端口，-p参数表示被动模式
# 可以先登录账号，然后使用 site PORT [port] 修改端口 
# 当然也可以直接输入ftp 进入交互式命令行输入 open [host] [port] 进行连接
# 也有些ftp命令行客户端是支持 -P参数直接指定服务端口
# windows 支持指定包含 FTP 命令的文本文件；文本中的命令在 FTP 启动后自动运行，例如 ftp -i -s:test.txt ，-i参数关闭多文件传输过程中的交互式提示，-s: 指定包含FTP 命令的脚本文件
# test.txt 内容如下，匿名登录然后下载文件
open 192.168.1.227 2121
anonymous
anonymous
get msftest.apk
quit

# 常用 FTP 命令，一旦成功连接到 FTP 服务器，你可以使用以下常用命令进行文件操作：
# ls：列出远程服务器上的文件和目录
# cd：更改远程服务器上的当前目录
# lpwd：查看本地的当前目录
# lcd：更改本地计算机上的当前目录
# get：从远程服务器下载文件到本地计算机：get remote_file local_file
# put：将本地文件上传到远程服务器：put local_file remote_file
# mget：下载多个文件：mget file1 file2 file3
# mput：上传多个文件：mput file1 file2 file3
# delete：删除远程服务器上的文件：delete remote_file
# mkdir：在远程服务器上创建目录： mkdir directory_name
# rmdir：删除远程服务器上的目录： rmdir directory_name
# rename：远程服务器上重命名文件：rename old.txt new.txt
# bye 或 quit：断开与 FTP 服务器的连接并退出 ftp 客户端
~~~

