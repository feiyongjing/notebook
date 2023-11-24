~~~shell
# 默认显示文件的 行数 单词数 字节数 文件名称 
wc [文件] 
# -l 参数只显示行 和 文件名称 
# -w 参数只显示单词数 和 文件名称 
# -c 参数只显示字节数 和 文件名称 
~~~

~~~shell
# jq命令查询json，需要提前安装jq命令，yum install jq -y 或者 apt install jq -y
jq -r ".data.records[0].messageContent" test.json
# -r 选项表示输出原始数据，即不添加任何格式化或转义符号，这通常用于将 JSON 格式数据转换为其他格式的数据，如文本、CSV、XML 等
# 以下是测试使用的 test.json文件的内容
{
    "code": "00000",
    "data": {
        "records": [
            {
                "gmtCreate": "2023-11-23 20:02:48",
                "gmtModified": "2023-11-23 20:02:48",
                "id": "1727658957061132289",
                "messageContent": "xxx新增了用户：自动化测试",
                "userId": "1533681422289534978",
            }
        ],
    },
    "msg": "一切ok"
}
~~~