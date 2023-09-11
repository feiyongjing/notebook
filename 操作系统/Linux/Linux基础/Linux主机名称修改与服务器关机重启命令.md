~~~shell
# 查看主机名称
hostname

# 临时修改主机名，重启失效
hostname [新的主机名称]

# 永久修改主机名
hostnamectl set-hostname [新的主机名称]

# 永久修改主机名，需要重启电脑生效
vi /etc/hostname 

~~~
----


~~~shell
# linux关机重启，当然还有其他的关机命令halt、init 0等。但是poweroff断电关机就不推荐
shutdown [参数] [时间]
# -h参数表示关机
# -r参数表示重启
# -c参数表示取消关机前一个命令，比如计划是晚上9点关机但是设置成晚上7点关机了，可以使用shutdown -c取消之前的设置
# 时间指定，例如now表示立刻、20:00固定晚上8点、30表示30分钟后关机

# 其他重启命令
init 6
reboot

# 退出登录
logout

~~~
----