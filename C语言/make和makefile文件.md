### makefile文件是用来管理项目工程的文件，规定了项目的编译流程

### make命令是用来解析并执行makefile文件

# makefile详细解释
~~~
目标:依赖
（注意这行开头是tab，必须是tab而不是空格）命令

makefile基本三元素
目标：要生成的目标文件
依赖：目标文件由哪些文件生成
命令：通过执行该命令由依赖文件生成目标文件

makefile 中使用#作为注释符号

简单的例子
main.o: test1.c test2.c test3.c
    gcc -o main.o test1.c test2.c test3.c
    
运行 make 会在当前目录下查找 名称是makefile或Makefile 的文件进行解析执行第一个目标
运行 make [目标] 会在当前目录下查找 名称是makefile或Makefile 的文件进行解析执行指定的目标
运行 make -f [指定的makefile文件] 会在按照指定的 makefile 文件进行解析执行第一个目标
运行 make -f [指定的makefile文件] [目标] 会在按照指定的 makefile 文件进行解析执行指定的目标

规则：检查目标文件和依赖文件是否存在
  1、目标文件不存在，依赖文件存在就直接执行命令生成目标文件
  2、目标文件不存在，依赖文件不存在就在当前makefile中查找对应的依赖是否有作为目标生成出来，如果有就先递归执行命令生成依赖在生成目标，没有就报错
  3、目标文件存在，依赖文件存在就检查依赖文件的修改时间和目标文件的修改时间，如果依赖文件的生成时间在目标文件的修改时间之后就重新执行命令生成新的目标文件覆盖原有目标文件，依赖文件的生成时间在目标文件的修改时间之前不会执行命令生成目标文件
  4、目标文件存在，依赖文件不存在就在当前makefile中查找对应的依赖是否有作为目标生成出来，如果有就先递归执行命令生成依赖在生成目标，没有就报错
  
变量：类似于c中的宏替换
  1、普通变量：用户自定义的变量
  2、自带变量：makefile中内置的默认变量，用户可以直接使用
  3、自动变量：注意自动变量只能在命令中使用
普通变量
  1、定义普通变量，AAA=123
  2、使用普通变量，$(AAA)
自带变量
  1、CC=gcc   # arm-linux-gcc
  2、CPPFLAGS # gcc的预处理参数 -I 指定需要的头文件目录
  3、CFLAGS   # gcc的编译器参数 -Wall -g -c  进行编译调试
  4、LDFLAGS  # gcc的链接器参数 -L -l 指定需要的库文件目录和库名称
自动变量：注意自动变量只能在命令中使用，不能使用自动变量替换目标和依赖
  1、$@ 表示规则中的目标
  2、$< 表示依赖列表的第一个依赖
  3、$^ 表示依赖全部的依赖列表，注意依赖列表是一个字符串，依赖是以空格隔开，并且会去除重复的依赖
  
规则匹配：用于简化重复的依赖和目标生成，但是这无法被make识别为第一个目标，并且注意
  %表示字符串，即依赖和目标的名称是一样的
%.o:%.c
    $(CC) -o $@ $^

makefile常见函数
  1、wildcard 函数用于获取指定类型的文件
  2、patsubst 函数用于替换匹配的字符串
wildcard 函数获取当前目录指定后缀的文件列并赋值给变量
  src=$(wildcard *.c) 
patsubst 函数替换src变量列表中后缀为 .c 的文件替换为 .o 文件
  obj=$(patsubst %.c, %.o, $(src))

清理生成的目标：本质还是添加一个新的目标，例如clean目标
.PHONY指定目标不进行依赖与目标的生成检查，该目标只执行命令
rm命令建议添加-f命令防止出现目标文件不存在时删除报错
命令前添加 @ 则在执行命令时不打印执行的命令到显示屏上
命令前添加 - 则在执行命令时遇见错误也会继续执行下面的命令
例子：
.PHONY:clean
clean:
    -rm [不存在的文件，执行会报错]
    rm -f [目标文件1] [目标文件2]
    
运行 make clean 或者是 make -f [指定的makefile文件] clean 执行clean目标

~~~

### makefile的例子
~~~makefile
src=$(wildcard *.c)
target=$(patsubst %.c, %.o, $(src))

include_path=/usr/include/mysql
lib_path=/usr/lib64/mysql
lib_name=mysqlclient

all: $(target)

# 这是规则被make识别为第一个目标
%.o: %.c
  $(CC) $^ -o $@ -I$(include_path) -L$(lib_path) -l$(lib_name)

.PHONY:clean
clean:
  -rm -f $(target)
~~~
 