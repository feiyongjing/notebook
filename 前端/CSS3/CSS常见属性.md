# 常见属性
~~~
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
margin              设置标签元素的外边距，另外还有例如 margin-left 属性表示左外边距设置，其他的依此类推。注意元素标签的宽高实际是内边距+边框+内容宽高，并不包含外边距
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

display             设置标签元素显示方式：inline表示行内元素，block表示块元素，inline-block表示行内块元素，none表示隐藏该元素（元素不占位）
visibility          设置标签元素是否显示，默认是show显示，设置为hidden表示隐藏，但是实际还是会占据页面该元素的位置

overflow            设置超出元素标签的内容部分如何处理，另外设置hidden值忽略超出的部分可以解决子元素的外边距设置导致当前元素发送的移动
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

## css属性继承
- css中和盒子相关的属性都无法继承，例如外边距、边框、内边距、背景、内容宽高等
- 盒子相关的属性之外的属性都是可以继承父元素或祖先元素的属性

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

    /* background设置背景的多个属性，包含上述的background-color、background-image、background-repeat、background-position*/
    .test4 {
        width: 500px;
        height: 500px;
        background: mediumaquamarine url(../img/KFC.jfif) right center no-repeat;
    }
</style>

<div class="test1">宽200px高200px的背景设置为绿色</div>
<div class="test2">背景宽500px高500px设置图片不平铺</div>
<div class="test3">背景宽500px高500px设置图片不平铺且在左下</div>
<div class="test4">背景宽500px高500px设置图片不平铺且在右中</div>
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

# css
~~~html
<style>
</style>
~~~













