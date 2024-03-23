## go语言常见关键字
- conture 跳出当前次的循环，进入下一次循环
- break 跳转语句，跳出循环或switch语句，可以跳转到指定的标签位置
- goto 指定一个的标签位置，通过break 指定标签跳转到该标签位置
- if 条件表达式，如果
- else 条件表达式，否则
- switch 流程控制语句，根据不同的条件执行不同的语句块，与case、default 一起使用
- case 用于 switch 或 select 语句块，case 用于指定一个或多个值（常量或者是表达式）表示满足条件就执行其中的语句块
- default 默认选项，switch 或 select 的默认选项
- fallthrough 表示通过当前语句，switch语句中表示可以执行下一个语句块
- select 是golang语言层面的IO多路复用机制，用于检测管道是否就绪，与case和default一起使用
- package 声明当前包名
- import 引入包
- interface 声明一个接口
- func 声明一个函数
- type 声明一个类型
- var 声明一个变量
- const 声明一个常量
- return 函数返回语句
- defer 用于函数延迟调用
- go 用于启动协程
- map 表示集合类型
- struct 表示结构体类型
- chan 表示管道类型
- range 用于for循环、遍历数组、切片、管道、集合等类型，获取其中的元素、索引、键值
- for 用于循环语句

# 预定义标识符
- iota 是用于常量声明，即该常量是默认类型的默认值，后续的常量会默认是前一个常量加1
- new 是用于开辟指定类型大小的内存空间，并返回该内存空间的指针