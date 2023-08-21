日志存储信息查看，存储的日志类型一般是json

~~~shell
cat /etc/docker/daemon.json
~~~

日志存储在 /var/log/containers/ 目录下，其中的文件名对于Pod的名称