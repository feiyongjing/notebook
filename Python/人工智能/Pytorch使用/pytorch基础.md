# pytorch安装
~~~shell
# 注意安装时先检查当前的CUDA和显卡驱动程序版本是否匹配（cmd中使用NVIDIA-smi查看版本）
# 请根据CUDA的版本在 https://pytorch.org/get-started/locally/#no-cuda-1 获取对应的pytorch安装命令
# 如下是cuda 12.1 版本
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
~~~

# 检查是否安装成功
~~~python
import torch
# 注意前提是显卡支持，即检查计算机是否支持 CUDA
print("是否支持GPU", torch.cuda.is_available())
~~~
---

# Tensor对象（也叫张量）内部存储一个多维数组，和 Numpy 中的 ndarry对象 或 Pandas 中的 DataFrame对象 类似

# Tensor对象创建和属性
~~~python
import torch

# empty()创建一个Tensor对象，默认内部的元素就是内存中原有的数据，可能数字极大或极小
data = torch.empty((3, 5))

# Tensor()创建一个Tensor对象，按照参数指定每个维度的大小，默认内部的元素就是内存中原有的数据，可能数字极大或极小
data = torch.Tensor(3, 5)

# IntTensor()创建一个Tensor对象内部多维数组元素是int类型，按照参数指定每个维度的大小，默认内部的元素就是内存中原有的数据，可能数字极大或极小
# 同理还有 FloatTensor() 等作用与IntTensor()类似
data = torch.IntTensor(3, 5)

# 创建一个元素全是0的Tensor对象
# dtype参数指定矩阵元素类型
data = torch.zeros((3, 5), dtype=torch.long)

# 创建一个元素全是指定数值的Tensor对象
# dtype参数指定矩阵元素类型
data = torch.full((3, 5), 7)

# arange函数是设置指定范围数字元素的Tensor对象
# 第一二参数设置元素的范围，元素可以等于第一参数但是必须小于第二参数
# 第三参数是步长，即从第一参数开始等距取范围内的数作为数组元素，如果不设置默认步长是1
data = torch.arange(0, 10, 2)

# linspace函数是设置指定范围数字的数组
# 第一二参数设置元素的范围，元素可以等于这两个数字
# 第三参数是最终数组的长度，会按照长度在范围内等距取数作为数组元素
data = torch.linspace(0, 32, 9)

# 根据多维数组创建Tensor对象
# 注意如果参数是一个数字则表示的是一个0维的数组，通过 0维数组 创建的Tensor对象是一个标量
# 通过 1维数组 创建的Tensor对象是一个向量
# 3维数组 创建的Tensor对象适合用于文字处理
# 4维数组 创建的Tensor对象适合用于图片处理，4个维度分布表示 图片的数量、图片的色彩（rgb等）、图片的高、图片的宽
data2 = torch.tensor([[1, 2], [3, 4]])

# rand()内部的元素呈现随机的均匀分布，内部的元素类型要求必须是浮点类型，例如 torch.rand(3,3) 生成3行3列随机的均匀分布的矩阵
# randn()内部的元素呈现随机的正态分布，内部的元素类型要求必须是浮点类型，例如 torch.rand(3,3) 生成3行3列随机的正态分布的矩阵
# randint()内部的元素呈现随机的均匀分布，但是内部的元素类型是int，torch.randint(1, 10, [3,3]) 生成3行3列随机的均匀分布的矩阵，其中矩阵元素的最小和最大值分别是1和10
# randn_like()根据一个Tensor对象内部多维数组的形状创建出一个新的Tensor对象，内部的元素呈现随机的正态分布，内部的元素类型要求必须是浮点类型
# 还有一些与上述函数名类似的函数，都是遵守这些规则，即随机分布类型、内部元素类型、按照形状创建新的Tensor对象
data3 = torch.randn_like(data2, dtype=torch.double)

print("Tensor对象中多维数组的维度", data3.dim())
print("Tensor对象的形状，是一个表示数组维度的元组，即每个维度有多少元素", data3.shape)
print("Tensor对象的形状，是一个表示数组维度的元组，即每个维度有多少元素", data3.size())
print("Tensor对象中多维数组中有多少元素", data3.numel())

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

# Tensor对象的形状变更
~~~python
import torch

data = torch.empty((4,4))
print("原始数据\n", data)
# view()函数和reshape()函数都修改Tensor对象的形状（使用方式完全一致），参数设置每个维度的长度，如果是-1就是根据总元素除以其它全部维度长度进行自动补全
data =data.view(-1,2,4)
print("修改后的形状\n", data)

data = torch.empty((4, 2, 1))
# 注意unsqueeze() 和 squeeze() 方法返回的是一个新的 Tensor 对象，原始的 Tensor 对象不会被改变
# unsqueeze()函数会给Tensor对象添加一个维度，参数是指定添加维度完成后的该维度索引（shape属性的索引，可以使用负数，默认最后一个元素的索引是-1）
# 注意添加的维度默认该维度长度是1
data = data.unsqueeze(1)
print("Tensor对象添加维度后的形状\n", data.shape)
# squeeze()函数会给Tensor对象减少一个维度，参数是指定添加维度完成后的该维度索引（shape属性的索引，可以使用负数，默认最后一个元素的索引是-1）
# 注意只能减少的维度长度是1的维度
data = data.squeeze(1)
print("Tensor对象减少维度后的形状\n", data.shape)


data = torch.empty((4, 1, 1))
# expand()函数将Tensor对象中每个维度的长度扩容到指定的长度
# 扩容需要保证如下规则
# 如果对应维度长度一致则该维度保持不变
# 如果对应维度长度不一致，并且原始Tensor对象对应维度长度不是1，则无法进行扩容执行函数会报错
# 原始Tensor对象对应维度长度如果是1而新的维度长度大于1则对该维度进行复制扩容到指定长度（更低维度的数据也会一起复制）
data = data.expand(4, 2, 3)
print("Tensor对象中每个维度的长度扩容到指定的长度后的形状\n", data.shape)
# repeat()函数将Tensor对象中每个维度的长度扩容指定倍数
# 扩容需要保证如下规则
# 如果对应维度扩容倍数是1，则该维度保持不变
# 如果对应维度扩容倍数大于1，则对该维度进行复制扩容指定倍数（更低维度的数据也会一起复制）
data = data.repeat(1, 3, 3)
print("Tensor对象中每个维度的长度扩容指定倍数后的形状\n", data.shape)


data = torch.empty((3, 5))
print("二维的Tensor对象将行变成列，列变成行\n", data.t().shape)

data = torch.rand((3,3,3))
print("Tensor对象原数据\n", data)
# permute()函数变更Tensor对象中全部元素的索引位置，即每个元素的索引都发生变化
# 元素索引变化规则如下
# permute()函数的参数设置新的维度对应于原来的维度
# 例如新的维度应于原来的维度 2，1，0 即新的维度中第一维度对应原来的第三维度，第二维度不变，第三维度对应原来的第第一维度
# 具体变换例如 原来元素的索引是 0,2,2 该元素在变换后索引就是2,2,0
print("二维的Tensor对象设置新的维度顺序\n", data.permute(2,1,0))
~~~
---

# Tensor对象的合并与拆分
~~~python
import torch

data = torch.rand((4, 3, 3))
data1 = torch.rand((5, 3, 3))
# cat()函数合并 Tensor 对象不会新增维度
# dim参数设置沿着指定维度进行合并，即指定的维度数组长度变长
# 注意所有的Tensor对象维度除了dim参数设置的维度之外，其他的维度长度都必须一致
data2 = torch.cat([data, data1], dim=0)
print("Tensor对象合并后的形状\n", data2.shape)

data = torch.rand((5, 4, 3))
data1 = torch.rand((5, 4, 3))
data2 = torch.rand((5, 4, 3))
# stack()函数合并 Tensor 对象会新增一个维度
# dim参数设置添加一个新维度的索引，新的维度长度就是 Tensor 对象的数量
# 注意所有的Tensor对象维度长度都必须一致
data3 = torch.stack([data, data1, data2], dim=0)
print("Tensor对象合并后的形状\n", data3.shape)

data = torch.rand((5, 4, 3))
# split()函数拆分 Tensor 对象
# 第一参数是数组或者是数字。如果是数组则按照数组指定维度拆分，数组长度决定拆分的段数，其中的元素数字分别决定每段的长度。
# 如果是数字则按照每段都拆分为这个长度的段数，当然如果指定维度长度无法整除数字则最后一段的长度就会短一些
# dim参数设置按照指定维度进行拆分
# 注意所有的Tensor对象维度长度都必须一致
data1, data2, data3 = data.split([2, 2, 1], dim=0)
print("Tensor对象拆分后的形状\n", data1.shape, data2.shape, data3.shape)

data = torch.rand((6, 4, 3))
# split()函数拆分 Tensor 对象
# 第一参数是数字，表示最终会拆分为这个数量的段数
# dim参数设置按照指定维度进行拆分
data1, data2, data3 = data.chunk(3, dim=0)
print("Tensor对象拆分后的形状\n", data1.shape, data2.shape, data3.shape)
~~~
---

# Tensor对象的算术运算
~~~python
# pip install torch torchvision torchaudio
import torch

data1 = torch.tensor([[1, 2], [3, 4]])
data2 = torch.tensor([[1, 2], [3, 4]])

# Tensor 对象之间可以通过加件乘除求余等算术运算符对 内部的全部元素进行操作
# 但是需要满足 broadcast (广播机制进行自动扩展)：
# 两个 Tensor 对象维度中从低维度开始对比，如果有其中有一个 Tensor 对象该维度数组长度为1 或者两个 Tensor 对象该维度数组长度相等，则该维度满足算术运算规则，继续比较更高的维度，直到其中维度数低的 Tensor 对象的全部维度检验完毕（当然两个Tensor 对象维度相等需要全部维度检验完毕），必须全部维度都满足运算规则，否则两个 Tensor 对象不能进行算术运算

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

# Tensor对象使用 * 向乘默认是内部对应位置的元素相乘
print("Tensor对象使用 * 向乘默认是内部对应位置的元素相乘\n", data1 * data2)

# 以下是矩阵运算的三种方式，
# 方式一：torch.mm() 只适用于二维矩阵运算
data3 = torch.mm(data1, data2)
# 方式一：torch.matmul() 适用于多维矩阵运算，多维矩阵运算本质其实只是最后两个维度进行矩阵运算，而其他的维度需要满足 broadcast (广播机制进行自动扩展)
data4 = torch.matmul(data1, data2)
# 方式三：@ 运算符号其实就是重载了 torch.matmul() 函数
data5 = data1 @ data2
print("Tensor对象进行矩阵运算", data3)
print("Tensor对象进行矩阵运算", data4)
print("Tensor对象进行矩阵运算", data5)

data = torch.tensor([[1, 2], [3, 4]])
# torch.clamp()函数设置限制 Tensor 对象中元素的取值范围
# Tensor 对象中的元素小于 min，则会被设置为 min；如果 Tensor 对象中的元素大于 max，则会被设置为 max
print(torch.clamp(input=data, min=2, max=3))
~~~
---


# Tensor对象放入显卡运算和从显卡拿出Tensor对象到cpu
~~~python
import torch

# 检查计算机是否支持 CUDA
if torch.cuda.is_available():
    # 定义GPU设备变量，通过cuda获取gpu设备,如果有多个gpu设备可以指定对应的gpu设备，如 cuda:0 使用数字指定gpu设备
    device = torch.device("cuda")

    # 创建Tensor对象变量x放在cpu中
    x = torch.randn(1)
    # 创建Tensor对象变量y放在指定设备上，这里是放在gpu中
    y = torch.randn_like(x, device=device)

    # 将cpu中的变量和gpu的变量无法直接运算，变量必须都在cpu或者是gpu才能运算
    # 将cpu的变量x放入gpu中
    # 方法一：将变量放入指定的gpu设备中
    x = x.to(device)
    # 方法二：注意这是放在默认的gpu设备中，如果需要放入指定的gpu设备请使用方法一
    # x=x.cuda()

    # gpu中的变量进行运算
    z = x + y
    print("打印GPU中的结果", z, type(z))
    # 将cpu中的变量转移到gpu中
    z = z.to("cpu", dtype=torch.double)
    print("打印CPU中的结果", z, type(z))

    print("查看当前GPU的显存使用情况，单位是字节", torch.cuda.memory_allocated())

    # 释放显存
    # torch.cuda.empty_cache()

~~~
---














