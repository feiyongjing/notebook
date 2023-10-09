

~~~python
# 在 Python 中，traceback 模块提供了处理和格式化异常信息的功能。
# 当程序发生异常时，可以使用 traceback 模块来获取异常的详细信息，包括异常类型、异常消息、异常发生的位置等。
import traceback
s = "abdiuhbdu"

# 使用 try 关键字确定可能出现异常的代码块
# 使用 except 关键字确定可能出现异常并且处理该异常，如果不写具体抓取的异常就是抓取Exception
# 接 else 没有出现异常是执行的代码块
# 使用 finally 关键字执行无论是否出现异常都执行的代码块
# Exception是所有异常的父类
# 异常具有传递性如果在函数栈中没有try、except处理异常就会一直向调用者函数传递异常，如果没有任何函数处理异常程序就会报错停止
try:
    a = s[20]
except IndexError as e :
    print("处理IndexError")
    # print_exc()函数会打印出当前异常的详细信息，包括异常类型、异常消息、异常发生的位置等。
    # 可以通过 limit 参数指定打印堆栈的深度，如果不指定则打印所有堆栈信息。
    # 可以通过 file 参数指定打印输出的文件，如果不指定则输出到标准错误流。
    traceback.print_exc()
except Exception as e :
    print("Exception")
else:
    print("没有出现异常")
finally:
    print("一定执行的操作，一般用于关闭一些资源")
~~~
---