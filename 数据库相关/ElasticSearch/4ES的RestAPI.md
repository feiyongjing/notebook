# ES的RestAPI
注意url地址中的类型名称在ES7之后被废弃了，可以使用_doc占位，例如/test1/_doc/1，对所有使用的Rest ful api请求的路径都可以使用_doc替换具体的类型
~~~
method      url地址                                                 描述
PUT         localhost:9200/索引名称/类型名称/文档id                   创建更新文档(指定文档id)                                
POST        localhost:9200/索引名称/类型名称                          创建文档(随机文档id)                       
POST        localhost:9200/索引名称/类型名称/文档id/_updateid         修改文档                                         
DELETE      localhost:9200/索引名称/类型名称/文档id                   删除文档                              
GET         localhost:9200/索引名称/类型名称/文档id                   通过文档id查询文档                            
POST        localhost:9200/索引名称/类型名称/_search                  查询所有数据                                 
~~~

### 使用RestAPI创建索引、文档（可以直接同时一起创建）
~~~
# 放入的文档数据类型如果没有提前确定那么ES会自动的进行猜测默认配置类型然后存放文档数据
PUT /test1/_doc/1
{
  "name":"大数据梦想家",
  "age":21
}

# 响应返回
{
  "_index" : "test1",   # 创建的索引
  "_type" : "type1",    # 类型
  "_id" : "1",          # 文档ID
  "_version" : 1,       # 版本号，修改文档会增加版本号
  "result" : "created", # Rest风格操作类型
  "_shards" : {         # 分片信息
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
~~~

### elasticsearch 常见的字段类型
- 字符串类型：text、keyword
- 数值类型：long、integer、short、byte、double、float、half_float、scaled_float
- 日期类型：date
- 布尔值类型：boolean
- 二进制类型：binary
- 其他类型

### 新建一个索引并指定字段类型，然后后续就可以向里面放数据了
- "number_of_shards": "3"        分片数
- "number_of_replicas": "2"      副本数
- "analyzer": "ik_max_word"      存入文档数据时选择用 ik_max_word （细粒度）类型IK分词器进行拆分文本然后倒排索引
- "search_analyzer": "ik_smart"  搜索文档数据时选择用 ik_smart（粗粒度）类型IK分词器对 用户输入的搜索关键词 做粗粒度的拆分，只拆分出最核心、最精准的词语，目的是 “减少冗余匹配，提升搜索精准度
~~~
PUT /test2
{
  "settings": {
    "number_of_shards": "3",
    "number_of_replicas": "2"
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",    
        "search_analyzer": "ik_smart" 
      },
      "content": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      },
      "createTime": {"type": "date"},
      "author": {"type": "keyword"}
    }
  }
}

# 返回响应如下创建索引成功
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test2"
}
~~~

### 查询索引、数据
~~~
# 查询指定索引的信息
GET /test2

# 响应索引的信息
{
  "test2" : {
    "aliases" : { },
    "mappings" : {           # 索引内的字段名称和类型
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "birthday" : {
          "type" : "date"
        },
        "name" : {
          "type" : "text"
        }
      }
    },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "test2",
        "creation_date" : "1769801455112",
        "number_of_replicas" : "1",
        "uuid" : "HEZkOT3xQk-C9YtobSsS5w",
        "version" : {
          "created" : "7172999"
        }
      }
    }
  }
}


# 查询指定ID文档的数据
GET /test1/_doc/1

# 响应文档数据
{
  "_index" : "test1",
  "_type" : "type1",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "大数据梦想家",
    "age" : 21
  }
}
~~~

### 编辑更新文档
~~~
# 方式一（不建议使用）：基本和创建文档没有什么变化，可以对指定文档ID进行修改文档内的属性，但是有缺点，不需要修改的数据也要在原有的请求体中存在，如果修改是漏掉一个属性那么这个属性就会被擦除消失
PUT /test1/_doc/1
{
  "name":"大数据梦想家",
  "age":22
}

# 方式二：请求头有固定的格式，doc属性确定要修改的属性和值就可以了，不会对文档其他的属性造成修改
POST /test1/_doc/1/_update
{
  "doc":{
    "age":28
  }
}
~~~

### 删除索引或文档
~~~
# 删除索引
DELETE /test1

# 删除文档
DELETE /test1/_doc/1
~~~

### 复杂查询
1. 插入5条数据
~~~
PUT /alice/user/1
{
  "name":"爱丽丝",
  "age":21,
  "desc":"在最美的年华，做最好的自己！",
  "tags":["技术宅","温暖","思维活跃"]
}

PUT /alice/user/2
{
  "name":"张三",
  "age":23,
  "desc":"法外狂徒",
  "tags":["渣男","交友"]
}

PUT /alice/user/3
{
  "name":"路人甲",
  "age":24,
  "desc":"不可描述",
  "tags":["靓仔","网游"]
}

PUT /alice/user/4
{
  "name":"爱丽丝学Java",
  "age":25,
  "desc":"技术成就自我！",
  "tags":["思维敏捷","喜欢学习"]
}

PUT /alice/user/5
{
  "name":"爱丽丝学Python",
  "age":26,
  "desc":"人生苦短，我用Python！",
  "tags":["好学","勤奋刻苦"]
}
~~~

2. 文档内容的简单查询
~~~
GET /alice/user/_search?q=name:张三

# 可以看到查询到张三的文档了，另外在 hits 中有该文档的 _score得分，就是根据算法算出和查询条件匹配度高的分就越高越靠前。
# 但是一般使用不会直接加条件去查询
{
  "took" : 9,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 3.3887196,
    "hits" : [
      {
        "_index" : "alice",
        "_type" : "user",
        "_id" : "2",
        "_score" : 3.3887196,
        "_source" : {
          "name" : "张三",
          "age" : 23,
          "desc" : "法外狂徒",
          "tags" : [
            "渣男",
            "交友"
          ]
        }
      }
    ]
  }
}
~~~

3. 构造请求体进行简单查询
响应主要观察hit，里面有如下信息
- 索引和文档的信息
- 查询的结果总数
- 然后就是查询出来的具体文档
- 数据中的东西都可以遍历出来
- 通过score分数我们可以判断谁更加符合结果
~~~
GET alice/user/_search
{
  "query":{
    "match": {
      "name": "爱丽丝"
    }
  }
}


# 响应如下
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.7353058,
    "hits" : [
      {
        "_index" : "alice",
        "_type" : "user",
        "_id" : "1",
        "_score" : 1.7353058,
        "_source" : {
          "name" : "爱丽丝",
          "age" : 21,
          "desc" : "在最美的年华，做最好的自己！",
          "tags" : [
            "技术宅",
            "温暖",
            "思维活跃"
          ]
        }
      },
      {
        "_index" : "alice",
        "_type" : "user",
        "_id" : "4",
        "_score" : 1.3950496,
        "_source" : {
          "name" : "爱丽丝学Java",
          "age" : 25,
          "desc" : "技术成就自我！",
          "tags" : [
            "思维敏捷",
            "喜欢学习"
          ]
        }
      },
      {
        "_index" : "alice",
        "_type" : "user",
        "_id" : "5",
        "_score" : 1.3950496,
        "_source" : {
          "name" : "爱丽丝学Python",
          "age" : 26,
          "desc" : "人生苦短，我用Python！",
          "tags" : [
            "好学",
            "勤奋刻苦"
          ]
        }
      }
    ]
  }
}
~~~

4. 只查询文档的部分字段数据
~~~
# 匹配name有爱丽丝，结果截取其他字段只获取name和desc字段的数据
GET alice/user/_search
{
  "query":{
    "match": {
      "name": "爱丽丝"
    }
  },
  "_source":["name","desc"]
}
~~~

5. 布尔查询
类似sql的and、or、not，and请使用must，or请使用should，not请使用must_not
~~~
# 查询name为alice，并且age是25岁的数据
GET alice/user/_search
{
  "query":{
    "bool": {
      "must":[
        {
          "match":{
            "name":"爱丽丝"
          }
        },
        {
          "match":{
            "age":25
          }
        }
      ]
    }
  }
}

# 查询name为alice，或者age是25岁的数据
GET alice/user/_search
{
  "query":{
    "bool": {
      "should":[
        {
          "match":{
            "name":"爱丽丝"
          }
        },
        {
          "match":{
            "age":25
          }
        }
      ]
    }
  }
}

# 查询age不是25岁的数据
GET alice/user/_search
{
  "query":{
    "bool": {
      "must_not":[
        {
          "match":{
            "age":25
          }
        }
      ]
    }
  }
}
~~~

6. 数值范围查询
filter 条件过滤查询，过滤条件的范围用 range 表示，其余操作如下:
- gt 表示大于
- gte 表示大于等于
- lt 表示小于
- lte 表示小于等于
~~~
# 查询 name 为爱丽丝，age 在24到26之间的数据，需要使用到filter进行过滤
GET alice/user/_search
{
  "query":{
    "bool":{
      "must": [
        {
          "match": {
            "name": "爱丽丝"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "age": {
              "gte": 24,
              "lte": 26
            }
          }
        }
      ]
    }
  }
}
~~~

7. 短语检索
类似sql的in，但是是模糊匹配，并且是两数组中的元素模糊匹配
~~~
# 新增两条数据
PUT /alice/user/6
{
  "name":"大数据老K",
  "age":25,
  "desc":"技术成就自我！",
  "tags":["男","学习","技术"]
}

PUT /alice/user/7
{
  "name":"Python女侠",
  "age":26,
  "desc":"人生苦短，我用Python！",
  "tags":["靓女","勤奋学习","善于交际"]
}

# 只要有一个元素模糊匹配就满足
GET alice/user/_search
{
  "query":{
    "match":{
      "tags":"男 学习"
    }
  }
}
~~~

8. 排序查询
注意:在排序的过程中，只能使用可排序的属性进行排序。只有数字、日期、ID可以进行排序
~~~
# 匹配name有爱丽丝，并根据age字段倒序查询（正序将desc更换为asc）
GET alice/user/_search
{
  "query":{
    "match": {
      "name": "爱丽丝"
    }
  },
  "sort": [
    { 
      "age": 
      { 
        "order": "desc"
        }
    
     }
   ]
}
~~~

9. 分页查询
~~~
# 表示从第from条开始查询出来size条数据
GET alice/user/_search
{ 
  "query":{
    "match": {
      "name": "爱丽丝"
    }
  },
  "from":0, 
  "size":2   
}
~~~

10. 精确匹配
- term：不经过分词，直接查询精确的值
- match：会使用分词器解析的模糊查询！（先分析文档，然后再通过分析的文档进行查询！）
但是请注意keyword类型不会被分析器处理，验证如下
~~~
# 创建一个索引，并指定类型
PUT testdb
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text"
      },
      "desc":{
        "type":"keyword"
      }
    }
  }
}

# 插入数据
PUT testdb/_doc/1
{
  "name":"爱丽丝学大数据name",
  "desc":"爱丽丝学大数据desc"
}

PUT testdb/_doc/2
{
  "name":"爱丽丝学大数据name2",
  "desc":"爱丽丝学大数据desc2"
}

# text 类型虽然会被分析器分析查询，但是这里选择的term精确匹配所有只会查询到第一条数据
GET testdb/_search         
{
  "query": {
    "term": {
        "desc": "爱丽丝学大数据desc"
      }
    }
}

#  虽然match是模糊匹配会被分析器分析查询，但是keyword 类型不会被分析所以直接查询，所以只会查询到第一条数据
GET testdb/_search         
{
  "query": {
    "match": {
        "desc": "爱丽丝学大数据desc"
      }
    }
}
~~~

11. 高亮显示
~~~
GET alice/user/_search
{
  "query":{
     "match": {
       "name": "爱丽丝"
     }
  },
  "highlight":{
    "fields": {
      "name": {}
    }
  }
}

# 在响应的 highlight 中的属性添加了高亮标签<em></em>
{
  "took" : 148,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.7353058,
    "hits" : [
      {
        "_index" : "alice",
        "_type" : "user",
        "_id" : "1",
        "_score" : 1.7353058,
        "_source" : {
          "name" : "爱丽丝",
          "age" : 21,
          "desc" : "在最美的年华，做最好的自己！",
          "tags" : [
            "技术宅",
            "温暖",
            "思维活跃"
          ]
        },
        "highlight" : {
          "name" : [
            "<em>爱</em><em>丽</em><em>丝</em>"
          ]
        }
      }
    ]
  }
}


# 自定义设置高亮样式
GET alice/user/_search
{
  "query":{
     "match": {
       "name": "爱丽丝"
     }
  },
  "highlight":{
    "pre_tags": "<b class='key' style='color:red'>", 
    "post_tags": "</b>",
    "fields": {
      "name": {}
    }
  }
}

# 在响应的 highlight 中的属性添加了自定义的高亮标签<b class='key' style='color:red'></b>
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.7353058,
    "hits" : [
      {
        "_index" : "alice",
        "_type" : "user",
        "_id" : "1",
        "_score" : 1.7353058,
        "_source" : {
          "name" : "爱丽丝",
          "age" : 21,
          "desc" : "在最美的年华，做最好的自己！",
          "tags" : [
            "技术宅",
            "温暖",
            "思维活跃"
          ]
        },
        "highlight" : {
          "name" : [
            "<b class='key' style='color:red'>爱</b><b class='key' style='color:red'>丽</b><b class='key' style='color:red'>丝</b>"
          ]
        }
      }
    ]
  }
}
~~~


### 其他API
~~~
# 获取集群的一个健康状态，心态检查
GET /_cat/health

# 获取到当前索引的很多信息，返回值包括所有索引的状态健康情况，分片，数据储存大小等等
GET /_cat/indices?v
~~~







