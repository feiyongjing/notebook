~~~HTML

<!-- 文档头标签 -->
<head></head>   

<!-- 文档元数据 -->
<meta>

<!-- css样式代码 -->
<style></style>

<!-- js代码
     如果有src属性就是引入外部的js文件代码
 -->
<script src="../js/hello.js"></script>

<!-- 引入外部链接资源，
     rel属性设置链接方式与html文档之间的关系，stylesheet表示是样式表
     href属性设置外部资源的路径
 -->
<link rel="stylesheet" href="../css/selectorType.css">

<!-- 文档体标签 -->
<body></body>   

<!-- 换行标签 -->
<br />

<!-- 段落标签 -->
<p></p>

<!-- 块标签 -->
<div></div>

<!-- 行标签 -->
<span></span>

<!-- 链接标签：href设置链接 -->
<a href="../img/OIP-C.jfif">a标签点击文字跳转指定页面</a>

<!-- 图片链接：src设置图片链接，width和heighe设置图片宽高 -->
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
<video src="视频路径" autoplay width="1000" heigth="800" controls loop muted poster="图片路径"> 视频显示失败时显示这行文字 </video>


<ul>
    <li>无序列表ul+li</li>
    <li>无序列表ul和li的属性type表示列表元素前的图案，属性值有disc、circle、square分别代表黑色实心圆、空心圆、实心矩形</li>
    <li>xxxx</li>
<ul>

<ol>
    <li>有序列表ol+li</li>
    <li>有序列表ol和li的属性type表示列表元素前的有序模板，属性值有A、a、1、I分别表示大写字母模板、小写字母模板、数字模板、罗马数字模板。start属性表示按照type属性模板的第几个开始，reversed属性无需设置属性值表示模板按照倒序排列
    </li>
    <li>有序列表ol+li</li>
</ol>

<dl>
    <dd>无序列表dl+dd</dd>
    <dd>xxxx</dd>
    <dd>xxxx</dd>
</dl>


<table> 
    <caption>表格标题1</caption>
    <thead> <!-- 表格头 -->
        <tr>
            <th></th>
            <th>中文</th>
            <th>英语</th>
        </tr>
    </thead>
    <tbody> <!-- 表格数据 -->
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

<!-- form表单标签 -->
<!-- input输入标签，type设置各式各样的输入 -->
<!-- button按钮标签 -->
<form action="服务器的地址" class="formEvent" method="post">
    <input type="text" class="formOninput" name="参数名称1">
    <input type="text" class="formOnSelect" name="参数名称2">
    <input type="text" class="formOnChange" name="参数名称3">
    <button class="formReset">重置输入框的内容</button>
    <button>提交表单</button>
</form>


~~~