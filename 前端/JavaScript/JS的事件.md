## html事件、DOM0级事件、DOM2级事件
- 事件本身是有3个阶段捕获阶段、目标阶段、冒泡阶段
  - 捕获阶段会由先捕获祖先元素，然后是下面逐层的后代元素
  - 目标阶段是捕获到阶段传递到最终事件触发的元素，这时会触发事件
  - 冒泡阶段会由目标阶段向外传播（逐层向祖先元素传播），触发祖先元素绑定的事件
- 如果将事件函数设置为null，则会解除事件绑定
~~~html
<style>
    .test {
        width: 200px;
        height: 200px;
        background-color: palegreen;
    }
</style>

<button class="test test1" onclick="clickHandle('触发了鼠标点击html事件')">html事件按钮</button>
<button class="test test2">DOM0级事件按钮</button>
<button class="test test3">DOM2级事件按钮</button>

<script>
    // html事件绑定，不推荐使用，原因是js和html没有分开
    var test1Element = document.querySelector(".test1");
    function clickHandle(str) {
        console.log(str)
    }

    // DOM0级事件绑定，js和html是分离的，但是只能触发一个事件(后面的事件覆盖前面的事件)，无法同时触发多个事件
    var test2Element = document.querySelector(".test2");
    // 添加鼠标点击DOM0级事件
    test2Element.onclick = function () {
        console.log("触发了鼠标点击DOM0级事件")
    }
    // test2Element.setAttribute("onclick", "clickHandle('触发了鼠标点击DOM0级事件')");

    // DOM2级事件绑定，js和html是分离的，可以同时触发多个事件
    var test3Element = document.querySelector(".test3");
    // 添加鼠标点击DOM2级事件1，注意addEventListener可以传递第3个参数表示是否在捕获阶段被捕获时触发该事件，默认是false
    test3Element.addEventListener("click",function(){
        console.log("触发了鼠标点击DOM2级事件：第一个事件")
    })
    // 添加鼠标点击DOM2级事件2
    test3Element.addEventListener("click",function(){
        console.log("触发了鼠标点击DOM2级事件：第二个事件")
    })


    // offsetLeft和offsetTop代表标签与父级元素的相对位置，offsetLeft会多8像素，这是浏览器自带的8像素
    console.log("标签的offsetLeft属性，" + test1Element.offsetLeft);
    console.log("标签的offsetTop属性，" + test1Element.offsetTop);

    console.log("视口高度（屏幕高度）是" + document.documentElement.clientHeight);
    console.log("页面高度是" + document.body.clientHeight);
    //scrollLeft和scrollTop代表屏幕的向右滚动和向下滚动的距离像素
    console.log("标签的scrollLeft属性，向右滚动的距离像素：" + document.documentElement.scrollLeft);
    console.log("标签的scrollTop属性，向下滚动的距离像素：" + document.documentElement.scrollTop);
</script>
~~~

## 事件对象的常见属性
~~~html
<button class="test1">从Event事件对象中获取元素</button>
<div>取消浏览器对当前事件的默认行为，比如a标签的跳转 <a class="test2" href="https://www.bilibili.com/">点击跳转到bilibili</a> </div>
<div class="test3" style="width: 200px;height: 200px;background-color: green;">
    <div class="test3" style="width: 100px;height: 100px;background-color: yellow;">
        禁止在子元素与父元素重叠的部分触发事件时触发父元素事件，而是只触发子元素事件
    </div>
</div>

<script>
    var test1Element = document.querySelector(".test1");
    // Event事件对象就是参数
    test1Element.onclick = function (event) {
        // 事件对象的target属性就是触发事件的元素
        console.log(event.target);
        event.target.innerHTML = "按钮内容使用Event事件对象修改";
        console.log("当前按钮的事件类型：" + event.type);
    }

    var test2Element = document.querySelector(".test2");
    // Event事件对象 preventDefault取消浏览器对当前事件的默认行为，比如a元素的跳转
    test2Element.onclick = function (event) {
        event.preventDefault();
        console.log("取消浏览器对当前事件的默认行为");
    }

    // 父元素的css样式
    var test3Element = document.querySelector(".test3");
    // 子元素的css样式
    var test3Element = document.querySelector(".test3");

    test3Element.onclick = function (event) {
        console.log("父元素事件触发");
    }
    test3Element.onclick = function (event) {
        // 禁止在子元素与父元素重叠的部分触发事件时触发父元素事件，而是只触发子元素，这叫阻止事件冒泡
        event.stopPropagation();

        // 设置 event.cancelBubble=true 也可以阻止事件冒泡
        console.log("子元素事件触发");
    }
</script>
~~~

## 鼠标事件
~~~html
<div class="mouseEvent">
    <button class="mouseClick">鼠标单击文字变蓝</button>
    <button class="mouseDoubleClick">鼠标双击背景变粉色</button>
    <button class="onMouseDown">鼠标按下字体变绿色</button>
    <button class="onMouseUp">鼠标抬起字体变青</button>
    <div class="onMouseMove">鼠标移动背景变红</div>
    <div class="onMouseEnter">鼠标进入背景变红</div>
    <div class="onMouseLeave">鼠标离开背景变红</div>
    <div class="onMouseOverFather">
        鼠标进入父元素，父元素背景变红，进入子元素会再次触发一次
        <div class="subOnMouseOverSon"></div>
    </div>
    <div class="onMouseOutFather">
        鼠标离开父元素背景（包含从子元素离开）或从父元素背景进入子元素变红，从子元素离开父元素会再次触发一次
        <div class="onMouseOutSon"></div>
    </div>
    <div class="onMouseonWheel">鼠标滚轮滑动背景变红</div>
</div>

<script>
    var mouseClickElement = document.querySelector(".mouseClick");
    mouseClickElement.onclick=function (event) {
        mouseClickElement.style.cssText = "color: blue";
        console.log("鼠标单击事件触发");
    }

    var mouseDoubleClickElement = document.querySelector(".mouseDoubleClick");
    mouseDoubleClickElement.ondblclick=function (event) {
        mouseDoubleClickElement.style.cssText = "background-color: pink";
        console.log("鼠标双击事件触发");
    }

    var onMouseDownElement = document.querySelector(".onMouseDown");
    onMouseDownElement.onmousedown=function (event) {
        onMouseDownElement.style.cssText = "color: rgb(0,255,0)";
        console.log("鼠标按下事件触发");
    }

    var onMouseUpElement = document.querySelector(".onMouseUp");
    onMouseUpElement.onmouseup=function (event) {
        onMouseUpElement.style.cssText = "color: rgb(6, 245, 229)";
        console.log("鼠标抬起事件触发");
    }

    var onMouseMoveElement = document.querySelector(".onMouseMove");
    onMouseMoveElement.style.cssText = "width: 100px;height: 100px;background-color: yellow;";
    onMouseMoveElement.onmousemove=function (event) {
        onMouseMoveElement.style.backgroundColor="red"
        console.log("鼠标移动事件触发");
    }

    var onMouseEnterElement = document.querySelector(".onMouseEnter");
    onMouseEnterElement.style.cssText = "width: 100px;height: 100px;background-color: rgb(240, 160, 55);";
    onMouseEnterElement.onmouseenter=function (event) {
        onMouseEnterElement.style.backgroundColor="red"
        console.log("鼠标进入事件触发");
    }

    var onMouseLeaveElement = document.querySelector(".onMouseLeave");
    onMouseLeaveElement.style.cssText = "width: 100px;height: 100px;background-color: rgb(206, 248, 117);";
    onMouseLeaveElement.onmouseleave=function (event) {
        onMouseLeaveElement.style.backgroundColor="red"
        console.log("鼠标离开事件触发");
    }

    var onMouseOverFatherElement = document.querySelector(".onMouseOverFather");
    var subOnMouseOverSonElement = document.querySelector(".subOnMouseOverSon");
    onMouseOverFatherElement.style.cssText = "width: 200px;height: 200px;background-color: rgb(206, 248, 117);";
    subOnMouseOverSonElement.style.cssText = "width: 100px;height: 100px;background-color: rgba(0, 38, 255, 0.5);";
    onMouseOverFatherElement.onmouseover=function (event) {
        onMouseOverFatherElement.style.backgroundColor="red"
        console.log("鼠标进入父元素，父元素背景变红，进入子元素会再次触发一次");
    }

    var onMouseOutFatherElement = document.querySelector(".onMouseOutFather");
    var onMouseOutSonElement = document.querySelector(".onMouseOutSon");
    onMouseOutFatherElement.style.cssText = "width: 200px;height: 200px;background-color: rgb(206, 248, 117);";
    onMouseOutSonElement.style.cssText = "width: 100px;height: 100px;background-color: rgba(187, 0, 255, 0.5);";
    onMouseOutFatherElement.onmouseout=function (event) {
        onMouseOutFatherElement.style.backgroundColor="red"
        console.log("鼠标离开父元素背景（包含从子元素离开）或从父元素背景进入子元素变红，从子元素离开父元素会再次触发一次");
    }

    var onMouseWheelElement = document.querySelector(".onMouseonWheel");
    onMouseWheelElement.style.cssText = "width: 200px;height: 200px;background-color: rgb(206, 248, 117);";
    onMouseWheelElement.onmousewheel=function (event) {
        onMouseWheelElement.style.backgroundColor="red"
        console.log("鼠标滚轮滑动背景变红");
    }
</script>
~~~

## 键盘事件
~~~html
<div class="keyboardEvent">
    <input type="text" class="keydown" style="width: 100%;" placeholder="键盘事件敲下键盘触发背景变红">
    <input type="text" class="keypress" style="width: 100%;" placeholder="键盘事件敲下有值的键盘触发背景变红，Ctrl、Alt、Shift、win是无值键盘，当然同时按下Ctrl+a是可以触发的">
    <input type="text" class="keyup" style="width: 100%;" placeholder="键盘事件敲下键盘松开时触发背景变黄">
</div>

<script>
    var keydownElement = document.querySelector(".keydown");
    keydownElement.onkeydown=function (event) {
        keydownElement.style.backgroundColor="red"
        console.log("键盘事件敲下触发背景变红");
    }

    var keypressElement = document.querySelector(".keypress");
    keypressElement.onkeypress=function (event) {
        event.target.style.backgroundColor = "green"
        console.log("键盘事件敲下有值的键盘触发背景变绿，Ctrl、Alt、Shift、win是无值键盘，当然同时按下Ctrl+a是可以触发的");
    }

    var keyupElement = document.querySelector(".keyup");
    keyupElement.onkeyup=function (event) {
        event.target.style.backgroundColor = "yellow"
        console.log("键盘事件敲下键盘松开时触发背景变黄");
        console.log("keyCode是验证输入的唯一标识，比如回车的keyCode是13");
        if(event.keyCode==13){
            console.log("验证回车的keyCode是13"+keyCode);
        }
    }
</script>
~~~

## 表单事件
~~~html
<form action="服务器的地址" id="formEvent" method="post">
    <input type="text" class="formOninput" name="参数名称1" style="width: 100%;" placeholder="内容发生变化时实时读取输入的数据">
    <input type="text" class="formOnSelect" name="参数名称2" style="width: 100%;" placeholder="内容被选中了触发">
    <input type="text" class="formOnChange" name="参数名称3" style="width: 100%;" placeholder="内容被修改完成失去焦点或者是按下回车时触发">
    <button class="formReset">重置输入框的内容</button>
    <button>提交表单</button>
</form>

<script>
    var formOninputElement = document.querySelector(".formOninput");
    formOninputElement.oninput = function (event) {
        console.log("内容发生变化时实时读取输入的数据：", event.target.value);
    }

    var formOnSelectElement = document.querySelector(".formOnSelect");
    formOnSelectElement.onselect = function (event) {
        console.log("内容被选中了触发");
    }

    var formOnChangeElement = document.querySelector(".formOnChange");
    formOnChangeElement.onchange = function (event) {
        console.log("内容被修改完成失去焦点或者是按下回车时触发");
    }

    var formResetElement = document.querySelector(".formReset");
    var formElement = document.getElementById("formEvent");
    formResetElement.onclick = function (event) {
        event.preventDefault(); // 去除这个按钮元素的默认提交表单
        formElement.reset();
        console.log("清除表单的全部输入");
    }

    formElement.onsubmit = function (event) {
        console.log("提交表单的全部输入");
    }
</script>
~~~












