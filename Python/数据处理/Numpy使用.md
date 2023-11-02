### Numpy是一个高性能的科学计算库，可以用于快速处理任意维度的数组运算

### Numpy使用 ndarray 对象处理多维数组，该对象是一个快速而灵活的大数据容器

### Numpy 与 原生list 的运算速度比较
~~~python
# pip install numpy
import numpy as np
import random
import time

a = []

tt=time.time()
for i in range(100000000):
    a.append(random.random())

print("创建具有一亿个数的数组需要的时间", time.time() - tt)

# 数组计算一亿个数相加
t1 = time.time()
suma = sum(a)
t2 = time.time()

# ndarray对象计算一亿个数相加
b = np.array(a)  # 返回 ndarray 对象
t3 = time.time()
sumb = np.sum(b)
t4 = time.time()

print("数组计算一亿个数相加的时间", t2 - t1)
print("numpy的ndarray对象计算一亿个数相加的时间", t4 - t3)
# Numpy大约快8倍

~~~
---

# ndarray 为啥要比 原生的list快
1. Numpy存储的是同一类型的数据所以内部是将数据存储在一块连续的内存空间中，批量操作快。而原生的list内部存储的元素类型可以不同，导致碎片化存储，批量操作元素比较慢
2. Numpy支持并行化运算（向量化运算）
3. Numpy的底层是使用c语言编写的，内部解除了GIL（全局解释器锁），其对数组的操作速度不受Python解释器的限制，效率远高于纯Python代码

### ndarray内部的属性
~~~python
# pip install numpy
import numpy as np
import random
import time

# 创建ndarray 对象，建议使用规整的数组，否则会出现后续的一些函数无法正常使用
# 可以传递参数 dtype 表示传递的元素类型，数字类型一般是默认，而其他的类型由于计算复杂所以一般不用设置
# 在 NumPy 中，可以使用以下数据类型或者是自定义数据类型
# bool：布尔类型，占用 1 个字节。
# int8、int16、int32、int64：分别为有符号的 8、16、32、64 位整数类型，占用 1、2、4、8 个字节。
# uint8、uint16、uint32、uint64：分别为无符号的 8、16、32、64 位整数类型，占用 1、2、4、8 个字节。
# float16、float32、float64、float128：分别为半精度、单精度、双精度、四倍精度浮点数类型，占用 2、4、8、16 个字节。
# complex64、complex128、complex256：分别为单精度、双精度、四倍精度复数类型，占用 8、16、32 个字节。
# object：Python 对象类型，可以存储任意 Python 对象。
# string_、unicode_：分别为 ASCII 和 Unicode 字符串类型，可以指定字符串长度。
score = np.array([
    [25, 51.2, 165, 3],
    [30, 49.6, 160, 6],
    [35, 55.8, 169, 2],
    [18, 49.9, 162, 4],
    [26, 60.3, 170, 9],
    [17, 70.8, 163, 0],
    [19, 66.2, 172, 4],
    [29, 67.8, 180, 8]
], dtype=np.float64)

print("ndarray对象的形状，是一个表示数组维度的元组，即每个维度有多少元素", score.shape)
print("ndarray对象的表示的数组维度", score.ndim)
print("ndarray对象中元素的数量", score.size)
print("ndarray对象一个数组元素占用的字节大小", score.itemsize)
print("ndarray对象中元素的类型", score.dtype)
~~~
---

### Numpy 自定义元素的数据类型 
~~~python
# pip install numpy
import numpy as np

# 创建一个自定义的数据类型，指定字段名、数据类型和字节顺序
my_dtype = np.dtype([('name', np.str_, 16), ('age', np.int32), ('height', np.float64)])

# 创建一个使用自定义数据类型的 ndarray
arr = np.array([('Alice', 25, 1.65), ('Bob', 30, 1.80)], dtype=my_dtype)

# 查看 ndarray 的数据类型
print(arr.dtype)
~~~
---

# 创建 ndarray 对象的方式

### 方式一：快速生成元素全是0的 ndarray对象
~~~python
# pip install numpy
import numpy as np

# zeros()函数生成元素全是0的多维数组
# 第一个参数是元组表示每个维度的元素数量
# dtype参数设置元素的类型
n1 = np.zeros((3, 5), dtype=float)

print(n1, n1.dtype)

# zeros()函数生成元素全是1的多维数组
# 第一个参数是元组表示每个维度的元素数量
# dtype参数设置元素的类型
n2 = np.ones((2, 3), dtype=int)

print(n2, n2.dtype)
~~~
---

### 方式二：通过 现有数组或ndarray对象 生成 ndarray对象
~~~python
# pip install numpy
import numpy as np

data = np.zeros((3, 5))
# 第一个参数是多维数组或 ndarray对象
# dtype参数设置元素的类型
# 当数据源是ndarray对象时 array()和copy()函数是深拷贝，asarray()函数是浅拷贝
score1 = np.array(data)
score2 = np.asarray(data)
score3 = np.copy(data)

# 修改原数组
data[0][0] = 10000
print("原数组", data)
print("array函数得到的ndarray对象", score1)
print("asarray函数得到的ndarray对象", score2)
print("copy函数得到的ndarray对象", score3)
~~~
---

### 方式三：指定范围等差数列生成 ndarray对象
~~~python
# pip install numpy
import numpy as np

# linspace函数是设置指定范围数字的数组
# 第一二参数设置元素的范围，元素可以等于这两个数字
# 第三参数是最终数组的长度，会按照长度在范围内等距取数作为数组元素
score1 = np.linspace(0, 32, 9)

# arange函数是设置指定范围数字的数组
# 第一二参数设置元素的范围，元素可以等于第一参数但是必须小于第二参数
# 第三参数是步长，即从第一参数开始等距取范围内的数作为数组元素
score2 = np.arange(0, 10, 2)

print(score1)
print(score2)
~~~
---


### 方式四：随机数组生成 ndarray对象，一般有均匀分布数组和正态分布数组
~~~python
# pip install numpy
import numpy as np

# uniform函数生成均匀分布的数组
# 第一二参数设置最大最小值
# 第三参数是元组设置数组每个维度的元素数量
score1 = np.random.uniform(0, 100, 100)

# uniform函数生成正态分布的数组
# 第一参数是平均值，第二参数是标准差，标准差越大则波动程度越大
# 第三参数是元组设置数组每个维度的元素数量
score2 = np.random.normal(10, 5, 100)

print("均匀分布的数组", score1)
print("均匀分布数组的平均值", score1.mean())
print("正态分布的数组", score2)
print("正态分布数组的平均值", score2.mean())
~~~
---

# ndarray 对象索引获取元素和分割元素
~~~python
# pip install numpy
import numpy as np

score1 = np.random.normal(0, 1, (3, 4, 5))

print(score1)

# 和普通的数组索引类似，使用逗号分割不同的维度元素获取，由前到后指定不同维度元素索引
print("ndarray索引获取元素", score1[1, 1, 1])

# 和普通的数组分割类似，使用逗号分割不同的维度元素获取，由前到后指定不同维度元素索引
print("ndarray索引分割获取元素", score1[1, 1:3, ])
~~~
---

# ndarray 对象内部多维数组的形状修改（维度数量与每个维度的长度） 和 行与列的数据转换
~~~python
# pip install numpy
import numpy as np

data = [[1, 2, 3, 4, 5], [6, 7, 8, 9, 10], [11, 12, 13, 14, 15]]

score1 = np.array(data)
print("原数据是3行5列", score1)

# reshape函数会重新设置多维数组每个维度的长度，并返回一个新的ndarray对象，但是内部数据错乱
score2 = score1.reshape((5, 3))
print("新的数据是5行3列，但是数据错乱", score2)

# resize函数会重新设置多维数组每个维度的长度，但是注意内部数据错乱，并且没有创建新的ndarray对象而是修改原ndarray对象数据
score1.resize((5, 3))
print("修改原数据为5行3列，但是数据错乱", score1)

# ndarray的T属性具有行变成列，列变成行的功能，返回一个新的ndarray对象
score3 = score1.T
print("新的数据是5行3列，并且数据不错乱", score3)
~~~
---

# ndarray 对象内部元素类型转换、ndarray 对象序列化和反序列化、ndarray 对象内部元素去重
~~~python
# pip install numpy
import numpy as np

score1 = np.random.normal(10, 5, 500)

print("原始数据", score1)

# astype函数进行元素数据类型转换
print("元素转换为int类型", score1.astype(int))

# tobytes函数将ndarray对象序列化转换为二进制字符串
score2 = score1.tobytes()
print("ndarray序列化转换为二进制字符串", score2)

# frombuffer函数二进制字符串反序列化转换为ndarray对象，但是注意需要明确声明内部元素的数据类型
score3 = np.frombuffer(score2, dtype=float)
print("二进制字符串反序列化转换为ndarray对象", score3)

# unique函数对ndarray对象内部的元素去重
# return_counts会返回去重后每个元素对应原数据出现的次数
score4, count = b, count = np.unique(np.array([[1, 2, 3, 4], [3, 4, 5, 6]]), return_counts=True)
print("ndarray对象去重数据和每个元素对应原数据出现的次数", score4, count)
~~~
---

# ndarray 对象的逻辑运算，即条件过滤和条件赋值
~~~python
# pip install numpy
import numpy as np

score1 = np.random.normal(10, 5, (5, 10))

print("原始数据1", score1)

print("逻辑判断大于8的元素标记为True", score1 > 8)

print("逻辑判断布尔索引过滤大于8的元素", score1[score1 > 8])

# 逻辑判断布尔索引统一修改数据
score1[score1 > 8] = 10
print("大于8的元素都会变成10", score1)

# all函数判断是否全部元素满足条件，any函数判断是否有部分元素满足条件
print("判断第一维度前两个数组并且第二维度在前5个元素是否全部大于5", np.all(score1[:2, :5] > 5))
print("判断第一维度前两个数组之外的数组并且第二维度在前5个元素之外的元素是否有小于5", np.any(score1[2:, 5:] < 5))

# where函数表示三元运算，logical_and函数和logical_or分别代表条件与和条件或
score2 = score1[:2, :5]
print("原始数据2", score2)
print("判断指定的部分数据是否大于8，如果满足则置为1，否则置为0", np.where(score2 > 8, 1, 0))
print("判断指定的部分数据是否大于6并且小于11，如果满足则置为1，否则置为0",
      np.where(np.logical_and(score2 > 6, score2 < 11), 1, 0))
print("判断指定的部分数据是否小于6或者大于11，如果满足则置为1，否则置为0",
      np.where(np.logical_or(score2 < 6, score2 > 11), 1, 0))
~~~
---