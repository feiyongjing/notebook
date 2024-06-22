~~~html
<button class="btn1">点我使用JQuery发起GET请求</button>
<button class="btn2">点我使用JQuery发起POST请求</button>
<button class="btn3">点我使用JQuery发起Ajax请求</button>

// 到bootcnd获取链接引入JQuery
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.1/jquery.js"></script>

<script>
    // 绑定选择器对应的标签事件
    $('.btn1').click(function () {
        // 发送GET请求
        // get方法第一个参数是url路径
        // get方法第二个参数是请求携带的参数，是直接?a=100&b=200跟在url后面，
        // 默认是application/x-www-form-urlencoded传递参数，
        // 如果传递的是json需要使用Jquery发起ajax请求设置JSON.stringify()将对象转字符串和"Content-Type":"application/json;charset=UTF-8"
        // get方法第三个参数是请求成功后的回调函数，回调函数的参数是响应的body
        // post方法第四个参数是响应体的数据类型，默认不传是String
        $.get('url', { a: 100, b: 200 }, function (data) {
            console.log("请求成功请将响应的JSON渲染到页面上")
            console.log("请求成功json对象", data)
        },'json')
    })

    // 绑定选择器对应的标签事件
    $('.btn2').click(function () {
        // 发送POST请求
        // post方法第一个参数是url路径
        // post方法第二个参数是请求携带的参数，
        // 默认是application/x-www-form-urlencoded传递参数，
        // 如果传递的是json需要使用Jquery发起ajax请求设置JSON.stringify()将对象转字符串和"Content-Type":"application/json;  charset=UTF-8"
        // post方法第三个参数是请求成功后的回调函数，回调函数的参数是响应的body
        // post方法第四个参数是响应体的数据类型，默认不传是String
        $.post('url', { a: 222, b: 333 }, function (data) {
            console.log("请求成功请将响应的JSON渲染到页面上")
            console.log("请求成功json对象", data)
        }, 'json')
    })

    // 绑定选择器对应的标签事件
    $('.btn3').click(function () {
        // 发送ajax请求
        // ajax方法第一个参数是url路径
        // ajax方法第二个参数是请求携带的参数
        // ajax方法第三个参数是请求成功后的回调函数，回调函数的参数是响应的body
        // ajax方法第四个参数是响应体的数据类型，默认不传是String
        $.ajax({
            // 设置url路径
            url: 'url',
            // 设置请求携带的参数，默认是application/x-www-form-urlencoded传递参数
            // 如果传递的是json需要使用JSON.stringify()将对象转字符串
            data: JSON.stringify({ a: 888, b: 999, c: { d: 123 } }),
            // 设置请求的类型
            type: 'POST',
            // 设置响应体的数据类型
            dataType: 'json',
            // 设置请求超时时间10s
            timeout: 10000,
            // 设置请求Header，注意中划线使用引号包裹
            headers: {
                "Content-Type": "application/json; charset=UTF-8"
            },
            // 设置contentType，当然在Header中设置是一样的
            // contentType: 'application/json',

            // 设置请求成功的回调函数
            success: function (data) {
                console.log("请求成功请将响应的JSON渲染到页面上")
                console.log("请求成功json对象", data)
            },
            // 设置请求失败的回调函数
            error: function () {
                console.log("请求出错了")
            },
        })
    })
</script>
~~~