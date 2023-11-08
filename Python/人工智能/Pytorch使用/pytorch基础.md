
# Tensor对象（也叫张量）内部存储一个多维数组，和 Numpy 中的 ndarry对象 或 Pandas 中的 DataFrame对象 类似


# Tensor对象创建和属性
~~~python
# pip install torch torchvision torchaudio
import torch

# 创建一个Tensor对象，默认内部的元素就是内存中原有的数据，可能数字极大或极小
data = torch.empty((3, 5))

# 创建一个元素全是0的Tensor对象
# dtype参数指定矩阵元素类型
data1 = torch.zeros((3, 5), dtype=torch.long)

# 根据多维数组创建Tensor对象
data2 = torch.tensor([[1, 2], [3, 4]])

# 根据一个Tensor对象内部多维数组的形状创建出一个新的Tensor对象，内部的元素都是随机数
# 注意randn_like()内部的元素类型要求必须是浮点类型
data3 = torch.randn_like(data2, dtype=torch.double)

print("Tensor对象的形状，是一个表示数组维度的元组，即每个维度有多少元素", data3.shape)
print("Tensor对象的形状，是一个表示数组维度的元组，即每个维度有多少元素", data1.size())


data4 = torch.empty((4,4))
print("原始数据\n", data1)
# view()函数修改Tensor对象的形状，参数设置每个维度的长度，如果是-1就是根据总元素除以其它全部维度长度进行自动补全
data5 =data4.view(-1,2,4)
print("修改后的形状\n", data5)
~~~
---

# Tensor对象的算术运算
~~~python
# pip install torch torchvision torchaudio
import torch

data1 = torch.tensor([[1, 2], [3, 4]])
data2 = torch.tensor([[1, 2], [3, 4]])

# 方式一：直接通过算术运算符进行算术运算
print("Tensor对象相加结果\n", data1 + data2)

# 方式二：通过对应的运算函数计算
print("Tensor对象相加结果\n", data1.add(data2))
print(data1.dtype)

# 方式三：通过对应的运算函数计算，注意out参数设置运算结果输出到指定的变量
data3 = torch.empty(data1.shape)
torch.add(data1, data2, out=data3)
print("Tensor对象相加结果\n", data3)

# 方式四：对应的运算函数如果函数名末尾是下划线结尾则代表计算结果会赋值给调用者，即会影响原数据
data1.add_(data2)
print("Tensor对象相加结果\n", data1)
~~~
---

# Tensor对象索引获取元素
~~~python
# pip install torch torchvision torchaudio
import torch

data1 = torch.tensor([[1, 2], [3, 4]])

# 注意索引时如果需要该维度的全部元素请只写 :
print("Tensor对象索引内部元素\n", data1[0:1, 1:])
print("Tensor对象索引内部元素\n", data1[:, 1:])
~~~
---


# Tensor对象和ndarray对象相互转换
~~~python
# pip install torch torchvision torchaudio
import torch

data1 = torch.empty((4, 4))
print("原始数据\n", data1)

# 注意CharTensor对象无法和ndarray对象相互转换，其他的Tensor对象基本都可以进行转换
# 注意Tensor对象和ndarray对象本质是使用的同一内存空间，所以修改其中一个另一个也会发生变化

# numpy()函数将Tensor对象转换ndarray对象
data2 = data1.numpy()
print("Tensor对象转为Numpy的ndarray对象\n", data2)

# numpy()函数将Tensor对象转换ndarray对象
data3 = torch.from_numpy(data2)
print("Numpy的ndarray对象转为Tensor对象\n", data3)
~~~
---


# 
~~~python
~~~
---

# 
~~~python
~~~
---

# 
~~~python
~~~
---










