### Pandas是一个数据处理工具，Pandas集成了 Matplotlib 和 Numpy，并且读取文件方便

# Pandas的核心数据结构：DataFrame、Series

## DataFrame的使用

### DataFrame的本质是有行索引和列索引的二维数组
~~~python
# pip install numpy
import numpy as np
# pip install pandas
import pandas as pd

score = np.random.normal(0, 0.3, (4, 5))
print("原始数据\n", score)

# DataFrame函数生成DataFrame对象
# 第一参数data是数据
# index是行索引，即行的表头
# columns是列索引，即列的表头
score1 = pd.DataFrame(score,
                      index=[f"股票{i}" for i in range(1, score.shape[0] + 1)],
                      columns=pd.date_range(start="2023-11-03", periods=score.shape[1]).strftime('%Y-%m-%d').tolist()
                      )
print(type(score1))
print("pandas处理后的数据\n", score1)

# DataFrame对象的一些属性
print("DataFrame对象的形状，是一个表示数组维度的元组，即每个维度有多少元素", score1.shape)
print("DataFrame对象的数组维度", score1.ndim)
print("DataFrame对象中元素的数量", score1.size)
print("DataFrame对象中列索引对应的数据类型\n", score1.dtypes)
print("DataFrame对象的行索引获取\n", score1.index)
print("DataFrame对象的列索引获取\n", score1.columns)
print("DataFrame对象中的多维数组\n", score1.values)
print("DataFrame对象行和列翻转，即行变成列，列变成行，数据是正确的\n", score1.T)

# DataFrame对象的一些常见函数
print("head函数获取的数组中开头指定行数的数据\n", score1.head(1))
print("tail函数获取的数组中结尾指定行数的数据\n", score1.tail(2))

# 修改行和列的索引注意只直接修改索引数组中的元素是不生效的，修改完索引数组中的元素后还需要赋值给DataFrame对象对应的索引属性
# 如下是股票后添加了一个下划线，先改索引数组或者是重新生成新的索引数组，再将索引数组赋值给DataFrame对象对应的索引属性
score1.index = [f"股票_{i}" for i in range(1, score.shape[0] + 1)]
print("修改索引后的数据展示\n", score1)

# reset_index函数会添加一列行索引，默认将原来的行索引变为列
# 注意drop参数代表是否去除原来的行索引转换的列, name参数设置原来的行索引变为列变为列后的名称
score1 = score1.reset_index(names='id')
print("去除索引后的数据展示\n", score1)

# set_index函数会将指定的列（可以使用元组指定多个列）变为新的行索引，注意会丢弃原来的行索引
# 注意drop参数代表是否去除指定的列，默认是去除
# 如果使用多个列最终会返回一个具有 Multilndex 的DataFrame对象，即行索引添加了一个维度，而DataFrame对象变为三维数组
score1 = score1.set_index(["id", "2023-11-03"])
print("指定列变为索引后的数据展示\n", score1)
print("DataFrame的Multilndex展示\n", score1.index)
print("Multilndex的表头，即列索引\n", score1.index.names)
print("Multilndex的内部数据\n", score1.index.levels)
~~~
---


## Series的使用
### Series的本质是一维数组，索引也只有行索引
~~~python
# pip install pandas
import pandas as pd

# Series对象的创建
# data参数是一维数组的数据，index参数设置索引，默认的索引是从0开始的自然数
# 当然也可以使用dict字典创建Series对象，默认字典的key是索引，而value是一维数组的数据
score = pd.Series([2, 67, 3, 6, 23], index=["周一", "周二", "周三", "周四", "周五"])
# score = pd.Series({"周一": 2, "周二": 67, "周三": 3, "周四": 6, "周五": 23})

print("Series对象\n", score)
print("Series对象索引获取\n", score.index)
print("Series对象中的数组\n", score.values)

score.index=range(1,6)

# ascending参数表示列索引对应的排序规则，True是升序,False是降序，默认是升序
score =score.sort_values()
print("Series对象按照数据排序\n", score)

score =score.sort_index()
print("Series对象按照索引排序\n", score)
~~~
---

# DataFrame索引操作获取对应的数据和修改数据
~~~python
# pip install numpy
import numpy as np
# pip install pandas
import pandas as pd

score = np.random.normal(0, 0.3, (4, 5))
print("原始数据\n", score)

score1 = pd.DataFrame(score,
                      index=[f"股票{i}" for i in range(1, score.shape[0] + 1)],
                      columns=pd.date_range(start="2023-11-03", periods=score.shape[1]).strftime('%Y-%m-%d').tolist()
                      )

print("pandas处理后的数据\n", score1)

# 索引名称获取数据
# 方式一：按照二维数组的方式通过行列索引名称获取数据即可，注意是先列索引后行索引
print("获取股票2在2023-11-03的数据", score1["2023-11-03"]["股票2"])
# 方式二：使用loc属性可以按照行列索引名称获取数据，先行索引后列索引，有如下两种写法
print("获取股票2在2023-11-03的数据", score1.loc["股票2", "2023-11-03"])
print("获取股票2在2023-11-03的数据", score1.loc["股票2"]["2023-11-03"])

# 按照数字索引,按照二维数组的方式通过数字获取数据
print("获取股票2在2023-11-03的数据", score1.iloc[1, 0])

# 混合索引：索引名称和数字混合使用获取数据
print("获取股票2和4后3天的数据", score1.loc[["股票2", "股票4"], score1.columns[2:]])
print("获取股票2和3后3天的数据", score1.loc[score1.index[1:3], ["2023-11-05", "2023-11-06", "2023-11-07"]])
print("获取股票2和3后3天的数据",
      score1.iloc[1:3, score1.columns.get_indexer(["2023-11-05", "2023-11-06", "2023-11-07"])])

# 如下是将股票1在2023-11-03的数据改为0.5，当然可以通过其他索引的方式获取到数据进行赋值修改
score1.loc["股票1", "2023-11-03"] = 0.5
print("股票1在2023-11-03的数据改为0.5后的数据展示\n", score1)
~~~
---

# DataFrame 按照数据排序和按照索引排序
~~~python
# pip install numpy
import numpy as np
# pip install pandas
import pandas as pd

score = np.random.normal(0, 0.3, (4, 5))

score1 = pd.DataFrame(score,
                      index=[f"股票{i}" for i in range(1, score.shape[0] + 1)],
                      columns=pd.date_range(start="2023-11-03", periods=score.shape[1]).strftime('%Y-%m-%d').tolist()
                      )

score1 = score1.reset_index(names='id')
print("pandas处理后的数据\n", score1)

# 按照数据进行排序
# sort_value函数对数据进行排序
# by参数是一个列表，表示按照那些列索引进行排序
# ascending参数是一个元组，表示列索引对应的排序规则，True是升序,False是降序，默认是升序
score1 = score1.sort_values(by=["2023-11-03", "2023-11-04"], ascending=(True, False))
print("数据排序处理后的数据\n", score1)

# 按照行索引进行排序
score1 = score1.sort_index()
print("行索引排序处理后的数据\n", score1)
~~~
---

# DataFrame 对象和 Series 对象的算术运算和逻辑运算
~~~python
# pip install numpy
import numpy as np
# pip install pandas
import pandas as pd

score = np.random.normal(0, 0.3, (4, 5))

score1 = pd.DataFrame(score,
                      index=[f"股票{i}" for i in range(1, score.shape[0] + 1)],
                      columns=pd.date_range(start="2023-11-03", periods=score.shape[1]).strftime('%Y-%m-%d').tolist()
                      )

print("pandas处理后的数据\n", score1)

# DataFrame 对象和 Series 对象可以直接与数字通过加件乘除求余等算术运算符对 内部的全部元素进行操作
score1 += 1
print("原始数据全部加1\n", score1)

print("逻辑判断大于0.5的元素标记为True\n", score1 > 0.5)
# 使用逻辑与和逻辑或是需要注意运算符优先级
print("逻辑判断大于0.4并且小于0.8的元素标记为True\n", (score1 > 0.4) & (score1 < 0.8))
print("逻辑判断布尔索引过滤小于0.4或者大于0.8的元素\n", score1[(score1 < 0.4) | (score1 > 0.8)])

# 使用query函数进行过滤
print("原始数据\n", score1.T)
print("过滤股票1大于0.4并且小于0.8的行数据\n", score1.T.query("股票1 > 0.4 & 股票1 < 0.8"))

# 逻辑判断布尔索引统一修改数据
score1[score1 > 0.8] = 1
print("大于0.8的元素都会变成1\n", score1)

# 使用isin函数逻辑判断是否包含指定数组中的值
print("判断2023-11-03是否有股票为1\n", score1["2023-11-03"].isin([1]))
print("过滤2023-11-03股票数据为1的股票\n", score1[score1["2023-11-03"].isin([1])])
~~~
---

# DataFrame 对象和 Series 对象的统计运算、自定义运算
~~~python
# pip install numpy
import numpy as np
# pip install pandas
import pandas as pd

score = np.random.normal(0, 0.3, (4, 5))

score1 = pd.DataFrame(score,
                      index=[f"股票{i}" for i in range(1, score.shape[0] + 1)],
                      columns=pd.date_range(start="2023-11-03", periods=score.shape[1]).strftime('%Y-%m-%d').tolist()
                      )

print("pandas处理后的数据\n", score1)

# max函数和min函数获取最大最小值
# axis参数设置统计区间，1是行，0是列，默认是列
print("按照列获取元素中的最大值\n", score1.max(axis=0))
print("按照列获取元素中的最小值\n", score1.min(axis=0))
print("按照列获取最大元素中的行索引，如果axis=1则是按照行获取最大元素中的列索引\n", score1.idxmax(axis=0))
print("按照列获取最小元素中的行索引，如果axis=1则是按照行获取最大元素中的列索引\n", score1.idxmin(axis=0))
print("按照列获取元素的标准差\n", score1.std(axis=0))
print("按照列获取元素的平均值\n", score1.mean(axis=0))
print("按照列获取元素的最大值、最小值、标准差、平均值等数据表格\n", score1.describe())

# 累计统计计算函数：计算前n行(或列)数的最大值、最小值、和、乘积等
# axis参数设置统计区间，1是行，0是列，默认是列
print("原始数据\n", score1.T)
print("按照列计算前n行数的最大值\n", score1.T.cummax())
print("按照列计算前n行数的最小值\n", score1.T.cummin())
print("按照列计算前n行数的和\n", score1.T.cumsum())
print("按照列计算前n行数的乘积\n", score1.T.cumprod())

# apply函数设置自定义运算函数
# 第一个参数就是自定义函数
# axis参数设置统计区间，1是行，0是列，默认是列
print("自定义一个函数计算每列的最大值与最小值的差\n", score1.apply(lambda x: x.max() - x.min(), axis=0))
~~~
---


# Pandas 支持读取多种文件的数据，包括csv、HDF5（是一种二进制文件）、json、剪切板的数据等

## Pandas 读取 csv文件 数据
~~~python
# pip install pandas
import pandas as pd

# read_csv函数读取csv文件为DataFrame对象
# 第一参数是csv文件路径
# usecols参数设置列名称，表示需要csv文件中这些列的数据
# 如果csv文件中没有在第一行表示列的名称，可以使用names参数设置列的名称
# encoding参数设置字符编码
score = pd.read_csv("test.csv",usecols=["时间","指标标识符","值"])

print("pandas读取csv文件的数据\n", score)

# to_csv函数将DataFrame对象数据写入到csv文件
# 第一参数是csv文件路径
# columns参数设置列名称，表示这些列的数据需要写入到csv文件中
# index参数表示是否将索引写入csv文件，默认写入，如果没有设置索引是会添加一个序号索引写入
# mode参数设置写入csv文件时的方式，w是重写，a是追加写入，默认是重写
# header参数设置布尔值或列表，默认不传递该参数就是True，设置布尔值表示写入csv文件时是否将列名称也一并写入文件，而如果是列表则表示重新新的列名称并且写入到csv文件
score[0:10].to_csv("test_copy.csv",
                   columns=["时间","指标标识符","值"],
                   index=False,
                   mode="a",
                   header=False
                   )
~~~
---

## Pandas 读取 json文件 数据
~~~python
# pip install pandas
import pandas as pd

# read_json函数读取json文件为DataFrame对象
# 第一参数是csv文件路径
# orient参数是设置json的字符串表示类型，有split、records、index、table、columns、values，格式化区别如下
# 'split' : dict like {{'index' -> [index], 'columns' -> [columns],'data' -> [values]}}
# 'records' : list like [{{column -> value}}, ... , {{column -> value}}]
# 'index' : dict like {{index -> {{column -> value}}}}
# 'columns' : dict like {{column -> {{index -> value}}}}
# 'values' : just the values array
# 'table' : dict like {{'schema': {{schema}}, 'data': {{data}}}}
# lines参数表示是否按行读取json数据，即整个文件内容并不是标准json格式，而是每一行是标准json
score = pd.read_json("test.json", orient="records", lines=True)

print("pandas读取json文件的数据\n", score)

# to_json函数将DataFrame对象数据写入到json文件
# 第一参数是json文件路径
# orient参数和读取时的设置是一样的
# lines参数表示是否显示为一行行的数据，默认是返回一个数组，将表格的每一行当成数组元素并且没有换行，文件相当难看，设置为True后表格的每一行对应一个文件中的一行json对象，但是缺陷是整个文件内容并不是标准json格式
score[0:10].to_json("test_copy.json", orient="records", lines=True)
~~~
---

## Pandas 读取 HDF5（是一种二进制文件） 数据
~~~python
# 读取写入HDF5文件需要安装 tables pip install tables
# pip install pandas
import pandas as pd

# read_hdf函数读取HDF5文件为DataFrame对象
# 第一参数是HDF5文件路径
# key参数设置表格的key，表示读取该key所对应的表格
# mode参数设置读取HDF5文件的方式，有 r、r+、a 等读取方式，默认是 r 读取
score = pd.read_hdf("test.h5",key="table1")

print("pandas读取HDF5文件的数据\n", score)

# to_hdf函数将DataFrame对象数据写入到HDF5（是一种二进制文件）文件
# HDF5文件中可以存储多个DataFrame对象（相当于表），存储DataFrame对象时需要设置key(相当于表的名称)
# 第一参数是HDF5文件路径
# key参数设置表格的key，表示将DataFrame对象作为该key所对应的value存储在HDF5文件中
# 注意如果HDF5文件只是存储一个DataFrame对象是可以不用设置key的，但是一旦存储多个DataFrame对象后读取时必须指定key读取对应的DataFrame对象
score[0:10].to_hdf("test_copy.h5", key="table1")
~~~
---

# Pandas 缺失值处理
### 方式一：删除缺失值
### 方式二：填补缺失值，使用平均值、中位数或者是其他数值填补
~~~python
import io

# pip install numpy
import numpy as np

# pip install pandas
import pandas as pd

data_str = '''姓名,年龄,体重,身高
张三,25,51.2,165
李四,30,?,160
王五,35,55.8,169
张六,18,?,162
李七,26,60.3,170
王八,17,,163
张九,19,66.2,172
李十,29,,180
'''

# 这里是直接模拟的数据，也可以从csv文件读取数据
score = pd.read_csv(io.StringIO(data_str), encoding='utf-8')

print("原始csv文件的数据包含?的标记值和NaN的缺失值\n", score)

# 处理标记值?，将标记值替换为Nan
# replace()函数可以进行元素的替换
# to_replace参数设置原始值，也就是这里的标记值
# value参数设置新的值
score = score.replace(to_replace='?', value=np.nan)
print("将标记值?处理为NaN\n", score)

# isnull()和notnull()都可以用于判断是否存在缺失值，返回的是一组布尔列表
if np.any(score.isnull()):
    # 方法一：删除缺失值
    # dropna()函数删除含有缺失值的行或者是列
    # axis参数设置0则删除含有缺失值的行，设置为1删除列，默认是0
    score1 = score.dropna(axis=0)
    print("注意dropna()函数不会改变原有数据，而是返回删除处理后的新数据\n", score1)

    # 方法二：填补缺失值
    # fillna()函数用于填补缺失值
    # value参数是填补值，这里写的是固定值，需要平均值之类的请自行计算平均值替换
    # inplace参数表示是否修改原有数据
    score2 = score.fillna(value=0, inplace=False)
    print("填补处理后的新数据\n", score2)

~~~
---

# Pandas 数据离散化，即数据分组和转换为one-hot编码的稀疏矩阵
~~~python
# pip install pandas
import pandas as pd

score = pd.Series([152, 167, 153, 168, 153, 172, 167, 175, 173])
print("原始数据\n", score)

# qcut()函数自动分组
# 第一参数是数据
# 第二参数是设置数据分成多少组
score1 = pd.qcut(score, 3)
print("自动分组后的数据，显示的是对应行的索引和数据所在的区间\n", score1)
print("自动分组后区间内数据统计数\n", score1.value_counts())

# get_dummies()函数将分组列表转换为 one-hot编码，one-hot编码就是用0和1(或者是布尔值)表示的稀疏矩阵
# 第一参数是数据
# 第二参数是设置分组区间的前缀
score2 = pd.get_dummies(score1, prefix="身高区间")
print("自动分组后列表转换为 one-hot编码\n", score2)

# qcut()函数自定义分组
# 第一参数是数据
# 第二参数是列表，列表中的两个元素的区间就是一个组的区间
score3 = pd.cut(score, [150, 160, 170, 180])
print("自定义分组后的数据，显示的是对应行的索引和数据所在的区间\n", score3)
print("自定义分组后区间内数据统计数\n", score3.value_counts())
score4 = pd.get_dummies(score3, prefix="身高区间")
print("自定义分组后列表转换为 one-hot编码\n", score4)
~~~
---

# Pandas 的数据合并
## 方式一：方向合并数据
~~~python
import io

# pip install pandas
import pandas as pd

data_str1 = '''姓名,年龄,体重
张三,25,51.2
李四,30,61.4
王五,35,55.8
'''

data_str2 = '''姓名,年龄,体重
李七,26,60.3
王八,17,49.8
张九,19,66.2
'''

data_str3 = '''姓名,身高
张三,165
李四,160
王五,169
李七,170
王八,163
张九,172
'''

score1 = pd.read_csv(io.StringIO(data_str1), encoding='utf-8')
score2 = pd.read_csv(io.StringIO(data_str2), encoding='utf-8')
score3 = pd.read_csv(io.StringIO(data_str3), encoding='utf-8')
print("原始数据1\n", score1)
print("原始数据2\n", score2)
print("原始数据3\n", score3)

# concat()函数进行拼接数据
# 第一参数是列表，表示需要拼接的表格数据
# axis参数表示水平还是垂直拼接，0表示水平，1表示垂直
# 注意水平拼接后行索引可能重复，而垂直拼接前需要保证每个数据自身的行索引不能重复
score4 = pd.concat([score1, score2], axis=0)
print("水平拼接数据\n", score4)

score5 = pd.concat([score3.set_index("姓名"), score4.set_index("姓名")], axis=1)
print("垂直拼接数据\n", score5)
~~~
---

## 方式二：索引合并数据
~~~python
import io

# pip install pandas
import pandas as pd

data_str1 = '''姓名1,年龄,体重
张三,25,51.2
李四,30,61.4
王五,35,55.8
'''

data_str2 = '''姓名,年龄,体重
李七,26,60.3
王八,17,49.8
张九,19,66.2
'''

data_str3 = '''姓名2,身高
张三,165
李四,160
王五,169
李七,170
王八,163
张九,172
'''

score1 = pd.read_csv(io.StringIO(data_str1), encoding='utf-8')
score2 = pd.read_csv(io.StringIO(data_str2), encoding='utf-8')
score3 = pd.read_csv(io.StringIO(data_str3), encoding='utf-8')
print("原始数据1\n", score1)
print("原始数据2\n", score2)
print("原始数据3\n", score3)

# merge()函数通过索引拼接数据
# left参数设置左表数据
# right参数设置右表数据
# how参数设置表连接方式，left、right、inner、outer分别代表左连接、右连接、内连接、外连接。默认是内连接
# on参数设置左右表都有的列名列表作为表格连接依据
# 如果左右表的列名不一致需要使用left_on和right_on分别指定不同表格的列名作为连接依据
score4 = pd.merge(left=score1, right=score3, how="inner",
                  left_on="姓名1",
                  right_on="姓名2",
                  # on=["姓名"]
                  )
print("通过索引拼接数据\n", score4)
~~~
---

# Pandas 分组与聚合
~~~python
import io

# pip install pandas
import pandas as pd

data_str = '''姓名,年龄,喜好颜色,身高
张三,16,red,165
李四,16,green,168
王五,16,green,157
李七,17,black,152
王八,16,black,172
张九,17,red,176
'''

score1 = pd.read_csv(io.StringIO(data_str), encoding='utf-8')

print("原始数据\n", score1)

# DataFrame对象直接分组再聚合
# groupby()函数进行分组，by参数指定需要分组的列，as_index参数表示是否将进行分组参考的列在函数调用结果返回时作为索引
print(score1.groupby(by=["喜好颜色"], as_index=False)["身高"].max())

# Series对象先聚合再分组
print(type(score1["身高"]))
print(score1["身高"].groupby(score1["喜好颜色"]).max())
~~~
---