# lxml库简介
xpath是在xml中搜索内容的一门语言，而html是xpath的子集，通过lxml可以定位搜索html元素

### 使用xpath定位搜索html元素
~~~python
# 安装lxml库 pip install -i https://pypi.tuna.tsinghua.edu.cn/simple lxml
from lxml import etree

xml="""
<book>
    <price>风暴之怒 迦娜</price>
    <nick>众星之子 索拉卡</nick>
    <author>
        <ul>
            <li><nick class="上单">海洋之灾 普朗克</nick></li>
            <li><nick class="打野">永猎双子 千珏</nick></li>
            <li><nick class="中单">离群之刺 阿卡丽</nick></li>
        </ul>
        <div>
            <nick>放逐之刃 锐雯</nick>
        </div>
        <span>
            <nick>暮光星灵 佐伊</nick>
        </span>
    </author>
    <partner>
        <nick>诺克萨斯统领 斯维因</nick>
    </partner>
</book>
"""

# 加载xml文本
tree = etree.XML(xml)

# 通过标签节点查找元素，末尾添加/text()获取元素的内容
print(tree.xpath("/book/price/text()"))

# 通过标签节点查找元素，如果出现 * 节点表示任意类型标签的一个节点
print(tree.xpath("/book/author/*/nick/text()"))

# 通过标签节点查找元素，不写的节点则按照前后节点匹配元素，中间的是任意类型标签的0个或多个节点
print(tree.xpath("/book//nick/text()"))

# 通过标签节点查找元素，通过索引获取节点，但是注意索引是从1开始，而不是和数组一样的0开始
print(tree.xpath("/book/author/ul/li[1]/nick/text()"))

# 通过标签节点查找元素，通过元素的属性值获取节点
print(tree.xpath("/book/author/ul/li/nick[@class='打野']/text()"))

# 通过标签节点查找元素，末尾添加/text()获取元素内的属性
print(tree.xpath("/book/author/ul/li/nick/@class"))

# 获取多个元素，遍历元素继续通过绝对路径获取内部元素
lst = tree.xpath("/book/author/ul/li")
for i in lst:
    print(i.xpath("./nick/text()"))

~~~