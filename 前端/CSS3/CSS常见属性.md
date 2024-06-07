# 常见属性
~~~
font-size           设置字体大小，不同浏览器的默认字体大小不同，所以建议手动设置字体大小，例子 .test {font-size: 30px}
font-weight         设置字体粗细
font-style          设置字体样式是否是斜体
font-family         设置字体样式
color               设置字体颜色
letter-spacing      设置字符间距
word-spacing        设置单词间距（只有英文文本生效）

background-color    设置背景颜色，一般是设置文本或者div块的背景颜色

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















