
~~~shell
javac  [class路径]    编译原代码生成字节码 ，路径需要加  .java  的后缀，如果出现编码错误在路径前加参数   -encoding utf-8

java [全限定类名路径]   运行字节码，路径不需要加后缀，

jps 查看Java进程

ps -ef | grep java 查看Java进程   第二列是pid

kill -9 8568 8569 ...  根据pid杀死多个进程

taskkill -pid  8568 -F  强行杀死pid是8568的进程

 

netstat -ano|findstr 8080  查看占用8080端口的pid，即最后一列

taskkill /pid 13188 /F           杀死pid是13188端口的进程
~~~
 

# windows 查看进程的端口和pid

在开始搜索资源监视器，打开资源监视器----》网络----》侦听端口 可以看到对应程序的pid  和端口

在任务管理器中找到对应的pid程序就可以关闭这个程序了

 