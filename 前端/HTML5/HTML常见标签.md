# html标签元素如下，官方文档推荐看mdn
~~~
DOCTYPE:  文档类型声明
html：    html标签表示内部标签按照html解析
head：    html头部标签
meta：    html元数据标签
titel：   网页标签
body:     html主体标签
 
h1~h6：   标题标签
p：       段落标签
br：      换行标签
hr：      分割线标签

span：    行标签
div：     块标签
pre：     原文显示标签，空格和换行都按照原文显示
 
a:        链接标签
img：     图片标签
audio：   音频标签
vidio:    视频标签

ul:       无序列表标签
ol:       有序列表标签
li:       列表行标签
dl:       自定义列表标签
dt:       自定义列表标题标签
dd:       自定义列表段落标签

table:    表格标签
caption:  表格标题标签
thead:    表格头部标签
tbody:    表格主体标签
tfoot:    表格尾部标签
tr:       表格行标签
th:       表格头部单元格标签
td:       表格主体单元格标签

form：    表单标签
input：   表单输入标签
button：  按钮标签
textarea：多行文本输入框标签
select：  下拉框标签
option：  下拉框选项标签
label：   表单输入焦点绑定标签：包裹input输入标签控件，实现点击显示的文本与输入框关联获取焦点

iframe:   这个标签可以用来在网站中嵌入一个其他的网页，有些网站就是使用这个嵌入的广告

b:        字体加粗标签
strong:   字体加粗标签
i:        字体倾斜标签
em：      字体倾斜标签
u：       字体下滑线标签
del:      字体删除线标签
s：       字体删除线标签
sup:      字体上标标签，例如2的3次方
sub:      字体下标签，例如氧气的化学表示
~~~

# html标签常用的全局属性，即所有标签都有的属性
~~~
id：      标签的唯一标识，同一html中不能与其他标签的id重复，后续可以通过id对对应的标签设置css样式
class:    标签指定的类名，后续可以通过对同一类名的标签设置css样式
sytle：   对标签设置css样式
dir：     设置标签中内容的方向，值可以设置为ltr或rtl
title:    设置鼠标放在标签上时悬浮的内容
~~~

# VScode安装LIveServer插件
Live Server是一个具有实时加载功能的小型服务器，可以使用它来破解html/css/javascript，但是不能用于部署最终站点。也就是说我们可以在项目中实时用live-server作为一个实时服务器实时查看开发的网页或项目效果
安装使用参考：https://blog.csdn.net/weixin_43272781/article/details/103875811
- 注意默认是占用的5500端口
- 注意如果VScode直接打开文件，而不是文件夹，则右键Open with Live Server 无法正常工作
- 注意如果你的html不符合规范，即没有<html> <head> <body>这些标签则Live Server启动之后修改代码保存浏览器不会自动刷新

# 常用标签使用案例
~~~html
<!DOCTYPE html> <!-- 文档类型声明必须写在第一行，这是html5的声明，其他的文档类型声明要比这个麻烦 -->

<html lang="zh-CN"> <!-- 页面主要语言类型，这样浏览器可以进行提供对应语言的翻译服务，zh-CN是简体中文、en是英文 -->

<!-- 文档头部标签，通常是网页在的标签 -->

<head>
    <!-- meta标签声明文档的元数据 -->
    <meta charset="utf-8"> <!-- meta标签中charset属性设置文档的字符编码 -->
    <meta name="viewport" content="width-device-width, initial-scale=1.0"> <!-- 使手机端和pc端的样式一致 --> 
    <meta http-equiv="x-ua-compatible" content="ie=edge"> <!-- 针对IE浏览器的兼容性设置 -->

    <!-- <meta name="keywords" content="网上购物,电商购物,电子产品"> 设置网页关键字，方便搜索引擎SEO  -->
    <!-- <meta name="description" content="购物商城描述"> 设置网页内容描述  -->
    <!-- <meta name="robots" content="index"> 设置网页是否可以被搜索引擎索引收录，要求不允许收录使用noindex  -->

    <title>考试系统</title> <!-- 在head标签中的title标签设置网页的标题 -->
</head>

<!-- 文档z主体标签 -->

<body>

    <!-- css样式代码 -->
    <!-- <style></style> -->

    <!-- js代码 如果有src属性就是引入外部的js文件代码-->
    <!-- <script src="../js/hello.js"></script> -->

    <!-- 引入外部链接资源，
     rel属性设置链接方式与html文档之间的关系，stylesheet表示是样式表
     href属性设置外部资源的路径
    -->
    <!-- <link rel="stylesheet" href="../css/selectorType.css"> -->

    <!-- h1到h6是标题标签 -->
    <h1></h1>

    <!-- 段落标签 -->
    <p></p>

    <b>加粗1</b>
    <strong>加粗2</strong>
    <i>倾斜1</i>
    <em>倾斜2</em>
    <u>下滑线</u>
    <del>删除线1</del>
    <s>删除线2</s>
    上标，用于显示2的多少次方，例子：2<sup>10</sup>
    下标，用于显示氧气等，例子：O<sub>2</sub>

    <!-- 换行标签 -->
    <br />
    <!-- 分割线标签 -->
    <hr />

    <!-- 原文显示标签，空格和换行都原文显示 -->
    <pre></pre>

    <!-- 行标签 -->
    <span></span>

    <!-- 块标签 -->
    <div></div>

    <!-- 链接标签：
        href属性设置链接(可以是内部html路径或者是外部网址)，
        如果href属性设置的链接中包含#则表示跳转到指定锚点，#后面的字符串是指定的锚点（如果没有就是回到页面的顶部），
        锚点就是a标签的名称或任意标签的id属性，会跳转到这些标签页面
        当然#前面没有字符串就是当前页面的锚点，如果有就是这些其他页面链接的锚点
        href属性还可以唤醒指定的应用，例如tel打电话、mailto发送邮件、sms发送短信，当然前提是电脑有这些应用，不过手机也是可以的
        target属性设置跳转方式，_blank是在新的浏览器页面打开，_self是在当前浏览器页面跳转链接打开，默认是_self
        a标签链接指向的文件如果浏览器可以解析就会直接解析，如果无法解析的文件会进行下载
        download属性设置浏览器下载文件，如果属性值没有设置则使用默认文件名称，如果有属性值就使用属性值作为文件名称
    -->
    <a href="https://baidu.com" target="_blank" download="百度.html">a标签点击文字跳转指定页面</a>
    <a href="tel:10010"></a>
    <a href="mailto:10010@qq.com"></a>
    <a href="sms:10086"></a>

    <!-- 图片链接：src设置图片链接或者图片的base64字符串，alt设置图片无法显示的提示文字，width和heighe设置图片宽高，只设置一个另一个也会等比例设置 -->
    <img src="../img/OIP-C.jfif" alt="图片的路径不对" width="300px" height="180px" title="花">

    <!-- 音频标签audio，支持mp3、ogg(移动端)、wav
     autoplay是自动播放，controls是显示视频的进度条和暂停等功能，
     loop是循环播放，muted是默认静音
    -->
    <audio src="音频路径" autoplay controls loop muted> 音频显示失败时显示这行文字 </audio>

    <!-- 视频标签video，支持mp4、ogg(移动端)、webM(高清格式)
    autoplay是自动播放，width属性值表示宽度，heigth属性值表示高度
    controls是显示视频的进度条和暂停等功能，loop是循环播放
    muted是默认静音，poster属性值设置视频播放之前显示的图片
    -->
    <video src="视频路径" autoplay width="300px" heigth="180px" controls loop muted poster="图片路径"> 视频显示失败时显示这行文字 </video>

    <h3>无序列表</h3>
    <ul>
        <li>无序列表ul+li</li>
        <li>无序列表ul和li的属性type表示列表元素前的图案，属性值有disc、circle、square分别代表黑色实心圆、空心圆、实心矩形</li>
        <li>xxxx</li>
    </ul>

    <h3>有序列表</h3>
    <ol>
        <li>有序列表ol+li</li>
        <li>有序列表ol和li的属性type表示列表元素前的有序模板，属性值有A、a、1、I分别表示大写字母模板、小写字母模板、数字模板、罗马数字模板。start属性表示按照type属性模板的第几个开始，reversed属性无需设置属性值表示模板按照倒序排列
        </li>
        <li>有序列表ol+li</li>
    </ol>

    <h3>自定义列表</h3>
    <dl>
        <dt>自定义列表标题：dl+dt+dd</dt>
        <dd>段落：xxxx</dd>
        <dd>段落：xxxx</dd>
    </dl>

    <h3>表格</h3>
    <!-- border设置表格边框线条粗细
         width和heigth设置表格的宽高，注意调整宽高影响总体的表格宽高（主要是表格主体单元格宽高，表格头和尾的单元格宽高不影响）
         cellspacing设置单元格之间的间距
         align设置单元格内文字水平方向对齐方式（left、center、right）
         valign设置单元格内文字垂直方向对齐方式（top、middle、bottom）
     -->
    <table border="1" width="500" heigth="500" cellspacing="0">
        <caption>表格标题1</caption>

        <!-- 表格头只能写heigth属性调整高度，当高度超过表格标签设置的高度时，表格头的高度确定，而表格的总体高度会更大一些 
            colspan属性设置单元格水平方向占据几个单元格，rowspan属性设置单元格垂直方向占据几个单元格
        -->
        <thead> <!-- 表格头 -->
            <tr>
                <th></th>
                <th>中文</th>
                <th>英语</th>
            </tr>
        </thead>
        <tbody> <!-- 表格主体 -->
            <tr>
                <th></th> <!-- tbody中的tr+th用于显示每行的表头，用于制造双表头的 -->
                <td>红色</td>
                <!-- td中colspan属性表示表格占据的宽度,即与左边的表格合并，rowspan属性表示表格占据的高度即与下方的表格合并，正常的表格的宽高colspan和rowspan都是1 -->
                <td>red</td>
            </tr>
            <tr>
                <th></th>
                <td>绿色</td>
                <td>green</td>
            </tr>
        </tbody>
        <tfoot> <!-- 表格尾 -->
            <tr>
                <th>词汇数</th> <!-- tfoot中的tr+th用于显示每行的表头，用于制造双表头的 -->
                <td>2</td>
                <td>2</td>
            </tr>
        </tfoot>
    </table>
    <hr>

    <!-- form表单标签
         action属性设置提交表单的网址url
         method属性设置请求方法
         target属性设置跳转方式，_blank是在新的浏览器页面打开，_self是在当前浏览器页面跳转链接打开，默认是_self
    -->
    <!-- label标签：包裹input输入标签控件，实现点击显示的文本与输入框关联获取焦点 -->
    <!-- input输入标签
         name属性设置参数的名称
         type属性设置输入的参数类型：有如下参数类型
            text: 单行文本输入框
            password: 密码输入框
            radio: 单选
            checkbox: 多选
            hidden：输入框隐藏
            submit: input会变成提交按钮
            reset: 重置按钮
            file: 文件上传
         placeholder属性设置输入框内的提示信息
         value属性设置默认值，或提交到后台的值，如果是submit,reset的类型时，value就是按钮的名称。
         checked属性设置标记单选框或者多选框的选项被默认选中
         autofocus属性设置自动获取光标
         color: 颜色
         datatime: 日期时间
         week: 周
         month： 月
         disabled属性设置禁用输入和修改
         maxlength属性设置最大长度
    -->
    <!-- textarea多行文本输入框标签
         name属性设置参数的名称
         cols属性设置文本框的宽是多少字符
         rows属性设置文本框的高是多少字符
         disabled属性禁用输入和修改
    -->
    <!-- select下拉框标签
         name属性设置参数的名称
         disabled属性禁用下拉框全部选择
    -->
    <!-- option下拉框选项标签
         value属性设置下拉框提交表单请求真实传递的参数，如果没有设置默认会获取option中的文本作为参数
         selected属性设置下拉框默认的选择选项
         disabled属性禁用下拉框选项选择
    -->
    <!-- button按钮标签
         type属性设置按钮类型
            submit: 默认的提交按钮
            reset: 重置按钮
         value: 按钮框中显示的信息
         disabled属性禁用按钮
    -->
    <form action="https://www.baidu.com/s" >
        <input type="text" name="wd">
        <button>去百度搜索</button>
    </form>
    <hr>
    <form action="https://pifa.pinduoduo.com/search" >
        <input type="text" name="word">
        <button>去拼多多搜索</button>
    </form>
    <hr>
    <form action="xxxx"  >
        <!-- 文本输入框 -->
        <label>用户名称：
            <input type="text" name="username" maxlength="8" placeholder="请输入你的用户名，长度最多是8位">
        </label>
        <br>
        
        <!-- 密码输入框 -->
        <label>用户密码：
            <input type="password" name="password" maxlength="8" placeholder="请输入你的密码，长度最多是8位">
        </label>
        <br>
        
        <!-- 单选框 -->
        性别：
        <label>
            <input type="radio" name="gender" value="male" checked>男
        </label>
        <label>
            <input type="radio" name="gender" value="female">女
        </label>
        <br>

        <!-- 多选框 -->
        爱好：
        <label>
            <input type="checkbox" name="hobby" value="basketball" checked>篮球
        </label>
        <label>
            <input type="checkbox" name="hobby" value="football">足球
        </label>
        <label>
            <input type="checkbox" name="hobby" value="ping pong">乒乓球
        </label>
        <br>
        
        <!-- 多行文本框： -->
        <label>
            其他：
            <textarea name="other" cols="30" rows="3" placeholder="请输入你的其他信息"></textarea>
        </label>
        <br>
        
        <!-- 下拉框 -->
        籍贯：
        <select name="place">
            <option value="河北省">河北</option>
            <option value="南京市">南京</option>
            <option value="河南省">河南</option>
            <option value="北京省" selected>北京</option>
        </select>

        <!-- 隐藏的输入框：最终表单提交发起请求会携带这个输入框的参数 -->
        <input type="hidden" name="abc" value="123" ><br>

        <!-- 提交按钮的第一种写法：注意按钮不要写name属性，并且按钮的默认type属性是submit -->
        <button>用户注册</button><br>
        <!-- 提交按钮的第二种写法：注意按钮不要写name属性 -->
        <!-- <input type="submit" value="用户注册"> -->

        <!-- 重置按钮的第一种写法：注意按钮不要写name属性 -->
        <button type="reset">重置默认值</button><br>
        <!-- 重置按钮的第二种写法：注意按钮不要写name属性 -->
        <!-- <input type="reset" value="重置默认值"> -->

        <!-- 普通的按钮作用是绑定js事件触发对应的事件，一般这些事件都是发起请求
             普通的按钮第一种写法：注意一定要写type属性是button，否则默认submit提交按钮
        -->
        <button type="button">检查账号是否已被注册</button><br>
        <!-- 普通的按钮第二种写法：注意一定要写type属性是button，否则是默认输入框-->
        <!-- <input type="button" value="检查账号是否已被注册"> -->
    </form>

    <!-- iframe标签用来在网站中嵌入一个其他的网页（有些网站做了现在无法将它们嵌入进来），有些网站就是使用这个嵌入的广告 -->
    <iframe src="https://www.bilibili.com/" width="100%" height="2469"></iframe>
</body>

</html>
~~~


### 浏览器渲染html，会先检查html是否合理，如果是不合理的文本就直接放入body标签元素中
~~~html
<html>
    <head></head>
    <body>
        <h1>浏览器渲染html</h1>
    </body>
</html>
~~~

# 单标签与双标签
- 单标签：meta、br、hr、img、input
- 双标签：除了上述单标签之外的常见标签基本都是双标签

# 块级元素和行内元素、行内块元素
- 块级元素：h1~h6、p、div、ul、ol、li、dd、dt、dl
- 行内元素：strong、b、em、i、del、s、ins、u、span、a、img、input、select、textarea、br

### 块级元素的特点
- 比较霸道，自己独占一行
- 高度，宽度、外边距以及内边距都可以控制
- 宽度默认是容器（父级宽度）的100%
- 是一个容器及盒子，里面可以放行内或者块级元素
- 注意：文字类的元素内不能使用块级元素，p 标签主要用于存放文字，因此 p 里面不能放块级元素，特别是不能放 div 。同理 h1~h6 等都是文字类块级标签，里面也不能放其他块级元素

### 行内元素的特点
- 相邻行内元素在一行上，一行可以显示多个
- 高、宽直接设置是无效的
- 默认宽度就是它本身内容的宽度
- 行内元素只能容纳文本或其他行内元素，不能容纳块级元素
- 注意：链接里面不能再放链接，特殊情况链接 a 里面可以放块级元素，但是给 a 转换一下块级模式最安全

### 行内块元素
在行内元素中有几个特殊的标签：img、input、td，它们同时具有块元素和行内元素的特点。有些资料称它们为行内块元素它们的特点如下
- 和相邻行内元素（行内块）在一行上，但是他们之间会有空白缝隙。一行可以显示多个（行内元素特点）
- 默认宽度就是它本身内容的宽度（行内元素特点）
- 高度，行高、外边距以及内边距都可以控制（块级元素特点）

### 块级元素和行内元素的转换
- 转换为块元素：display:block;
- 转换为行内元素：display:inline;
- 转换为行内块元素：display: inline-block;

### html特殊符号表示
- 一些不太常见的符合在html中无法正常表示，例如<和>需要使用特殊的表示方式，这种表示方式是HTML实体
- 常见的HTML实体符号如下（注意markdown文件的最终展示效果也是和html一样，请看原始的.md文件）
  - 空格的HTML实体表示是 &nbsp;
  - 小于号<的HTML实体表示是 &lt;
  - 大于号>的HTML实体表示是 &gt;
  - &的HTML实体表示是 &amp;
  - 人民币符号￥的实体表示 &yen;
  - 版权符号©的实体表示 &copy;
  - 双引号"的实体表示 &quot;
  - 反引号`的实体表示 &acute;