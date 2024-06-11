# CSS简介
- CSS的全称是层叠样式表，是一种标记语言，用来给HTML结构设置样式，例如字体大小、背景颜色、元素宽高等等

### CSS特性
- 层叠性：如果发生了样式冲突就按照一定的规则（选择器的优先级和先后顺序），进行样式覆盖
- 继承性：元素会继承父元素和祖元素的上设置的样式（例如 font-??、line-??、color等），优先继承离的近的元素样式（当然有些样式是无法继承的）
- 优先级：css的样式属性值后添加!important（注意只是改一个属性值，而不是整个选择器） > 行内样式 > id选择器 > 类选择器 > 元素选择器 > 通配选择器 > 元素标签默认样式 >继承的样式，当然复杂的选择器还是需要根据权重计算优先级

### css的语法
~~~
选择器 {
    样式属性名称: 样式属性值
}
~~~

# css的编写位置
- 行内样式：直接在html标签的sytle属性上编写css样式，编写的样式只对当前标签有效，无法复用样式
- 内部样式：在html的style标签中编写css样式，编写选择器选择标签和样式，对选中的标签样式生效，只能对当前html页面复用样式，其他html页面无法复用样式
- 外部样式：在html的link标签引入.css文件，该文件中编写选择器选择标签和样式，对选中的标签样式生效，可以在多个html页面复用样式，推荐使用

### 行内样式
~~~html
<p style="color:red;font-size:40px">哈哈哈</p>
~~~

### 内部样式，注意按照HTML的标准规范style标签应该是需要放在head标签中
~~~html
<!DOCTYPE html> 
<html lang="zh-CN"> 
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width-device-width, initial-scale=1.0"> 
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    <title>考试系统</title>

    <!-- css样式代码 -->
    <style>
        h1{
            color:red;
            font-size:60px;
        }
        p{
            color:green;
            font-size:40px;
        }
    </style>
</head>
<body>
    <h1>css内部样式</h1>
    <p>哈哈哈</p>
</body>
</html>
~~~

### 外部样式，注意按照HTML的标准规范link标签应该是需要放在head标签中
html和css文件如下(注意它们之间的路径)
~~~html
<!DOCTYPE html> 
<html lang="zh-CN"> 
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width-device-width, initial-scale=1.0"> 
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    <title>考试系统</title>
</head>
<body>
    <!-- 引入外部链接资源
     rel属性设置链接方式与html文档之间的关系，stylesheet表示是样式表
     href属性设置外部资源的路径
    -->
    <link rel="stylesheet" href="../css/test.css">

    <h1>css内部样式</h1>
    <p>哈哈哈</p>
</body>
</html>
~~~
~~~css
h1{
    color:red;
    font-size:60px;
}
p{
    color:green;
    font-size:40px;
}
~~~
---

### 行内样式、内部样式、外部样式在相同的样式上冲突之后的生效情况
- 行内样式比内部样式、外部样式的优先级都高
- 内部样式和外部样式是同样的优先级，但是会按照html的解析顺序生效，即后面设置的样式会覆盖前面设置的样式
- 同一种样式是按照html的解析顺序生效（例如两个内部样式冲突），即后面设置的样式会覆盖前面设置的样式

### CSS页面编写
~~~html
<style>
    .test{
        font-size: 0px;
    }

    .test1-1 {
        font-size: 20px;
        width: 200px;
        height: 80px;
        background-color: yellowgreen;
        display: inline-block;
    }

    .test1-2 {
        font-size: 20px;
        width: 540px;
        height: 80px;
        background-color: yellowgreen;
        display: inline-block;
        margin: 0px 10px;
    }

    .test1-3 {
        font-size: 20px;
        width: 200px;
        height: 80px;
        background-color: yellowgreen;
        display: inline-block;
    }

    .test2 {
        font-size: 20px;
        width: 960px;
        height: 30px;
        background-color: red;
        margin-top: 10px;
    }

    .test3-1 {
        display: inline-block;
        margin-top: 10px;
    }

    .test3-1-1 {
        font-size: 20px;
        width: 370px;
        height: 200px;
        background-color: violet;
    }

    .test3-1-2 {
        font-size: 20px;
        width: 180px;
        height: 200px;
        background-color: violet;
        display: inline-block;
        margin-top: 10px;
        margin-right: 10px;
    }

    .test3-1-3 {
        font-size: 20px;
        width: 180px;
        height: 200px;
        background-color: violet;
        display: inline-block;
        margin-top: 10px;
    }

    .test3-2 {
        display: inline-block;
        margin: 0px 10px;
        margin-top: 10px;
    }

    .test3-2-1 {
        font-size: 20px;
        width: 370px;
        height: 200px;
        background-color: green;
    }

    .test3-2-2 {
        font-size: 20px;
        width: 180px;
        height: 200px;
        background-color: green;
        display: inline-block;
        margin-top: 10px;
        margin-right: 10px;
    }

    .test3-2-3 {
        font-size: 20px;
        width: 180px;
        height: 200px;
        background-color: green;
        display: inline-block;
        margin-top: 10px;
    }

    .test3-3 {
        display: inline-block;
    }

    .test3-3-x {
        font-size: 20px;
        width: 200px;
        height: 130px;
        background-color: palegreen;
        margin-top: 10px;
    }

    .test4 {
        font-size: 20px;
        width: 960px;
        height: 60px;
        background-color: slateblue;
        margin-top: 10px;
    }
</style>

<div class="test">
    <div class="test1">
        <div class="test1-1"></div>
        <div class="test1-2"></div>
        <div class="test1-3"></div>
    </div>
    <div class="test2"></div>
    <div class="test3">
        <div class="test3-1">
            <div class="test3-1-1"></div>
            <div class="test3-1-2"></div>
            <div class="test3-1-3"></div>
        </div>
        <div class="test3-2">
            <div class="test3-2-1"></div>
            <div class="test3-2-2"></div>
            <div class="test3-2-3"></div>
        </div>
        <div class="test3-3">
            <div class="test3-3-x"></div>
            <div class="test3-3-x"></div>
            <div class="test3-3-x"></div>
        </div>
    </div>
    <div class="test4"></div>
</div>
~~~

