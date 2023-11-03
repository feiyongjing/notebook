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

# DataFrame 对象和 Series 对象的统计运算
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