~~~python
# 需要导入json
import json

images = [
    {"name": "沙漠皇帝阿兹尔.jpg",
     "url": "https://game.gtimg.cn/images/lol/act/img/skinloading/41d4cdd8-0f29-4fd7-a849-2b8f478da4b5.jpg"},
    {"name": "疾风剑豪亚索.jpg",
     "url": "https://game.gtimg.cn/images/lol/act/img/skinloading/23b8e9d6-e81c-47fa-9b96-491dd2d1f2d2.jpg"},
    {"name": "青钢影卡蜜尔.jpg",
     "url": "https://game.gtimg.cn/images/lol/act/img/skinloading/1418505c-d1fe-4deb-9566-03856ee618cc.jpg"}
]
# 将对象序列化为json字符串使用json库的dumps() 函数，注意如果有中文请使用ensure_ascii=False参数，否则中文会转换为utf-8编码
json_str = json.dumps(images, ensure_ascii=False)
print(type(json_str))
print(json_str)


l_str = '["Java", "JavaScript", "C", "C++", "Python"]'
# 将json字符串反序列化为对象使用json库的loads() 函数，默认会将json字符串转换为列表或者是字典dict
lst = json.loads(l_str)
print(type(lst))
print(lst)

~~~
---