# IK 分词器
分词：即把一段中文或者别的内容划分成一个个的关键字，我们在搜索时候会把自己的信息进行分词，是因为数据库中或者索引库中的数据也会进行分词，然后进行一个匹配操作，默认的中文分词是将每个字看成一个词，比如 “我爱大数据” 会被分为"我"，"爱"，"大", "数"，"据"，这显然是不符合要求的，所以我们需要安装中文分词器 ik 来解决这个问题



## IK分词器安装
### 方式一：在线安装
~~~
# 使用ES安装目录下bin目录中的elasticsearch-plugin 进行安装插件
# 核心插件直接通过插件名称安装
elasticsearch-plugin install [plugin_name]
# 其他的插件可以通过插件链接安装
elasticsearch-plugin install https://get.infini.cloud/elasticsearch/analysis-ik/9.1.4

# 查看安装的插件列表
elasticsearch-plugin list

# 卸载插件
elasticsearch-plugin remove [pluginname]
~~~

### 方式二：离线安装
~~~
在 https://release.infinilabs.com/ 中找到对应版本的IK插件下载然后解压缩
在ES安装目录下的插件目录plugins创建新的目录ik，将下载解压的内容全部复制进去，然后重启ES
可以在启动日志中看到 loaded plugin [analysis-ik] 加载插件

# 查看安装的插件列表
elasticsearch-plugin list
~~~

## 使用IK分词器
IK提供了两个分词算法：ik_smart 和 ik_max_word ，其中 ik_smart 为最少切分，ik_max_word 为 最细粒度划分！
- ik_smart 是粗粒度分词，优先匹配最长词，不会有重复的数据
- ik_max_word 是细粒度分词，会穷尽一个语句中所有分词可能

在kibana 界面或者使用其他方式发起http请求
### ik_smart粗粒度分词请求
~~~
GET _analyze
{
  "analyzer":"ik_smart",
  "text": "百科商城墙"
}

# 响应只是将段落直接拆分成词语，其中的所有文字在上下文中只能使用一次
{
  "tokens" : [
    {
      "token" : "百科",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "商",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "城墙",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    }
  ]
}
~~~

### ik_max_word细粒度分词请求
~~~
GET _analyze
{
  "analyzer":"ik_max_word",
  "text": "百科商城墙"
}

# 响应提取段落中所有可以拆分成的词语，其中的所有文字在上下文中可以多次出现
{
  "tokens" : [
    {
      "token" : "百科",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "百",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "TYPE_CNUM",
      "position" : 1
    },
    {
      "token" : "科",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "COUNT",
      "position" : 2
    },
    {
      "token" : "商城",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "城墙",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 4
    }
  ]
}
~~~

### 添加自定义字典
在一些场景下有一些特殊词语是需要独立拆分的，但是ik_smart和ik_max_word都无法拆分出来，这是因为使用的默认字典，而这些特殊词语不在默认字典中
在这种特殊词语需要拆分的场景可以额外引入自定义的字典
1. 在elasticsearch/plugins/ik/config目录下有默认的dic字典文件，可以新建自定义的dic字典文件，例如my.dic，编辑内容添加特殊词语，例如“三国杀”
2. 修改elasticsearch/plugins/ik/config目录下的IKAnalyzer.cfg.xml
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">my.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
~~~
3. 修改完配置重新启动 elasticsearch，再次测试
~~~
GET _analyze
{
  "analyzer":"ik_smart",
  "text": "三国杀"
}

# 可以看出直接是一个词语“三国杀”，而不是之前拆分成“三国”和“杀”
{
  "tokens" : [
    {
      "token" : "三国杀",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    }
  ]
}
~~~




