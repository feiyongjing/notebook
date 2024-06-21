### 定时器案例
~~~html
<button class="restriction">限制事件在3秒内只执行一次，其他的点击无效</button>
<input type="text" class="throttling" name="参数名称1" placeholder="限制事件在3秒内只执行一次，其他的点击无效">

<script>
    // setTimeout设置延时定时器(只执行一次)，第一参数是执行函数，第二参数是延时多久执行单位毫秒
    // 需要注意的是第一参数函数中的变量是针对与全局的变量，除非局部变量声明
    var timer = setTimeout(
        function () {
            console.log("定时器执行了");
        },
        3000
    )
    // setTimeout设置周期定时器，第一参数是执行函数，第二参数是周期时长执行单位毫秒
    // 需要注意的是第一参数函数中的变量是针对与全局的变量，除非局部变量声明
    var i = 1;
    var interval = setInterval(

        function () {
            console.log("定时器周期执行次数：", i++);
        },
        3000
    )

    // 取消定时器，参数是定时器
    clearTimeout(interval)

    // 限制事件在3秒内只执行一次，其他的点击无效
    var restrictionElement = document.querySelector(".restriction");
    // restrictionElement.onclick = debounce(e, 3000);
    restrictionElement.addEventListener("click", debounce)
    isClick = true;
    function debounce(e) {
        if (isClick) {
            isClick = false;
            //事件
            console.log(e.target.innerHTML);
            console.log('我被点击了');
            //定时器
            setTimeout(function () {
                isClick = true;
            }, 3000);//3秒内不能重复点击
        } else {
            console.log('请勿过快点击');
        }
    }

    // 限制输入框内容发生变化在5秒内只能提交执行一次
    var formOninputElement = document.querySelector(".throttling");
    isInput = true;
    formOninputElement.oninput = function (event) {
        if (isInput) {
            isInput = false;
            //事件
            console.log("提交的内容"+event.target.value);
            //定时器
            setTimeout(function () {
                isInput = true;
            }, 5000);//5秒内不能重复提交
        } else {
            console.log('5秒内不能重复提交');
        }
    }
</script>
~~~