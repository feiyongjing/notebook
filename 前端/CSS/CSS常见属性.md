# 常见属性
~~~
href                设置a标签超链接地址
target              设置a标签超链接跳转方式或者form标签提交表单跳转方式
download            设置a标签超链接资源下载

src                 设置img标签图片地址
alt                 设置img标签图片无法显示提示文字

colspan             设置td标签单元格占据的宽度,即与左边的几个单元格合并，正常的表格的宽colspan是1
rowspan             设置td标签单元格占据的高度,即与下边的几个单元格合并，正常的表格的高rowspan是1

action              设置form标签提交表单的网址url
method              设置form标签提交表单请求方法

name                设置input标签、textarea多行文本输入框标签、select下拉框标签输入参数的名称
type                设置input标签输入的参数类型，有如下参数类型
                        text: 单行文本输入框
                        password: 密码输入框
                        email: 邮箱输入框，自动校验邮箱是否携带@合法
                        tel: 电话输入框，自动校验电话是否合法
                        color：颜色选择器
                        number: 数值输入框
                        datatime: 日期时间选择
                        datatime-local: 本地日期时间选择
                        week: 周选择
                        month：月选择
                        radio: 单选
                        checkbox: 多选
                        hidden：输入框隐藏
                        submit: input会变成提交按钮
                        reset: 重置按钮
                        file: 文件上传
placeholder         设置input标签输入框内的提示信息
value               设置input标签输入默认值，或提交到后台的值，如果是submit,reset的类型时，value就是按钮的名称
checked             设置input标签输入单选框或者多选框的选项被默认选中
required            设置input标签、textarea多行文本输入框标签、select下拉框标签输入参数必须填写，注意在单选框中表示必须选择一个，而多选框表示必须选择指定的项，所以有校验参数的选项都必须和required一起使用，否则可以提交空字符
disabled            设置input标签、textarea多行文本输入框标签、select下拉框标签禁用输入和修改
maxlength           设置input标签输入最大长度
pattern             设置input标签的正则表达式限制，但是前提是输入不为空，即一般和required一起使用

cols                设置textarea多行文本输入框标签文本框的宽是多少字符
rows                设置textarea多行文本输入框标签文本框的高是多少字符

font-size           设置字体大小，不同浏览器的默认字体大小不同，所以建议手动设置字体大小，例子 .test {font-size: 30px}
font-weight         设置字体粗细
font-style          设置字体样式是否是斜体
font-family         设置字体样式
color               设置字体颜色
font                设置字体的多个属性，即可以同时设置字体的多个属性
letter-spacing      设置字符间距
word-spacing        设置单词间距（只有英文文本生效）
text-decoration     设置文本上的线条（上划线、下划线、中划线、去除线条、线条样式和颜色）
text-indent         设置文本缩进
text-align          设置文本水平对齐方式
line-height         设置文本的行高，主要是用于设置多行文字的行间距，一般都是设置字体大小的1.5到2倍之间
vertical-align      设置文本的垂直对齐方式，但是注意是根据父元素的文本来进行的垂直定位，另外块级元素（行内块元素可以使用）无法使用vertical-align（如果需要设置垂直对齐方式请手动设置外边距实现，当然手动设置外边距可能会导致父元素位置移动，添加visibility属性可以解决）
text-shadow         设置文本的阴影：参考box-shadow使用，不过没有外延距离和内阴影
white-space         设置文本换行，nowrap表示不换行，pre-line表示按照标签宽度进行自动换行
text-overflow       设置文本溢出标签处理方式，clip表示直接截掉超出的文本，ellipsis表示超出的部分变为三个点省略，注意如果需要生效前提需要设置overflow属性和white-space属性为nowrap不换行

background-color    设置背景颜色，一般是设置文本或者div块的背景颜色

list-style-type     设置列表的列表符号
list-style-image    设置自定义的列表符号
list-style          设置列表的多个属性，即可以同时设置列表的多个属性

width               设置标签元素的内容宽度
height              设置标签元素的内容高度
min-width           设置标签元素跟随浏览器窗口大小变化的最小宽度
max-width           设置标签元素跟随浏览器窗口大小变化的最大宽度
min-height          设置标签元素跟随浏览器窗口大小变化的最小高度
max-height          设置标签元素跟随浏览器窗口大小变化的最大高度
margin              设置标签元素的外边距，另外还有例如 margin-left 属性表示左外边距设置，其他的依此类推。注意元素标签的宽高实际是内边距+边框+内容宽高，并不包含外边距，在一些情况下父元素会抢夺子元素的外边距设置
padding             设置标签元素的内边距，另外还有例如 padding-left 属性表示左内边距设置，其他的依此类推。
border-width        设置边框线条粗细，另外还有例如 border-left-width 属性表示左边框线条粗细设置，其他的依此类推。
border-color        设置边框线条颜色，另外还有例如 border-left-color 属性表示左边框颜色设置，其他的依此类推。
border-style        设置边框线条的类型，例如solid表示实线，dashed表示虚线，另外还有例如 border-left-style 属性表示左边框线条类型设置，其他的依此类推。
border              设置边框线条的多个属性，包含边框粗细、边框颜色、边框线条类型，另外还有例如 border-left 属性表示左边框设置，其他的依此类推。

table-layout        设置表格的列宽
border-spacing      设置单元格之间的间距，注意border-collapse: collapse设置单元格合并则设置的单元格间距无效
border-collapse     设置相邻的单元格边框是否合并
empty-cells         设置是否隐藏空内容的单元格
caption-side        设置表格标题的位置

background-color    设置背景的颜色，默认的背景颜色是transparent透明色
background-image    设置背景图片
background-repeat   设置背景图片的平铺方式，默认是即水平平铺也垂直平铺
background-position 设置背景图片的起始位置
background          设置背景的多个属性，包含上述的background-color、background-image、background-repeat、background-position
background-origin   设置背景图片平铺的起始点位置，border-box、padding-box、content-box分别代表边框左上角、内边距左上角、内容区左上角，默认是padding-box
background-clip     设置背景图片平铺后的裁切位置，border-box、padding-box、content-box分别代表边框位置、内边距位置、内容区裁切，默认是border-box
background-size     设置背景图片宽高占背景的宽高大小

display             设置标签元素显示方式：inline表示行内元素，block表示块元素，inline-block表示行内块元素，none表示隐藏该元素（元素不占位）
visibility          设置标签元素是否显示，默认是show显示，设置为hidden表示隐藏，但是实际还是会占据页面该元素的位置
hidden              设置标签元素隐藏

overflow            设置超出元素标签的内容部分如何处理，另外设置hidden值忽略超出的部分可以解决子元素的外边距设置导致当前元素发送的移动

float               设置元素浮动（设计浮动的初始目的是文字环绕图片块），元素浮动之后就会脱离文档流飘浮在其他元素上面（不属于父元素的内容，无法撑开父元素，在一些情况下会导致父元素塌陷，然后后面的元素或者是下一个兄弟元素位置不对），但是下方的文字会躲开覆盖挤压到其他地方，并且浮动元素的宽高默认是内容撑开的，浮动元素没有行内元素、行内块元素、块元素的特点
clear               设置元素消除它附件其他元素浮动的影响，注意当前元素只能是块元素并且不能设置浮动，否则会出现奇怪的样式页面

position            设置定位方式，relative、absolute、fixed、sticky分别代表相对定位、绝对定位、固定定位、粘性定位。多个定位（包含不同类型的定位）是后面的定位元素覆盖前面的定位元素
left                设置按照定位方式从左到右的定位距离，注意无法与right同时使用
right               设置按照定位方式从右到左的定位距离，注意无法与left同时使用
bottom              设置按照定位方式从下到上的定位距离，注意无法与top同时使用
top                 设置按照定位方式从上到下的定位距离，注意无法与bottom同时使用
z-index             设置定位块的覆盖优先级，兄弟元素之间该属性数值大的块会覆盖小的块

transform           设置2D和3D的位移（类似于定位）、缩放、旋转，该属性对行内元素不生效（但是对行内块生效）
transform-origin    设置2D和3D的缩放、旋转的原点

resize              设置标签宽高在标签右下角可以拖动伸缩，horizontal、vertical、both值分别代表可以在水平方向、垂直方向、水平和垂直方向调整宽高，注意必须与overflow属性一起使用（overflow的属性值可以随意设置，但是必须要有）
box-sizing          设置盒子模型方式，默认是content-box值表示width和height设置标签内容宽高，而border-box表示width和height设置标签总宽高（包含边框、内边距，如果设置了这些会默认挤压内容宽高变小，这种也叫怪异盒子模型）;
box-shadow          设置标签的阴影：默认是box-shadow: none没有阴影
opacity             设置标签的透明度，默认是1表示完全不透明，而0表示完全透明

transition-property           设置需要过渡的属性
transition-duration           设置过渡消耗时长
transition-timing-function    设置过渡时的变化方式，默认是 ease，即过渡的速度由慢到快再到慢
transition-delay              设置触发事件时过渡的延迟时间，即多少秒后执行过渡效果，默认是0s不延迟
transition                    设置过渡的多个属性

animation-name                设置自定义的动画名称
animation-duration            设置动画的周期时间
animation-delay               设置动画开始的延时时间
animation-timing-function     设置动画执行的速率
animation-iteration-count     设置动画的循环次数，默认是1次，infinite是无限次
animation-direction           设置动画播放的方向，默认是normal是向前播放，alternate表示在偶数次向前播放在奇数次反向播放
animation-play-state          设置动画是否播放，running代表运行动画效果，paused代表暂停动画效果
animation-fill-mode           设置动画停止时的帧，backwards、forwards分别表示第一帧和最后一帧
animation                     设置动画的多个属性

column-count        设置报纸排版段落列数
column-width        设置报纸排版每个段落的宽度，如果父元素宽度放不下就按照这个宽度放下最大的段落列数
column-gap          设置报纸排版段落之间纵向的宽度间距
column-rule-color   设置报纸排版段落的分割线颜色
column-rule-style   设置报纸排版段落的分割线线条条类型
column-rule-width   设置报纸排版段落的分割线的宽度
column-rule         设置报纸排版段落的分割线多个属性，包含分割线颜色、线条类型、宽度
~~~

## 颜色设置
- rgb和reba设置
  - rgb是指光学三元色红黄蓝，rgba的表示的是透明色
  - 使用0到255表示一种元色的深度，使用0到1直接的数表示透明色的透明度，当然元色和透明色也可以使用百分比表示
  - 例子 rgba(255, 0, 0, 0.5)
- HEX和HEXA设置
  - HEX和HEXA的表示实际和rgb和rgba的表示是一样的
  - HEX和HEXA的表示是 #???????? 
  - #后的12位表示的是红的元色，34两位表示的是绿的元色，56位表示的是蓝的元色，78位表示的是透明色，这些都是16进制数
  - 注意如果每种元色或者透明色表示的两位数是一样的可以简写为1为数，另外IE不支持HEXA
- HSL和HSLA设置
  - H代表色相角。即颜色环的角度
  - S代表饱和度。在这里，100% 是完全饱和，而 0% 是完全不饱和（灰色）
  - L代表亮度。在这里，100% 是白色，0% 是黑色，50% 是“正常”
  - A代表透明度，其中数字 1 代表 100%（完全不透明）
  - 例子 hsla(120deg, 75%, 25%, 1)

## 长度单位
- px像素：显示屏的分辨率的单位
- em：em是当前元素或者是祖先元素设置的font-size属性大小的倍数
- rem：rem是当前根元素设置的font-size属性大小的倍数
- 百分比：百分比的长度是相对与父元素的相同属性值的百分比（但是一些百分比并不是相对于父元素的属性）
- vw：vw表示长度的是窗口的宽度的百分之一，窗口宽度发送变化实际使用vw表示的长度也发生对应的变化
- vh：vh表示长度的是窗口的高度的百分之一，窗口长度发送变化实际使用vh表示的长度也发生对应的变化
- vmax：vmax表示长度的是窗口的宽度与窗口的高度更长一方的百分之一，窗口长度或宽度发送变化实际使用vmax表示的长度也发生对应的变化
- vmin：vmin表示长度的是窗口的宽度与窗口的高度更短一方的百分之一，窗口长度或宽度发送变化实际使用vmin表示的长度也发生对应的变化

## css属性继承
- css中和盒子相关的属性都无法继承，例如外边距、边框、内边距、背景、内容宽高等
- 盒子相关的属性之外的属性都是可以继承父元素或祖先元素的属性

## 行内元素或行内块元素在同一行之间出现空白的问题
- 写代码时的换行导致换行符被浏览器解析为空格的空白问题，解决方案是对父元素设置font-size字体大小为0px，然后当前元素手动设置字体大小
- 写行内块时出现上或者下面有空的间隙，原因是换内块默认是对齐的字体的基线导致的
  - 解决方案1：直接设置行内块元素的字体为0px(在行内块需要字体时不适用)
  - 解决方案2：不使用行内块元素，而是使用块元素（但是其他的元素会被挤到下一行）
  - 解决方案3：使用vertical-align设置文本的垂直对齐方式，top、middle、bottom、baseline分别表示向上对齐、居中对齐、向下对齐、默认字体基线对齐，只要不是默认的baseline基线对齐就行，注意这样会导致后面的文本垂直方向会发生变化

## 字体和字体图标
- 去 https://www.iconfont.cn/ 找字体和字体图标

# css字体属性设置
~~~html
<style>
    /* 设置字体大小，注意谷歌游览器最新只能设置12px */
    .fontSize {font-size: 30px;}

    /* 设置字体粗细：bold和bolder都是加粗（600以上都是这个效果），lighter设置细体(100~300都是这个效果)，正常字体是400~500*/
    .fontBold {font-weight: bold;}
    .fontBolder {font-weight: bolder;}
    .fontLighter {font-weight: lighter;}
    .fontWeightNum {font-weight: 1000;}

    /* 设置字体样式是否是斜体：normal是正常字体，italic是斜体*/
    .fontNormal {font-style: normal;}
    .fontItalic {font-style: italic;}

    /* 设置字体样式：可以设置多个字体，客户浏览器从前向后找字体，找到对应的字体就直接使用*/
    .fontFamilyYaheiOrSongti {font-family: "Microsoft Yahei", "宋体";}

    /* 设置颜色：直接使用单词颜色、使用16进制的颜色、rgb设置颜色、rgba设置颜色和透明度*/
    .fontColorRed {color: red;}
    .fontColorHex {color: #0000ff;}
    .fontColorRgb {color: rgb(0,255,0);}
    .fontColorRgba {color: rgba(0,255,0,0.5);}

    /* 字符间距设置，正数的px表示加大简介，负数的px表示减小简介 */
    .fontLetterSpacing{letter-spacing: 20px;}

    /* 单词的间距设置（只有英文文本生效），正数的px表示加大简介，负数的px表示减小简介 */
    .fontWordSpacing{word-spacing: 30px;}
</style>

<div class="fontSize">30px大小的文字</div>
<div class="fontBold">加粗到bold的文字</div>
<div class="fontBolder">加粗到bulder的文字</div>
<div class="fontLighter">细体Lighter的文字</div>
<div class="fontWeightNum">字体1000粗细的文字</div>
<div class="fontNormal">正常的字体<span class="fontItalic"> 斜体</span></div>
<div class="fontFamilyYaheiOrSongti">字体样式是微软雅黑或宋体</div>
<div class="fontColorRed">文字颜色使用可直接描述的颜色设置</div>
<div class="fontColorHex">文字颜色使用16进制设置</div>
<div class="fontColorRgb">文字颜色使用rgb设置</div>
<div class="fontColorRgba">文字颜色使用rgba设置</div>
<div class="fontLetterSpacing">字符简介为20px</div>
<div class="fontWordSpacing">I live you</div>
~~~

# css文本属性设置
~~~html
<style>
    /* text-decoration的属性值overline表示上划线、dotted表示虚线、darkgreen表示颜色 */
    .test1{
        text-decoration: overline dotted darkgreen;
    }

    /* text-decoration的属性值line-through表示中划线 */
    .test2{
        text-decoration: line-through;
    }

    /* text-decoration的属性值underline表示下划线、wavy表示波浪线、hotpink表示颜色 */
    .test3{
        text-decoration: underline wavy hotpink;
    }

    /* text-decoration的属性值none值表示去除文本标签上的线条 */
    .test4{
        text-decoration: none;
    }

    /* text-indent设置文本缩进，1em就是当前元素或者是祖先元素设置的font-size属性大小 */
    .test5{
        text-indent: 2em;
    }

    /* text-align设置文本水平对齐方式，left、center、right */
    .test6{
        background-color: aquamarine;
        text-align: center;
    }

    /* line-height设置文本的行高，有4种写法
       1、直接写像素高度，例如60px
       2、写normal，这是默认浏览器自动根据字体调整行高确保上下行的文本不会重叠
       3、写数字，表示的是字体大小的倍数，例如1.8，推荐使用这个，一般都是1.5到2倍之间确保阅读轻松
       4、写百分比，表示的也是字体大小的倍数，例如 180%
     */
    .test7{
        line-height: 1.8;
    }

    /* vertical-align设置文本的垂直对齐方式，但是注意是根据父元素的文本来进行的垂直定位，另外块级元素无法使用vertical-align
       top、middle、bottom、baseline分别表示向上对齐、居中对齐、向下对齐、默认字体基线对齐
    */
    .test8{
        font-size: 30px;
        vertical-align: top;
    }
</style>

<div class="test1">文本上划线</div>
<div class="test2">文本中划线</div>
<div class="test3">文本下划线</div>
<a class="test4" href="https://www.baidu.com">去除a标签的下滑线</a>
<div class="test5">文本缩进</div>
<div class="test6">文本水平方向居中对齐</div>
<div class="test7">
    滕王高阁临江渚，佩玉鸣鸾罢歌舞。<br>
    画栋朝飞南浦云，珠帘暮卷西山雨。<br>
    闲云潭影日悠悠，物换星移几度秋。<br>
    阁中帝子今何在？槛外长江空自流。<br>
</div>
<div style="height: 200px;background-color: khaki; font-size: 100px;">
    父元素文本x<span class="test8">x子元素文本根据父元素文本垂直向上对齐</span>
</div>
~~~

# css列表属性设置(有序列表和无序列表都可以这样设置)
~~~html
<style>
    .test{
        /*  list-style-type设置列表的列表符号
            disc、circle、square、decimal、none分别代表黑色实心圆、空心圆、实心矩形、有序数字符号、不显示列表符号
         */
        /* list-style-type: decimal; */

        /* list-style-image设置自定义的列表符号 */
        /* list-style-image: url("../xxx.png"); */

        /* list-style可以设置列表的多个属性包含上述的list-style-type和list-style-image属性设置 */
        list-style: decimal url("../xxx.png");
    }
</style>

<h3>列表</h3>
<ul class="test">
    <li>无序列表ul+li</li>
    <li>disc、circle、square分别代表黑色实心圆、空心圆、实心矩形</li>
    <li>xxxx</li>
</ul>
~~~

# css边框设置
~~~html
<style>
    .test1{
        /* border-width设置边框线条粗细 */
        border-width: 5px;

        /* border-color设置边框线条颜色 */
        border-color: green;

        /* border-style设置边框线条的类型，例如solid表示实线，dashed表示虚线 */
        border-style: solid;
    }

    .test2{
        /* border设置边框线条的多个属性，包含边框粗细、边框颜色、边框线条类型 */
        border: 3px greenyellow dashed;
    }
</style>

<p class="test1">边框设置为5px绿色实线</p>
<div class="test2">边框设置为3px黄绿色虚线</div>
~~~

# css表格属性设置
~~~html
<style>
    .test{
        /* 设置表格边框 */
        border: 5px green solid;

        /* table-layout设置表格的列宽
           默认是auto自动根据单元格内容调整宽度 
           fixed强制使所有单元格的宽度保持一致
        */
        table-layout: fixed;

        /* border-spacing 设置单元格之间的间距
           注意border-collapse: collapse设置单元格合并则设置的单元格间距无效
         */
        border-spacing: 2px;

        /* border-collapse 设置相邻的单元格边框是否合并
           默认separate是不合并单元格边框
           collapse表示合并单元格边框
         */
        border-collapse: separate;

        /* empty-cells设置是否隐藏空内容的单元格
           默认show显示，hide表示隐藏
         */
        empty-cells: hide;

        /* caption-side设置表格标题的位置，默认是top在表格上方显示，bottom表示在表格下方显示表格标题 */
        caption-side: bottom;
    }

    .test th,td{
        /* 设置表格单元格边框*/
        border: 5px orange solid;
    }
</style>

<table class="test">
    <caption>表格标题test</caption>
    <thead> <!-- 表格头 -->
        <tr>
            <th></th>
            <th>中文</th>
            <th>英语</th>
        </tr>
    </thead>
    <tbody> <!-- 表格主体 -->
        <tr>
            <th></th> 
            <td>红色</td>
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
            <th>词汇数</th>
            <td>2</td>
            <td>2</td>
        </tr>
    </tfoot>
</table>
~~~

# css背景属性设置
~~~html
<style>
    /* background-color设置背景的颜色，注意默认的背景颜色是transparent透明色*/
    .test1 {
        width: 200px;
        height: 200px;
        background-color: green;
    }

    /* background-image属性设置背景图片
       background-repeat属性设置背景图片的平铺方式，默认是即水平平铺也垂直平铺
         background-repeat属性默认是repeat即水平平铺也垂直平铺，repeat-x、repeat-y、no-repeat分别表示水平平铺、垂直平铺、不平铺
     */
    .test2 {
        width: 500px;
        height: 500px;
        background-color: sandybrown;
        background-image: url(../img/KFC.jfif);
        background-repeat: no-repeat;
    }

    /* background-position设置背景图片的起始位置
       第一个值设置左中右（left、center、right、百分比、具体的像素）
       第二个值设置上中下（top、center、bottom、百分比、具体的像素） 
       如果只写一个值则默认先识别是左右还是上下设置（当然百分比、具体的像素只写一个值默认是水平方向设置），至于另外一个值默认是center居中
    */
    .test3 {
        width: 500px;
        height: 500px;
        background-color: hotpink;
        background-image: url(../img/KFC.jfif);
        background-repeat: no-repeat;
        background-position: left bottom;
    }

    /* background设置背景的多个属性
       模板是 background: 背景颜色 图片地址 平铺方式 图片起始位置 / 图片大小伸缩 图片原点 裁切方式
       并且如果需要设置多张图片可以继续在后面添加逗号分割
    */
    .test4 {
        width: 500px;
        height: 500px;
        background: mediumaquamarine url(../img/KFC.jfif) right center no-repeat;
    }

    /* background-size设置图片宽高占背景的大小 
       可以直接写px像素大小，auto表示根据另外的宽或者高自适应
       写百分比表示占据背景宽或者高的百分比
       写cover表示背景图片在不变型情况下的充满整个背景，可能图片会有裁剪
       写contain表示背景图片在不变型情况下尽可能的充满整个背景，图片可能不会占用整个背景，会留下一些空白
    */
    .test5 {
        width: 400px; 
        height: 300px; 
        background-image: url(../img/KFC.jfif); 
        background-size: 300px auto; 
    }
</style>

<div class="test1">宽200px高200px的背景设置为绿色</div>
<div class="test2">背景宽500px高500px设置图片不平铺</div>
<div class="test3">背景宽500px高500px设置图片不平铺且在左下</div>
<div class="test4">背景宽500px高500px设置图片不平铺且在右中</div>
<div class="test5">设置图片宽高占背景的大小</div>
~~~

# css背景图片颜色线性渐变设置
~~~html
<style>
    * {
        font-size: 0px;
    }

    .test {
        width: 300px;
        height: 200px;
        display: inline-block;
        margin: 0px 10px;
    }

    .test1 {
        /* linear-gradient设置背景图片为线性渐变颜色图片，可以传入多个颜色参数进行渐变 */
        background-image: linear-gradient(red, yellow, green);
    }

    .test2 {
        /* 第一个参数可以设置颜色渐变线的方向（默认是向下渐变），例如 to right top 表示向右上角开始渐变 */
        background-image: linear-gradient(to right top, red, yellow, green);
    }

    .test3 {
        /* 第一个参数也可以设置颜色渐变线方向角度 */
        background-image: linear-gradient(20deg, red, yellow, green);
    }

    .test4 {
        /* 可以在颜色的后面添加长度，表示在颜色渐变的方向线的该长度必定是指定的颜色（例如纯红色）
           注意颜色渐变方向线开始到第一个指定刻度颜色这段和最后一个指定刻度颜色到颜色渐变方向线结束位置段都是纯的颜色（第一或者最后刻度的纯色）
         */
        background-image: linear-gradient(red 50px, yellow 100px, green 150px);
    }

    .test5 {
        font-size: 40px;
        font-weight: bolder;
        text-align: center;
        vertical-align: top;
        line-height: 200px;
        color: transparent;
        background-image: linear-gradient(red, yellow, green);
        -webkit-background-clip: text;
    }
</style>

<div>
    <div class="test test1 "></div>
    <div class="test test2"></div>
    <div class="test test3 "></div>
    <div class="test test4"></div>
    <div class="test test5 ">三彩渐变的文字</div>
</div>
~~~

# css背景图片颜色径向渐变设置
~~~html
<style>
    .test {
        width: 300px;
        height: 200px;
        display: inline-block;
        margin: 0px 10px;
    }

    .test1 {
        /* linear-gradient设置背景图片为径向渐变颜色图片，可以传入多个颜色参数进行渐变 */
        background-image: radial-gradient(red, yellow, green);
    }

    .test2 {
        /* at参数可以设置径向渐变的中心点位，例如 at right top 表示向右上角是中心点位 */
        background-image: radial-gradient(at right top, red, yellow, green);
    }

    .test3 {
        /* at参数也可以设置px像素来改变径向渐变的中心点位 */
        background-image: radial-gradient(at 100px 100px, red, yellow, green);
    }

    .test4 {
        /* circle参数可以设置径向渐变是一个正圆
           可以在颜色的后面添加长度，表示在颜色渐变的半径线的该长度必定是指定的颜色（例如纯红色）
           注意颜色渐变半径线开始到第一个指定刻度颜色这段和最后一个指定刻度颜色到颜色渐变半径线结束位置段都是纯的颜色（第一或者最后刻度的纯色）
         */
        background-image: radial-gradient(circle,red 50px, yellow 100px, green 150px);
    }

    .test5 {
        /* 也可以直接设置径向渐变水平半径和垂直半径
         */
        background-image: radial-gradient(100px 50px ,red, yellow, green);
    }

    .test6 {
        /* 设置径向渐变水平半径和垂直半径和径向渐变的中心点位的综合写法 */
        background-image: radial-gradient(100px 50px at 100px 100px, red, yellow, green);
    }
</style>

<div>
    <div class="test test1"></div>
    <div class="test test2"></div>
    <div class="test test3 "></div>
    <div class="test test4"></div>
    <div class="test test5"></div>
    <div class="test test6"></div>
</div>
~~~

# css背景图片纯色区域重复渐变设置
~~~html
<style>
    * {
        font-size: 0px;
    }

    .test {
        width: 300px;
        height: 200px;
        display: inline-block;
        margin: 0px 10px;
    }

    .test1 {
        /* 线性渐变的纯色区域重复渐变 */
        background-image: repeating-radial-gradient(100px 50px at 100px 100px, red 50px, yellow 100px, green 150px);
    }

    .test2 {
        /* 径向渐变的纯色区域重复渐变 */
        background-image: repeating-linear-gradient(red 50px, yellow 100px, green 150px);
    }

    .test3 {
        font-size: 40px;
        font-weight: bolder;
        text-align: center;
        vertical-align: top;
        line-height: 200px;
        color: transparent;
        background-image: repeating-linear-gradient(30deg, red 100px, yellow 150px, green 200px);
        -webkit-background-clip: text;
    }
</style>

<div>
    <div class="test test1"></div>
    <div class="test test2"></div>
    <div class="test test3">三彩渐变的文字</div>
</div>
~~~

# css块级元素、行内元素、行内块元素转换
~~~html
<style>
    /* display设置标签元素显示方式：inline表示行内元素，block表示块元素，inline-block表示行内块元素*/
    div {
        width: 200px;
        height: 200px;
        display: inline-block;
    }

    .test1 {
        background-color: green;
    }

    .test2 {
        background-color: yellowgreen;
    }

    .test3 {
        background-color: skyblue;
}
</style>

<div class="test1"></div>
<div class="test2"></div>
<div class="test3"></div>
~~~

# css设置标签元素外边距、内边距、边框、内容宽高
~~~html
<style>
    /* width设置标签元素的内容宽度
       height设置标签元素的内容高度
       min-width设置标签元素跟随浏览器窗口大小变化的最小宽度
       max-width设置标签元素跟随浏览器窗口大小变化的最大宽度
       min-height设置标签元素跟随浏览器窗口大小变化的最小高度
       max-height设置标签元素跟随浏览器窗口大小变化的最大高度
       margin设置标签元素的外边距
       padding设置标签元素的内边距
       注意margin、padding、border都是设置的四个方向的边距或边框，在写1~4个值的长度会有不同的效果
       写1个表示的是4个方向长度一致
       写2个按照先后顺序表示的是上下、左右
       写3个按照先后顺序表示的是上、左右、下
       写4个按照先后顺序表示的是上、右、下、左
       另外margin、padding、border它们还有专门对应方向的属性，例如 border-left 属性表示左边框设置，其他的依此类推
    */
    .test {
        width: 200px;
        height: 200px;
        /* min-width: 600px; */
        /* max-width: 1000px; */
        /* min-height: 600px; */
        /* max-height: 1000px; */
        background-color: yellowgreen;
        margin: 50px;
        border: 10px rebeccapurple solid;
        padding: 100px;
    }
</style>

<div class="test"></div>
~~~

# css设置超出元素部分的内容如何处理
~~~html
<style>
    /* overflow 设置超出元素标签的内容部分如何处理，
       hidden忽略超出的部分，另外忽略超出的部分可以解决子元素的外边距设置导致当前元素发送的移动
       scroll使用进度条拉取展示忽略的部分
       auto表示如果有超出的部分就显示出进度条展示忽略的部分，没有就不显示进度条
    */
    .test1 {
        width: 400px;
        height: 400px;
        background-color: yellowgreen;
        overflow: scroll;
    }

    .test2 {
        width: 1000px;
        background-color: burlywood;
    }
</style>

<div class="test1">
    豫章故郡，洪都新府。星分翼轸(zhěn)，地接衡庐。襟三江而带五湖，控蛮荆而引瓯（ōu）越。物华天宝，龙光射牛斗之墟；
    <div class="test2">
        人杰地灵，徐孺下陈蕃(fān)之榻。雄州雾列，俊采星驰，台隍(huáng)枕夷夏之交，宾主尽东南之美。都督阎公之雅望，棨(qǐ)戟遥临；宇文新州之懿(yì)范，襜(chān )帷(wéi)暂驻。十旬休假，胜友如云；千里逢迎，高朋满座。腾蛟起凤，孟学士之词宗；紫电清霜，王将军之武库。家君作宰，路出名区；童子何知，躬逢胜饯。
    　　时维九月，序属三秋。潦（lǎo）水尽而寒潭清，烟光凝而暮山紫。俨(yǎn)骖騑(cān fēi)于上路，访风景于崇阿(ē)。临帝子之长洲，得天人之旧馆。层峦耸翠，上出重霄；飞阁流（一作翔）丹，下临无地。鹤汀（tīng）凫(fú )渚（zhǔ），穷岛屿之萦(yíng)回；桂殿兰宫，即（一作 列）冈峦之体势。
        披绣闼（tà），俯雕甍(méng )。山原旷其盈视，川泽纡(yū)其骇瞩。闾(lǘ)阎(yán) 扑地，钟鸣鼎食之家；舸（gě)舰弥津，青雀黄龙之舳（zhú）。云销雨霁(jì)，彩彻区明（或作 虹销雨霁，彩彻云衢qú）。落霞与孤鹜(wù)齐飞，秋水共长天一色。渔舟唱晚，响穷彭蠡（l ǐ）之滨；雁阵惊寒，声断衡阳之浦。
    </div>
    　　遥襟甫畅，逸兴遄(chuán)飞。爽籁发而清风生，纤歌凝而白云遏(è)。睢(suī)园绿竹，气凌彭泽之樽；邺(yè)水朱华，光照临川之笔。四美具，二难并。穷睇眄(dì
    miǎn)于中天，极娱游于暇日。天高地迥(jiǒng)，觉宇宙之无穷；兴尽悲来，识盈虚之有数。望长安于日下，目吴会（kuài）于云间。地势极而南溟(míng)深，天柱高而北辰远。关山难越，谁悲失路之人；萍水相逢，尽是他乡之客。怀帝阍(hūn)而不见，奉宣室以何年。
</div>
~~~

# css设置浮动和清除浮动影响
~~~html
<style>
    .test {
        background-color: mediumpurple;
        border: 2px red solid;
        /* 消除浮动的第一种方式：设置父元素高度消除父元素高度崩塌，但是浮动元素相邻的非浮动兄弟元素会被覆盖，然后该元素文本被挤到其他地方 */
        /* height: 200px; */
        /* 消除浮动的第二种方式：设置父元素也浮动消除父元素高度崩塌，会有第一种方式的问题并且浮动会导致父元素的内容被子元素撑开 */
        /* float: left; */
        /* 消除浮动的第三种方式：设置父元素清除浮动消除父元素之外的样式，但是浮动元素相邻的非浮动兄弟元素会被清除  */
        /* overflow: hidden; */
    }

    .test1 {
        font-size: 20px;
        width: 200px;
        height: 200px;
        background-color: yellowgreen;
        display: inline-block;
        vertical-align: top;
        float: left;
    }

    .test2 {
        font-size: 20px;
        width: 200px;
        height: 200px;
        background-color: red;
        display: inline-block;
        vertical-align: top;
        float: left;
    }

    .test3 {
        font-size: 20px;
        width: 200px;
        height: 200px;
        background-color: violet;
        display: inline-block;
        vertical-align: top;
        float: left;
    }

    .test4 {
        font-size: 20px;
        width: 200px;
        height: 200px;
        background-color: green;
        /* 消除浮动的第四种方式：设置当前元素消除它附件其他元素浮动的影响，注意当前元素只能是块元素并且不能设置浮动
           left、right、both分别代表清除左边、右边、左右的元素浮动影响
           一般是在最后一个元素添加空的div清除浮动影响
        */
        clear: both;
    }

    /* 消除浮动的第五种方式：使用伪元素选择器，在父元素包含的子元素列表的最后中添加一个空的子元素，在这个子元素中设置清除浮动影响，
       注意这样写代码一定要确保子元素列表原来的最后一个元素必须设置了浮动，如果没有设置浮动会出现奇怪的样式
    */
    /* .test::after {
        content: '';
        display: block;
        clear: both;
    } */

    .test5 {
        font-size: 20px;
        width: 200px;
        height: 200px;
        background-color: palegreen;
    }
</style>

<div class="test">
    <div class="test1">xxxxxxxxrel属性设置链接方式与html文档之间的关系，stylesheet表示是样式表</div>
    <div class="test2">xxx属性设置链接方式与html文档之间的关系，stylesheet表示是样式表</div>
    <div class="test3">eqwd</div>
    <div class="test4">vsdc</div>
</div>
<div class="test5">mukjy</div>
~~~

# css设置元素相对定位、绝对定位、固定定位
~~~html
<style>
    .test {
        width: 400px;
        height: 400px;
        background-color:#f66daa;  
        /* 后代元素如果需要设置绝对定位，则祖辈元素必须设置position: relative;声明，
        否则依次向上级元素查找，如果所有的上级元素都找不到就按照顶层文档作为绝对定位的相对位置 */
        position: relative;
        margin-left: 300px;
        left: 300px;
    }

    .test1 {
        width: 100px;
        height: 250px;
        background-color:rgb(75, 245, 129);
        /* 设置相对定位，会覆盖元素，并且多个定位也是后面的元素覆盖前面的元素，子元素的左上角针对父元素的相对位置，
        left设置从左到右的位置，right设置从右到左的位置,bottom设置从下到上的位置，top设置从上到下的位置 */
        position: relative;
        left: 100px;
        top: 50px;
    }

    .test2 {
        width: 100px;
        height: 100px;
        background-color:rgb(184, 55, 249);
        /* 设置绝对定位，会脱离文档流覆盖下层元素（无法与浮动一起使用），并且多个定位也是后面的元素覆盖前面的元素，子元素的左上角针对上个设置了position: relative定位祖辈元素（如果没有就是html标签元素）的相对位置，
        也就是说父辈元素的位置变化对设置绝对定位的子元素也会跟着一起移动
        left设置从左到右的位置，right设置从右到左的位置,bottom设置从下到上的位置，top设置从上到下的位置*/
        position: absolute;
        left: 150px;
        top: 150px;
    }

    .test3 {
        width: 100px;
        height: 100px;
        background-color:rgb(75, 240, 238);
        /* 设置固定定位，固定在浏览器页面
        left设置从左到右的位置，right设置从右到左的位置,bottom设置从下到上的位置，top设置从上到下的位置*/
        position: fixed;
        right: 100px;
        top: 50px;
    }
</style>

<div class="test">
    <div class="test1">相对定位，左100px高100px</div>
    <div class="test2">绝对定位，左150px高150px</div>
    <div class="test3">固定定位，右100px高50px</div>
</div>
~~~

# css设置元素粘性定位
~~~html
<style>
    * {
        margin: 0px;
        padding: 0px;
        height: 2500px;
    }

    .test {
        height: 100px;
        background-color:#f66daa;  
        text-align: center;
        line-height: 100px;
        font-size: 60px;
    }

    .test1 {
        height: 100px;
        background-color:cadetblue;
        line-height: 100px;
        font-size: 40px;
        /* 粘性定位，当元素通过滚动条移动到距离拥有滚动条属性的元素一定距离后会之间定位在该位置，直到其他元素也通过粘性定位覆盖 */
        position: sticky;
        /* 粘性定位距离拥有滚动条属性元素的先边缘一定距离后会之间定位在该位置，同理 left、right、bottom也是一样 */
        top: 0px;
    }

    span {
        font-size: 25px;
        background-color:palegoldenrod; 
    }
</style>

<div class="test">向下滑动滚动条</div>
<div>
    <div class="test1">第一段</div>
    <span>二愣子睁大着双眼，直直望着茅草和烂泥糊成的黑屋顶，身上盖着的旧棉被，已呈深黄色，看不出原来的本来面目，还若有若无的散发着淡淡的霉味。
        在他身边紧挨着的另一人，是二哥韩铸，酣睡的十分香甜，从他身上不时传来轻重不一的阵阵打呼声。
        离床大约半丈远的地方，是一堵黄泥糊成的土墙，因为时间过久，墙壁上裂开了几丝不起眼的细长口子，
        从这些裂纹中，隐隐约约的传来韩母唠唠叨叨的埋怨声，偶还掺杂着韩父，抽旱烟杆的“啪嗒”“啪嗒”吸允声。
        二愣子缓缓的闭上已有些发涩的双目，迫使自己尽早进入深深的睡梦中。
        他心里非常清楚，再不老实入睡的话，明天就无法早起些了，也就无法和其他约好的同伴一进山拣干柴。
        二愣子姓韩名立，这么像模像样的名字,他父母可起不出来，这是他父亲用两个粗粮制成的窝头，求村里老张叔给起的名字。
        老张叔年轻时，曾经跟城里的有钱人当过几年的伴读书童，是村里唯一认识几个字的读书人，村里小孩子的名字，倒有一多半是他给起的。
        韩立被村里人叫作“二愣子”，可人并不是真愣真傻，反而是村中首屈一指的聪明孩子，但就像其他村中的孩子一样，
        除了家里人外，他就很少听到有人正式叫他名“韩立”，倒是“二愣子”“二愣子”的称呼一直伴随至今。
        而之所以被人起了个“二愣子”的绰号，也只不过是因为村里已有一个叫“愣子”的孩子了。
        这也没啥，村里的其他孩子也是“狗娃”“二蛋”之类的被人一直称呼着，这些名字也不见得比“二愣子”好听了哪里去。
        因此，韩立虽然并不喜欢这个称呼，但也只能这样一直的自我安慰着。
    </span>
    <div class="test1">第二段</div>
    <span>韩立外表长得很不起眼，皮肤黑黑的，就是一个普通的农家小孩模样。但他的内心深处，却比同龄人早熟了许多，
        他从小就向往外面世界的富饶繁华，梦想有一天他能走出这个巴掌大的村子，去看看老张叔经常所说的外面世界。
        当韩立的这个想法，一直没敢和其他人说起过。否则，一定会使村里人感到愕然，一个乳臭未干的小屁孩，
        竟然会有这么一个大人也不敢轻易想的念头。要知道，其同韩立差不多大的小孩，都还只会满村的追鸡摸狗，更别说会有离开故土，这么一个古怪的念头。
        韩立一家七口人，有两个兄长，一个姐姐，还有一个小妹，他在家里排行老四，今年刚十岁，
        家里的生活很清苦，一年也吃不上几顿带荤腥的饭菜，全家人一直在温线上徘徊着。
        此时的韩立，正处于迷迷糊糊，似睡未睡之间，恼中还一直残留着这样的念头：上山时，一定要帮他最疼爱的妹妹，多拣些她最喜欢吃的红浆果。
        第二天中午时分，当韩立顶着火辣辣的太阳，背着半人高的木柴堆，怀里还揣着满满一布袋浆果，
        从山里往家里赶的时侯，并不知道家中已来了一位，会改变他一生运的客人。
        这位贵客，是跟他血缘很近的一位至亲，他的亲三叔。
        听说，在附近一个小城的酒楼，给人当大掌柜，是他父母口中的大能人。韩家近百年来，可能就出了三叔这么一位有点身份的亲戚。
        韩立只在很小的时侯，见过这位三叔几次。他大哥在城里给一位老铁匠当学徒的工作，就是这位三叔给介绍的，
        这位三叔还经常托人给他父母捎带一些吃的用的东西很是照顾他们一家，因此韩立对这位三叔的印像也很好，知道父母虽然嘴里不说，心里也是很感激的。
    </span>
    <div class="test1">第三段</div>
    <span>大哥可是一家人的骄傲，听说当铁匠的学徒，不但管吃管住，一个月还有三十个铜板拿，等到正式出师被人雇用时，挣的钱可就更多了。
        每当父母一提起大哥，就神采飞扬，像换了一个人一样。
        韩立年龄虽小，也羡慕不已，心目最好的工作也早早就有了，就是给小城里的哪位手艺师傅看上，收做学徒从此变成靠手艺吃饭的体面人。
        所以当韩立见到穿着一身崭新的缎子衣服，胖胖的圆脸，留着一撮小胡子的三叔时，心里兴奋极了。
        把木柴在屋后放好后，便到前屋腼腆的给三叔见了个礼，乖乖的叫了声：“三叔好”，就老老实实的站在一边，听父母同三叔聊天。
        三叔笑眯眯的望着韩立，打量着他一番，嘴里夸了他几句“听话”“懂事”之类的话，然后就转过头，和他父母说起这次的来意。
        韩立虽然年龄尚小，不能完全听懂三叔的话，但也听明白了大概的意思。
        原来三叔工作的酒楼，属于一个叫“七玄门”的江湖门派所有，这个门派有外门和内门之分，
        而前不久，三叔才正式成为了这个门派的外门弟子，能够推举7岁到12岁孩童去参加七玄门招收内门弟子的考验。
        五年一次的“七玄门”招收内门弟子测试，下个月就要开始了。这位有着几分精明劲自己尚无子女的三叔，自然想到了适龄的韩立。
    </span>
    <div class="test1">第四段</div>
    <span>一向老实巴交的韩父，听到“江湖”“门派”之类的从未听闻过的话，心里有些犹豫不决拿不定主意。
        便一把拿起旱烟杆，“吧嗒”“吧嗒”的狠狠抽了几口，就坐在里，一声不吭。
        在三叔嘴里，“七玄门”自然是这方圆数百里内，了不起的、数一数二的大门派。
        只要成为内门弟子，不但以后可以免费习武吃喝不愁，每月还能有一两多的散银子零花。
        而且参加考验的人，即使未能入选也有机会成为像三叔一样的外门人员，专替“七玄门”打理门外的生意。
        当听到有可能每月有一两银子可拿，还有机会成为和三叔一样的体面人，韩父终于拿定了主意，答应了下来。
        三叔见到韩父应承了下来，心里很是高兴。又留下几两银子，说一个月后就来带韩立走，在这期间给韩立多做点好吃的，给他补补身子，好应付考验。
        随后三叔和韩打声招呼，摸了摸韩立的头，出门回城了。
        韩立虽然不全明白三叔所说的话，但可以进城能挣大钱还是明白的。
        一直以来的愿望，眼看就有可能实现，他一连好几个晚上兴奋的睡不着觉。
        三叔在一个多月后，准时的来到村中，要带韩立走了，临走前韩父反复嘱咐韩立，做人要老实，遇事要忍让，别和其他人起争执，而韩母则要他多注意身体，要吃好好。
    </span>
    <div class="test1">第五段</div>
    <span>在马车上，看着父母渐渐远去的身影，韩立咬紧了嘴唇，强忍着不让自己眼框中的泪珠流出来。
        他虽然从小就比其他孩子成熟的多，但毕竟还是个十岁的小孩，第一次出远门让他的心里有点伤感和彷徨。
        他年幼的心里暗暗下定了决心，等挣到了大钱就马上赶来，和父母再也不分开。
        韩立从未想到，此次出去后钱财的多少对他已失去了意义，他竟然走上了一条与凡人不同的仙业大道，走出了自己的修仙之路。（未完待续）
    </span>
</div>
~~~

# css设置2D位移
~~~html
<style>
    .test {
        width: 300px;
        height: 300px;
        border: 3px red solid;
        background-color: aquamarine;
        position: relative;
        display: inline-block;
    }

    .test1 {
        width: 100px;
        height: 100px;
        position: absolute;
        top: 50%;
        left: 50%;
        /* 设置2D水平位移，正数像素表示向右移动，负数像素表示向左移动  */
        /* transform: translateX(50px); */
        /* 设置2D垂直位移，正数像素表示向下移动，负数像素表示向上移动 */
        /* transform: translateY(100px); */
        /* 同时设置2D水平位移和2D垂直位移可以像上述那样，但是注意要同时设置translateX和translateY并且使用空格分隔
           或者直接使用translate设置
           注意如果translate只写一个值表示的是水平位移，垂直方向不会动
           百分比表示的长度是当前元素宽高的百分比
         */
        transform: translate(-50%, -50%);
        background-color: pink;
    }

    .test2 {
        width: 100px;
        height: 100px;
        position: absolute;
        top: 50%;
        left: 50%;
        /* 设置2D左右缩放，正常默认是1，0表示看不见，0~1表示缩小，大于1表示扩大，如果是负数就会左右颠倒，当然如果大于-1就会左右颠倒并且扩大  */
        /* transform: scaleX(1.5); */
        /* 设置2D上下缩放，正常默认是1，0表示看不见，0~1表示缩小，大于1表示扩大，如果是负数就会上下颠倒，当然如果大于-1就会上下颠倒并且扩大  */
        /* transform: scaleY(1.5); */
        /* 同时设置2D左右缩放和2D上下缩放可以像上述那样，但是注意要同时设置scaleX和scaleY并且使用空格分隔
           或者直接使用scale设置
           注意如果scale只写一个值表示的是同时左右缩放和上下缩放，并且缩放可以实现缩小文字到比浏览器的最小文字还要小
         */
        transform: translate(-50%, -50%) scale(-2);
        background-color: pink;
    }

    .test3 {
        width: 100px;
        height: 100px;
        position: absolute;
        top: 50%;
        left: 50%;
        /* 设置2D旋转，默认是按照元素的正中心原点进行的旋转，正数角度表示顺时针旋转，如果是负数角度表示逆时针旋转  */
        /* transform: rotateZ(30deg); */
        /* 当然写rotate(30deg) 传递一个值默认也是表示的2D旋转 */
        /* 注意当位移与旋转同时编写时请将位移写在旋转前面，否则旋转写在前面，先发生旋转导致原点变化，
           同时水平方向和垂直方向角度发生变化，后面的位移位置也会发生变化 */
        transform: translate(-50%, -50%) rotateZ(30deg);
        background-color: pink;
    }

    /* 原点默认是当前元素的左上角，位移对行内元素不生效 */
    .test4 {
        width: 100px;
        height: 100px;
        position: absolute;
        top: 50%;
        left: 50%;
        /* transform-origin设置原点 */
        /* transform-origin: right bottom; */
        /* 可以写具体的像素值和百分比，只写一个像素值或者百分比同时表示的水平和垂直方向，而写一个方向值就表示另外一个方向是中心值 */
        /* 注意修改原点会影响缩放和旋转，会根据原点进行缩放和旋转 */
        transform-origin: left top;
        transform: translate(-50%, -50%) scale(1.5) rotateZ(-30deg);
        background-color: pink;
    }
</style>

<div class="test">
    <div class="test1">靠定位和位移实现垂直水平居中</div>
</div>
<div class="test">
    <div class="test2">水平缩放和垂直缩放都是负数就颠倒两次</div>
</div>
<div class="test">
    <div class="test3">顺时针旋转30度</div>
</div>
<div class="test">
    <div class="test4">顺时针旋转30度</div>
</div>
~~~

# css设置元素阴影、透明、圆角
~~~html
<style>
    .test1 {
        margin: 50px;
        width: 100px;
        height: 100px;
        background-color: mediumspringgreen;
        /* box-shadow设置标签的阴影：默认是box-shadow: none没有阴影
           第一个值设置水平方向阴影，正数向右移动，负数向左移动
           第二个值设置垂直方向阴影，正数向下移动，负数向上移动
           第三个值是可选项，设置阴影模糊距离，距离越大阴影越大也越模糊
           第四个值是可选项，设置阴影的外延距离，即阴影的四周比标签的四周大多少
           第五个值是可选项，设置阴影颜色，默认是黑色
           第六个值是可选项，设置内外阴影，默认不写是外阴影，inset表示内阴影
         */
        box-shadow: 20px 20px 10px red;
    }

    .test2 {
        width: 100px;
        height: 100px;
        background-color: red;
        /* opacity设置标签的透明度，默认是1表示完全不透明，而0表示完全透明 */
        opacity: 0.3;
    }

    .test3 {
        text-align: center;
        font-size: 18px;
        width: 200px;
        height: 200px;
        background-color: rgb(241, 198, 105);
        /* 圆角设置，可以设置1到4个值，直接设置圆角半径px像素或者使用百分比表示占用宽高作为圆角半径
           设置一个值：4个角的圆角都是这个值
           设置两个值：第一给值是左上角和右下角的圆角值，第二个值是右上角和左下角的圆角值
           设置三个值：第一给值是左上角的圆角值，第二个值是右上角和左下角的圆角值，第三个值是右下角的圆角值
           设置四个值：第一给值是左上角的圆角值，第二个值是右上角的圆角值，第三个值是右下角的圆角值，第四个值是左下角的圆角值
        */
        border-radius: 20px;
    }

    .test4 {
        text-align: center;
        font-size: 18px;
        width: 200px;
        height: 200px;
        background-color: rgba(235, 77, 77, 0.97);
        border-radius: 100%;
    }
</style>

<div class="test1">阴影设置</div>
<div class="test2">透明设置</div>
<div class="test3">20px的圆角</div>
<div class="test4">圆角100%的圆</div>
~~~

# css设置怪异盒子模型
~~~html
<style>
    .test1 {
        border: 3px violet solid;
        padding: 25px;
        width: 100px;
        height: 100px;
        background-color: mediumspringgreen;
        display: inline-block;
        /* 设置盒子模型方式，
           默认是content-box值表示width和height设置标签内容宽高，
           而border-box表示width和height设置标签总宽高（包含边框、内边距，如果设置了这些会默认挤压内容宽高变小，这种也叫怪异盒子模型）
        */
        box-sizing: border-box
    }

    .test2 {
        border: 3px violet solid;
        padding: 25px;
        width: 100px;
        height: 100px;
        background-color: red;
        display: inline-block;
    }
</style>

<div class="test1">标准盒子模型</div>
<div class="test2">怪异盒子模型</div>
~~~

# css设置过渡：过渡是指触发一个事件发生随着时间改变其CSS属性，并且按照一定方式显示变化过程
~~~html
<style>
    .test1 {
        width: 100px;
        height: 100px;
        background-color: pink;
        /* 过渡是指触发一个事件发生随着时间改变其CSS属性，并且按照一定方式显示变化过程 */
        /* transition-property设置哪些属性需要过渡属性，需要使用逗号分隔
           可以过渡的CSS属性只要能用数字表示的属性，其他属性无法过渡
           可以使用all来表示所有能过渡的属性
         */
        /* transition-property: width, height, background-color; */

        /* transition-duration设置过渡消耗时长
           注意可以按照transition-property设置的属性过渡先后顺序分别给这些CSS属性的设置过渡时间，但是需要使用逗号分隔否则过渡不生效
           只设置一个时间表示所有属性的过渡消耗时长一致
        */
        /* transition-duration: 1s, 5s, 5s; */

        /* transition-timing-function设置过渡时的变化方式，
           默认是 ease，即速度由慢到快再到慢，
           常用的还有 linear表示线性变化速度均匀、ease-in表示先慢后快、ease-out表示先快后慢、steps(n)表示n个分刻度过渡（不够柔和）
        */
        /* transition-timing-function: linear; */

        /* transition-delay设置触发事件时过渡的延迟时间，即多少秒后执行过渡效果，默认是0s不延迟 */
        /* transition-delay: 1s; */

        /* 上述过渡属性的复合写法，默认第一个时间是transition-duration，第二个时间是transition-delay 
           如果需要设置不同属性的过渡时间请手动设置transition-duration属性
        */
        transition: all 1s linear 0s;
    }

    /* 悬浮过渡的最终样式 */
    .test1:hover {
        width: 300px;
        height: 300px;
        background-color: red;
    }
</style>

<div class="test1"></div>
~~~

# css设置过渡动画效果
~~~html
<style>
    .test {
        width: 100px;
        height: 100px;
        text-align: center;
        background-color: red;
        /* 设置动画：
           第一个值name是自定义的动画名称，
           第二个值duration是动画的周期时间，
           第三个值动画执行的速率timing-function可以设置ease(逐渐变慢默认是这个)、linear（匀速）、ease-in（加速）、
           ease-out（减速）、ease-in-out（先加速后减速），
           第四个值delay是设置动画开始的延时时间，
           第五个值iteration-count是设置动画的循环次数，默认是1次，infinite是无限次
           第六个值direction设置动画播放的方向，默认是normal是向前播放，alternate表示在偶数次向前播放在奇数次反向播放
           注意还可以添加其他属性值，但是duration是动画的周期时间必须在delay设置动画开始的延时时间之前
        */
        animation: myAnim 8s ease-in-out 0.5s infinite alternate;
    }

    .test:hover {
        /* running代表运行动画效果，paused代表暂停动画效果 */
        animation-play-state: paused;
    }


    /* myAnim是自己定义的动画名称 */
    @keyframes myAnim {

        /* from指定动画开始帧（可以不写使用元素默认样式），to指定动画结束帧。
           但是使用它们只有一个动画变化，推荐使用百分比，从0%到100%之间可以设置多个变化帧
        */
        /* from{

        }
        to{

        } */

        0%{
            width: 100px;
            background-color: red;
        }

        12.5%{
            width: 200px;
            background-color: pink;
        }
        25%{
            width: 300px;
            background-color: rgb(204, 0, 255);
        }

        37.5%{
            width: 400px;
            background-color: blue;
        }

        50%{
            width: 500px;
            background-color: rgb(0, 234, 255);
        }

        62.5%{
            width: 600px;
            background-color: rgb(30, 255, 0);
        }
        75%{
            width: 700px;
            background-color: yellow;
        }
        87.5%{
            width: 800px;
            background-color: orange;
        }
        100%{
            width: 900px;
            background-color: red;
        }
    }
</style>

<div class="test">鼠标滑倒背景上暂停动画效果</div>
~~~

# css设置文本报纸排版
~~~html
<style>
    .test {
        font-size: 60px;
        font-weight: bolder;
        text-align: center;
        margin: 0px auto;
    }

    .test1 {
        /* 设置段落列数 */
        column-count: 4;

        /* 设置每个段落的宽度，如果父元素宽度放不下就按照这个宽度放下最大的段落列数 */
        column-width: 200px;

        /* 设置段落之间纵向的宽度间距 */
        column-gap: 20px;

        /* 设置段落的分割线颜色 */
        /* column-rule-color: red; */
        /* 设置段落的分割线线条条类型 */
        /* column-rule-style: solid; */
        /* 设置段落的分割线的宽度 */
        /* column-rule-width: 3px; */
        /* 设置段落的分割线多个属性，包含分割线颜色、线条类型、宽度 */
        column-rule: red solid 3px;
    }
</style>

<div class="test">报纸标题</div>
<div class="test1">
    <p>二愣子睁大着双眼，直直望着茅草和烂泥糊成的黑屋顶，身上盖着的旧棉被，已呈深黄色，看不出原来的本来面目，还若有若无的散发着淡淡的霉味。
        在他身边紧挨着的另一人，是二哥韩铸，酣睡的十分香甜，从他身上不时传来轻重不一的阵阵打呼声。
        离床大约半丈远的地方，是一堵黄泥糊成的土墙，因为时间过久，墙壁上裂开了几丝不起眼的细长口子，
        从这些裂纹中，隐隐约约的传来韩母唠唠叨叨的埋怨声，偶还掺杂着韩父，抽旱烟杆的“啪嗒”“啪嗒”吸允声。
        二愣子缓缓的闭上已有些发涩的双目，迫使自己尽早进入深深的睡梦中。
        他心里非常清楚，再不老实入睡的话，明天就无法早起些了，也就无法和其他约好的同伴一进山拣干柴。
        二愣子姓韩名立，这么像模像样的名字,他父母可起不出来，这是他父亲用两个粗粮制成的窝头，求村里老张叔给起的名字。
        老张叔年轻时，曾经跟城里的有钱人当过几年的伴读书童，是村里唯一认识几个字的读书人，村里小孩子的名字，倒有一多半是他给起的。
        韩立被村里人叫作“二愣子”，可人并不是真愣真傻，反而是村中首屈一指的聪明孩子，但就像其他村中的孩子一样，
        除了家里人外，他就很少听到有人正式叫他名“韩立”，倒是“二愣子”“二愣子”的称呼一直伴随至今。
        而之所以被人起了个“二愣子”的绰号，也只不过是因为村里已有一个叫“愣子”的孩子了。
        这也没啥，村里的其他孩子也是“狗娃”“二蛋”之类的被人一直称呼着，这些名字也不见得比“二愣子”好听了哪里去。
        因此，韩立虽然并不喜欢这个称呼，但也只能这样一直的自我安慰着。
    </p>
    <p>韩立外表长得很不起眼，皮肤黑黑的，就是一个普通的农家小孩模样。但他的内心深处，却比同龄人早熟了许多，
        他从小就向往外面世界的富饶繁华，梦想有一天他能走出这个巴掌大的村子，去看看老张叔经常所说的外面世界。
        当韩立的这个想法，一直没敢和其他人说起过。否则，一定会使村里人感到愕然，一个乳臭未干的小屁孩，
        竟然会有这么一个大人也不敢轻易想的念头。要知道，其同韩立差不多大的小孩，都还只会满村的追鸡摸狗，更别说会有离开故土，这么一个古怪的念头。
        韩立一家七口人，有两个兄长，一个姐姐，还有一个小妹，他在家里排行老四，今年刚十岁，
        家里的生活很清苦，一年也吃不上几顿带荤腥的饭菜，全家人一直在温线上徘徊着。
        此时的韩立，正处于迷迷糊糊，似睡未睡之间，恼中还一直残留着这样的念头：上山时，一定要帮他最疼爱的妹妹，多拣些她最喜欢吃的红浆果。
        第二天中午时分，当韩立顶着火辣辣的太阳，背着半人高的木柴堆，怀里还揣着满满一布袋浆果，
        从山里往家里赶的时侯，并不知道家中已来了一位，会改变他一生运的客人。
        这位贵客，是跟他血缘很近的一位至亲，他的亲三叔。
        听说，在附近一个小城的酒楼，给人当大掌柜，是他父母口中的大能人。韩家近百年来，可能就出了三叔这么一位有点身份的亲戚。
        韩立只在很小的时侯，见过这位三叔几次。他大哥在城里给一位老铁匠当学徒的工作，就是这位三叔给介绍的，
        这位三叔还经常托人给他父母捎带一些吃的用的东西很是照顾他们一家，因此韩立对这位三叔的印像也很好，知道父母虽然嘴里不说，心里也是很感激的。
    </p>
    <p>大哥可是一家人的骄傲，听说当铁匠的学徒，不但管吃管住，一个月还有三十个铜板拿，等到正式出师被人雇用时，挣的钱可就更多了。
        每当父母一提起大哥，就神采飞扬，像换了一个人一样。
        韩立年龄虽小，也羡慕不已，心目最好的工作也早早就有了，就是给小城里的哪位手艺师傅看上，收做学徒从此变成靠手艺吃饭的体面人。
        所以当韩立见到穿着一身崭新的缎子衣服，胖胖的圆脸，留着一撮小胡子的三叔时，心里兴奋极了。
        把木柴在屋后放好后，便到前屋腼腆的给三叔见了个礼，乖乖的叫了声：“三叔好”，就老老实实的站在一边，听父母同三叔聊天。
        三叔笑眯眯的望着韩立，打量着他一番，嘴里夸了他几句“听话”“懂事”之类的话，然后就转过头，和他父母说起这次的来意。
        韩立虽然年龄尚小，不能完全听懂三叔的话，但也听明白了大概的意思。
        原来三叔工作的酒楼，属于一个叫“七玄门”的江湖门派所有，这个门派有外门和内门之分，
        而前不久，三叔才正式成为了这个门派的外门弟子，能够推举7岁到12岁孩童去参加七玄门招收内门弟子的考验。
        五年一次的“七玄门”招收内门弟子测试，下个月就要开始了。这位有着几分精明劲自己尚无子女的三叔，自然想到了适龄的韩立。
    </p>
    <p>一向老实巴交的韩父，听到“江湖”“门派”之类的从未听闻过的话，心里有些犹豫不决拿不定主意。
        便一把拿起旱烟杆，“吧嗒”“吧嗒”的狠狠抽了几口，就坐在里，一声不吭。
        在三叔嘴里，“七玄门”自然是这方圆数百里内，了不起的、数一数二的大门派。
        只要成为内门弟子，不但以后可以免费习武吃喝不愁，每月还能有一两多的散银子零花。
        而且参加考验的人，即使未能入选也有机会成为像三叔一样的外门人员，专替“七玄门”打理门外的生意。
        当听到有可能每月有一两银子可拿，还有机会成为和三叔一样的体面人，韩父终于拿定了主意，答应了下来。
        三叔见到韩父应承了下来，心里很是高兴。又留下几两银子，说一个月后就来带韩立走，在这期间给韩立多做点好吃的，给他补补身子，好应付考验。
        随后三叔和韩打声招呼，摸了摸韩立的头，出门回城了。
        韩立虽然不全明白三叔所说的话，但可以进城能挣大钱还是明白的。
        一直以来的愿望，眼看就有可能实现，他一连好几个晚上兴奋的睡不着觉。
        三叔在一个多月后，准时的来到村中，要带韩立走了，临走前韩父反复嘱咐韩立，做人要老实，遇事要忍让，别和其他人起争执，而韩母则要他多注意身体，要吃好好。
    </p>
    <p>在马车上，看着父母渐渐远去的身影，韩立咬紧了嘴唇，强忍着不让自己眼框中的泪珠流出来。
        他虽然从小就比其他孩子成熟的多，但毕竟还是个十岁的小孩，第一次出远门让他的心里有点伤感和彷徨。
        他年幼的心里暗暗下定了决心，等挣到了大钱就马上赶来，和父母再也不分开。
        韩立从未想到，此次出去后钱财的多少对他已失去了意义，他竟然走上了一条与凡人不同的仙业大道，走出了自己的修仙之路。（未完待续）
    </p>
</div>
~~~

# css
~~~html
<style>
</style>
~~~

# css
~~~html
<style>
</style>
~~~











