# 常见内置函数
~~~python

# print() 函数打印字符串
print("121w")

# type() 函数判断参数类型
print(type("aaa"))

# input() 函数打印字符串，并且阻塞等待标准输入返回输入的字符串
input("用户请输入密码")

# len() 函数获取字符串的长度
print(len("Aw12w"))

# reversed() 函数翻转列表或者字符串
print(list(reversed([10, 20, 30])))

# slice() 函数返回字符串或列表的截取方式
# 例如如下，和 "axg87qgde783"[1:9:2] 的效果是一样的
print("axg87qgde783"[slice(1, 9, 2)])

# all() 函数的参数只有全部参数表示true时返回true，否则返回false 
all([1,"","123"])

# any() 函数的参数只要有一个参数表示true时就返回true，而全部是false时才返回false 
any([1,"","123"])

# for循环使用 range() 函数遍历列表可以获取到列表的索引和元素
for i in range(len(lst)):
    print(lst[i])

# for循环使用 enumerate() 函数可以获取到列表的索引和元素
for index, item in enumerate(["cy","232","123"]):
    print(index, item)

# hash() 函数计算不可变对象的hash值，hash值并不是内存地址，而内存地址是根据hash值计算出来的
hash("adjiwe")

# id() 函数获取不可变对象的内存地址
id("adjiwe")

# help() 函数查看class帮助信息，其实就是看源码，一般在开发时用不到，而在只有控制台命令行时需要使用
help(str)

# dir() 函数查看变量可以执行的操作，其实就是看源码，一般在开发时用不到，而在只有控制台命令行时需要使用
dir("哈哈")

# zip() 函数将多个列表的元素按照索引拆分为聚合为多个元组（即将每个列表同一索引位置的元素放入一个元组，每个索引位置产生一个元组），返回的是迭代器
lst1 = ["打野恶魔小丑 萨科", "辅助琴瑟仙女 娑娜", "AD寒冰射手 艾希"]
lst2 = ["打野破败之王 佛耶戈", "辅助光辉女郎 拉克丝", "AD荣耀行刑官 德莱文"]
lst3 = ["打野时间刺客 艾克", "辅助魂锁典狱长 锤石", "AD暗夜猎手 薇恩"]
res = zip(lst1, lst2, lst3)
for item in res:
    print(item)

# globals() 函数返回一个dict字典，内部有全局作用域中所有变量名和值
lst1 = ["打野恶魔小丑 萨科", "辅助琴瑟仙女 娑娜", "AD寒冰射手 艾希"]
lst2 = ["打野破败之王 佛耶戈", "辅助光辉女郎 拉克丝", "AD荣耀行刑官 德莱文"]
lst3 = ["打野时间刺客 艾克", "辅助魂锁典狱长 锤石", "AD暗夜猎手 薇恩"]
print(globals())

# locals() 函数返回一个dict字典，内部有当前所在局部作用域中所有变量名和值
def fn():
    a=18
    print(locals())
fn()


# sorted() 函数进行列表排序，返回排序后的列表
# key参数是元素排序函数，指定元素按照什么进行排序，默认是从小到大排序
# reverse参数表示是否翻转排序后的列表，默认是不翻转，即最终从小到大排序，翻转最终是从大到小排序
dt = [
    {"id": 1, "age": 23, "name": "AD寒冰射手 艾希"},
    {"id": 2, "age": 38, "name": "AD荣耀行刑官 德莱文"},
    {"id": 3, "age": 21, "name": "AD暗夜猎手 薇恩"}
]
print(sorted(dt, key=lambda item: item["age"], reverse=True))

# filter() 函数进行列表过滤，返回一个迭代器，从迭代器获取过滤后的列表
# 第一个参数是过滤函数，只返回true的元素
# 第二参数是需要过滤的列表
dt = [
    {"id": 1, "age": 23, "name": "辅助琴瑟仙女 娑娜"},
    {"id": 2, "age": 38, "name": "打野破败之王 佛耶戈"},
    {"id": 3, "age": 21, "name": "AD暗夜猎手 薇恩"}
]

f = filter(lambda item: not item["name"].startswith("AD"), dt)

for i in f:
    print(i)


# map() 函数进行列表元素映射到另一种类型的元素，返回一个迭代器，从迭代器获取映射后的列表
# 第一个参数是元素映射函数，返回新的元素
# 第二参数是需要映射的列表
dt = [
    {"id": 1, "age": 23, "name": "上单灵罗娃娃 格温"},
    {"id": 2, "age": 38, "name": "中单时间刺客 艾克"},
    {"id": 3, "age": 21, "name": "中单刀锋之影 泰隆"}
]

for i in map(lambda item: "打野" + item["name"][2:], dt):
    print(i)

~~~
---

### 基础数据类型相关的内置函数（一般没有参数就是创建的空字符串、空列表、空字典等）
~~~python
# 创建bool值
b = bool()

# 创建int值
i = int()

# 创建float值
f = float()

# 创建字符串
s = str()

# 创建list
lst = list()

# 创建元组
t = tuple()

# 创建set
st = set()

# 创建frozenset，其实frozenset就是不可变的集合
st = frozenset()

# 创建dict字典
di = dict()

# 创建bytes
by = bytes()

~~~
---

### 进制转换（注意除了转换为十进制之外的转换都是返回的是字符串）
~~~python
# 转换为二进制数的字符串
b = bin(21)

# 转换为八进制数的字符串
o = oct(21)

# 转换为十进制数
i = int(0b10101)

# 转换为十六进制数的字符串
h = hex(21)

# 进行进制转换，b 表示 2进制，o 表示 8进制，x 表示 16进制，前面的 08 表示最终结果如果不足 8 位就在前面补 0
b1 = format(10,"08b")
~~~
---

### 数学运算函数
~~~python
# 求和运算
sum([10, 20, 30])

# 获取最大值
max([10, 20, 30])

# 获取最小值
min([10, 20, 30])

# 获取次幂，例如获取 10 的 3 次方
pow(10, 3)

~~~
---

### unicode编码转换
~~~python
# 获取字符的 unicode 码
print(ord("꧃"))


# unicode 码转换为 字符
print(chr("43459"))


# 查看unicode 码 0到65536的字符
for i in range(65536):
    print(chr(i)+" ",end="")

~~~
---

###


# python的关键字
### 以下代码打印的都是关键字
~~~python
import keyword

print(keyword.kwlist)
~~~
---
