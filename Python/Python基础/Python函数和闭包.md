# 函数的定义
~~~python
# 定义函数
# 可以直接设置默认参数，如下message
def main(name, message='Hello', age="18"):
    print(name, message)

# 调用函数，使用默认参数    
main("张三")

# 调用函数，覆盖默认参数    
main("张三","你今天吃饭了吗？")

# 调用函数不按照参数的先后顺序传递参数，而是根据参数的名称直接进行匹配传递
main(message="沙漠死神",name="内瑟斯")

# 混合调用，同时使用参数的位置顺序和参数的名称传递参数，但是位置参数必须在全部都在参数的名称传递参数之前
# 调用函数不写参数名称的参数必须按照顺序传递，但是写参数名称的参数可以不按照参数的先后顺序传递参数
main("艾尼维亚", age="???", message="冰晶凤凰")


# 函数的定义中如果在参数的前面加* 就表示接收的参数在方法内部被当成是一个元组
def main1(*args):
    print(args)

main1("恶魔小丑 萨科", "琴瑟仙女 娑娜", "迅捷斥候 提莫")

# 函数的定义中如果在参数的前面加** 就表示这是一个字典dict
def main2(**name):
    print(name)
    
main2(刀锋之影 = "泰隆",暮光之眼 = "慎")


# 函数定义时参数的顺序一般是：位置参数，*args(元组), 默认值参数（使用时通过名称传递），**kwargs(字典dict)
def main3(a, b, *args, c="哈哈", **kwargs):
    print(a, b, args, c, kwargs)

main3("a", "b", 1, 2, c="呵呵", name1="琴瑟仙女 娑娜", name2="迅捷斥候 提莫")

# *args, **kwargs可以接受任意的参数，这在源码中大量的出现
def main4(*args, **kwargs):

~~~
---

### 传递长元组、列表、字典参数的简写
~~~python
def main(*args):
    print(args)

lst=['{', '"恶魔小丑": "萨科",', '"琴瑟仙女": "娑娜",', '"迅捷斥候": "提莫"', '}']

# 传递长元组、列表、字典
# 传递列表（元组）参数和 字典参数 时可以简写为 *列表 和 **字典 直接解析列表或字典传递参数 
main(*lst)
~~~
---

# 函数的返回
~~~python
def main(*args):
    print(args)
    # return会立即退出函数，不写return 或者 return时不写返回值会返回None
    # return 值1,值2,值3,... 该函数返回多个值，在外界是得到一个元组，元组内部存放的是返回的多个值
    return args

~~~
---

### 闭包
~~~python
# 通过函数的嵌套来封装内部的变量，暴露的函数就是闭包函数
def main():
    b = 1
    def aaa():
        nonlocal b
        b += 1
        return b
    return aaa

f = main()

print(f())
print(f())
~~~
---

### 匿名函数（lambda表达式）
~~~python
# 语法 
# lambda 参数1, 参数2, ... : 返回值

fn = lambda a, b: a + b

print(fn(10, 20))

~~~
---