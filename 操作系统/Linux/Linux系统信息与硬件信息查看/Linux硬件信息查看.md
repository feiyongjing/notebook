~~~shell
# cpu核心数查看，lscpu看CPU(s) 行显示的是多少
cat /proc/cpuinfo | grep processor | wc -l
lscpu  

# 内存查看
free -m 
free -g

# 查看磁盘容量
lsblk
df -h
~~~