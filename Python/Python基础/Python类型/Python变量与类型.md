### 常见类型
~~~python
# 单行注释

"""
3个单引号或者是双引号表示多行注释或字符串
"""

# 一般函数的注释是写在函数内部的样例如下，其中有参数名和返回值的说明
'''    
    :param conn: 
    :return: 
'''

# TODO 在注释后添加TODO表示该注释处有那些功能没有完成

# 在Python中没有常量是声明，但是约定变量名称全部大写的变量是常量并且一次赋值后不会去修改它
PI = 3.14

# 数字类型：int、float

# 布尔类型bool，只有True或者False。
# 0以外的数字转换为布尔都是True，而0转换为布尔是False。
# ""以外的字符串转换为布尔都是True，而""字符串转换为布尔是False
# []以外的列表转换为布尔都是True，而[]列表转换为布尔是False

# 空None
# 字符串类型str
# 列表list
# 元组tuple
# set
# 字典dict
# bytes
# 函数function 函数的名称就代表函数变量，本质是一个内存地址

~~~
---


### 局部变量和全家变量
~~~python
# global 关键字在函数内部引入全局变量并且修改
a=10
def abc():
    global a
    a = 20
    print(a)

abc()
print(a)


# nonlocal 关键字在函数内部引入上一层作用域的变量并且修改
def main():
    b = 10
    print(b)
    def aaa():
        nonlocal b
        b = 20
    aaa()
    print(b)

main()

~~~
---

