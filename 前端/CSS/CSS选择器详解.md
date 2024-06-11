# css选择器（下面都有案例详解）
- 通配选择器：使用 * 进行选择所有的html标签，一般是用于清除样式
- 元素选择器：直接选中html元素标签
- 类选择器：使用 .class值 选中class值相同的元素标签，同一元素标签可以指定多个类名，使用空格分隔不同的class值
- id选择器：使用 .id值 选中id值相同的元素标签
- 交集选择器：使用 标签名.class值1.class值2 选中同时符合标签名和所有class值的元素标签，注意如果有标签名设置一定要在最前面
- 并集选择器：使用 逗号分隔不同的选择器 选中符合其中之一选择器的元素标签
- 后代选择器：使用 祖辈选择器 后辈选择器 更低辈份选择器 选中符合高辈份选择器的低辈份元素标签，即符合要求的后代都选中
- 子代选择器：使用 父辈选择器 子辈选择器 选中符合父辈选择器下的同时符合子辈选择器的元素标签，即符合要求的儿子辈都选中
- 兄弟选择器：见下面的详解
- 属性选择器：见下面的详解
- 伪类选择器
  - 动态伪类选择器：见下面的详解
  - 结构伪类选择器：见下面的详解
  - 否定伪类选择器：使用 选择器1:not(选择器2) 选中满足选择器1不满足选择器2的元素标签
  - UI伪类选择器：见下面的详解
- 伪元素选择器：见下面的详解

# css选择器优先级
- 简单的选择器优先级比较：css的样式属性值后添加!important（注意只是改一个属性值，而不是整个选择器） > 行内样式 > id选择器 > 类选择器 > 元素选择器 > 通配选择器 
- 复杂的选择器优先级比较：按照权重比较
  - 权重计算的结果是（a, b, c），a是id选择器的个数，b是类、伪类、属性选择器的个数，c是元素、伪元素选择器的个数
  - 注意并集选择器的权重是分开计算的
  - 权重比较是按照a、b、c依次比较，数字大的优先级高，当然如果a、b、c的权重都一样则按照css的顺序，后来者生效


### 通配选择器：使用通配符选中元素标签
~~~html
<style>
    * {
        color: red;
        font-size: 30px;
    }
</style>

<h1>通配选择器</h1>
<p>哈哈哈</p>
~~~

### 元素选择器：直接选中html元素标签
~~~html
<style>
    h1 {
        color: red;
        font-size: 60px;
    }
    p {
        color: green;
        font-size: 40px;
    }
</style>

<h1>元素选择器</h1>
<p>哈哈哈</p>
~~~

### 类选择器：使用 .class值 选中class值相同的元素标签
~~~html
<style>
    .test1 {
        color: blue;
        font-size: 60px;
    }
    .test2 {
        color: yellow;
        font-size: 40px;
    }
</style>

<p class="test1">类选择器</p>
<p class="test2">哈哈哈</p>
~~~

### id选择器：直接选中html元素标签
~~~html
<style>
    #test1 {
        color: pink;
        font-size: 60px;
    }
    #test2 {
        color: purple;
        font-size: 40px;
    }
</style>

<h1 id="test1">id选择器</h1>
<p id="test2">哈哈哈</p>
~~~

### 交集选择器：使用 标签名.class值1.class值2 选中同时符合标签名和所有class值的元素标签，注意如果有标签名设置一定要在最前面
~~~html
<style>
    h1.test1 {
        color: blue;
        font-size: 60px;
    }
    p.test2.test3 {
        color: yellow;
        font-size: 40px;
    }
</style>

<h1 class="test1">类选择器</h1>
<p class="test2 test3">哈哈哈</p>
~~~

### 并集选择器：使用 逗号分隔不同的选择器 选中符合其中之一选择器的元素标签
~~~html
<style>
    h1,
    .test1,
    .test2 {
        color: blue;
        font-size: 60px;
    }
</style>

<h1>类选择器</h1>
<p class="test1">哈哈哈1</p>
<p class="test2">哈哈哈2</p>
~~~

### 后代选择器：使用 祖辈选择器 后辈选择器 更低辈份选择器 选中符合高辈份选择器的低辈份元素标签，即符合要求的后代都选中
~~~html
<style>
    ul li {
        color: red;
        font-size: 60px;
    }

    ul .test1 {
        color: darkorchid;
        font-size: 60px;
    }

    ul div p {
        color: darkcyan;
        font-size: 60px;
    }
</style>

<ul>
    <li>qqqq</li>
    <li>wwww</li>
    <p class="test1">xxxx</p>
    <div>
        <p>eeee</p>
        <p>rrrr</p>
        <li>ffff</li>
    </div>
</ul>
~~~

### 子代选择器：使用 父辈选择器 子辈选择器 选中符合父辈选择器下的同时符合子辈选择器的元素标签，即符合要求的儿子辈都选中
~~~html
<style>
    ul>li {
        color: red;
        font-size: 60px;
    }

    div>li {
        color: darkcyan;
        font-size: 60px;
    }
</style>

<ul>
    <li>qqqq</li>
    <li>wwww</li>
    <p class="test1">xxxx</p>
    <div>
        <p>eeee</p>
        <p>rrrr</p>
        <li>ffff</li>
    </div>
</ul>
~~~

### 兄弟选择器
- 使用 选择器1+选择器2 选中选择器1与它紧紧相连（之间不相隔任何元素）且符合选择器2的下一个元素标签（上一个是不算的）
- 使用 选择器1~选择器2 选中选择器1与它同一级别且符合选择器2的全部元素标签
~~~html
<style>
    li+p {
        color: red;
        font-size: 60px;
    }

    p~li {
        color: darkcyan;
        font-size: 60px;
    }
</style>

<ul>
    <li>qqqq</li>
    <li>wwww</li>
    <p class="test1">xxxx</p>
    <div>
        <p>eeee</p>
        <li>rrrr</li>
        <li>ffff</li>
    </div>
</ul>
~~~

### 属性选择器
- 使用 [标签属性名称] 选中带有指定标签属性名称的元素标签
- 使用 [标签属性名称="值"] 选中带有指定标签属性名称和值的元素标签
- 使用 [标签属性名称^="前缀值"] 选中带有指定标签属性名称和包含前缀值的元素标签
- 使用 [标签属性名称$="后缀值"] 选中带有指定标签属性名称和包含后缀值的元素标签
- 使用 [标签属性名称*="子字符串值"] 选中带有指定标签属性名称和包含子字符串值的元素标签
~~~html
<style>
    [title1] {
        color: red;
        font-size: 60px;
    }

    [title2="wwww"] {
        color: darkcyan;
        font-size: 60px;
    }

    [title3^="e"] {
        color: pink;
        font-size: 60px;
    }

    [title4$="r"] {
        color: khaki;
        font-size: 60px;
    }

    [title5*="f"] {
        color: magenta;
        font-size: 60px;
    }
</style>

<p title1="qqqq">qqqq</p>
<p title2="wwww">wwww</p>
<p title3="emmm">emmm</p>
<p title4="yyyr">yyyr</p>
<p title5="nfhb">nfhb</p>
~~~

### 动态伪类选择器：当触发一些操作产生的状态会附加在元素标签上，而动态伪类选择器就是根据这些状态选中元素标签
~~~html
<style>
    /* a:link 伪类选择器选中的是没有点击访问过的a元素标签 
       :link 是a标签独有的状态，其他标签没有
    */
    a:link {
        color: red;
        font-size: 30px;
    }

    /* a:visited 伪类选择器选中的是有点击访问过的a元素标签 
       :visited 是a标签独有的状态，其他标签没有
    */
    a:visited {
        color: darkcyan;
        font-size: 30px;
    }

    /* a:hover 伪类选择器选中的是鼠标正在悬浮的a元素标签 
       :hover 是所有标签都有的状态
    */
    a:hover {
        color: skyblue;
        font-size: 30px;
    }

    /* a:active 伪类选择器选中的是鼠标正在点击但是还未抬起鼠标时的a元素标签 
       :active 是所有标签都有的状态
    */
    a:active {
        color: greenyellow;
        font-size: 30px;
    }

    /* input:focus 伪类选择器选中的是输入表单焦点聚集时的input元素标签
       :focus 是input或select标签独有的状态（注意只有input的单选框和多选框不使用这个修改焦点样式），其他标签没有
    */
    input:focus {
        background-color: wheat;
        width: 300px;
    }

    select:focus {
        background-color: turquoise;
        width: 300px;
    }
</style>

<a href="https://www.bilibili.com/" target="_blank">去哔哩哔哩</a><br>
<a href="https://pifa.pinduoduo.com" target="_blank">去拼多多</a><br>

<input type="text" name="abc" placeholder="焦点聚集样式变更"><br>
<input type="text" name="abc" placeholder="焦点聚集样式变更"><br>
<input type="text" name="abc" placeholder="焦点聚集样式变更"><br>

<select>
    <option value="河北省">河北</option>
    <option value="南京市">南京</option>
    <option value="河南省">河南</option>
    <option value="北京省">北京</option>
</select>
~~~


### 结构伪类选择器
- 使用 选择器1:first-child 先选中选择器1的元素标签，最终选中的是这些元素标签同一级任意类型(前提是选择器1不是元素选择器)的元素标签（兄弟标签）中的第一个元素标签
- 使用 选择器1:last-child 先选中选择器1的元素标签，最终选中的是这些元素标签同一级任意类型(前提是选择器1不是元素选择器)的元素标签（兄弟标签）中的最后一个元素标签
- 使用 选择器1:nth-child(n) 先选中选择器1的元素标签，最终选中的是这些元素标签同一级任意类型(前提是选择器1不是元素选择器)的元素标签（兄弟标签）中的第n个元素标签
  - 括号中可以写的形式是：an+b，a和b都是常数
  - 括号中写2n或even表示选中偶数元素
  - 括号中写2n+1或odd表示选中奇数元素
  - 括号中写-n+5表示选中前5个元素
- 使用 选择器1:first-of-type 先选中选择器1的元素标签，最终选中的是这些元素标签同一级且同一类型的元素标签（兄弟标签）中的第一个元素标签
- 使用 选择器1:last-of-type 先选中选择器1的元素标签，最终选中的是这些元素标签同一级且同一类型的元素标签（兄弟标签）中的最后一个元素标签
- 使用 选择器1:nth-of-type(n) 先选中选择器1的元素标签，最终选中的是这些元素标签同一级且同一类型的元素标签（兄弟标签）中的第n个元素标签
  - 括号中可以写的形式是：an+b，a和b都是常数
  - 括号中写2n或even表示选中偶数元素
  - 括号中写2n+1或odd表示选中奇数元素
  - 括号中写-n+5表示选中前5个元素
~~~html
<style>
    p:first-child {
        color: red;
        font-size: 30px;
    }

    p:last-child {
        color: violet;
        font-size: 30px;
    }

    p:nth-child(3) {
        color: lightcoral;
        font-size: 30px;
    }

    span:first-of-type {
        color: orange;
        font-size: 30px;
    }

    span:last-of-type {
        color: yellowgreen;
        font-size: 30px;
    }

    span:nth-of-type(3) {
        color: purple;
        font-size: 30px;
    }
</style>

<div>
    <p >qqqq</p>
    <p >wwww</p>
    <p >xxxx</p>
    <h1 >kkkk</h1>
    <p >mmmm</p>
</div>
<div>
    <span >uuuu</span><br>
    <span >oooo</span><br>
    <h1 >llll</h1><br>
    <span >ffff</span><br>
    <span >vvvv</span><br>
</div>
~~~

### 否定伪类选择器：使用 选择器1:not(选择器2) 选中满足选择器1不满足选择器2的元素标签
~~~html
<style>
    p:not(.test) {
        color: red;
        font-size: 30px;
    }
</style>

<div>
    <p >qqqq</p>
    <p class="test">wwww</p>
    <p >xxxx</p>
    <p >mmmm</p>
</div>
~~~

### UI伪类选择器
- 使用 选择器1:checked 选中焦点聚集状态且满足选择器1的input元素标签，这个一般是用于单选框和多选框（如果是需要选中文本相关的输入框焦点聚集状态还是使用input:focus）
- 使用 选择器1:disabled 选中满足选择器1且属性包含disabled禁止输入的input元素标签
~~~html
<style>
    input:checked {
        width: 100px;
        height: 100px;
    }

    input:disabled {
        background-color: navajowhite;
    }
</style>

性别：
<input type="radio" name="gender" value="male">男
<input type="radio" name="gender" value="female">女
<br>

爱好：
<input type="checkbox" name="hobby" value="basketball">篮球
<input type="checkbox" name="hobby" value="football">足球
<input type="checkbox" name="hobby" value="ping pong">乒乓球
<br>

<input type="text" disabled><br>
~~~

### 伪元素选择器：通过伪元素选择器选中元素的一部分（内容或者是属性），修改这部分的样式
~~~html
<style>
    /* 使用 选择器1:first-letter 选中满足选择器1的元素标签中内容的第一个文本文字 */
    div::first-letter {
        color: red;
    }

    /* 使用 选择器1:first-line 选中满足选择器1的元素标签中内容的最后一个文本文字 */
    div::first-line {
        color: magenta;
        background-color: mediumspringgreen;
    }

    /* 使用 选择器1:selection 选中满足选择器1的元素标签中内容被鼠标选中时文本文字 */
    div::selection {
        color: orange;
        background-color: greenyellow;
    }

    /* 使用 选择器1:placeholder 选中满足选择器1的input元素标签中的placeholder属性值 */
    input::placeholder {
        color: darkorchid;
    }

    /* 使用 选择器1:before 选中满足选择器1的元素标签中文本内容的开始位置 
       content 设置添加的前缀
    */
    p::before {
        content: "￥";
    }

    /* 使用 选择器1:after 选中满足选择器1的元素标签中文本内容的结束位置 
       content 设置添加的后缀
    */
    p::after {
        content: ".00";
    }
</style>

<div>
    滕王高阁临江渚，佩玉鸣鸾罢歌舞。<br>
    画栋朝飞南浦云，珠帘暮卷西山雨。<br>
    闲云潭影日悠悠，物换星移几度秋。<br>
    阁中帝子今何在？槛外长江空自流。<br>
</div>
<input type="text" placeholder="请填写它的作者?">
<p>19</p>
~~~







