~~~
// 到bootcnd获取链接引入axios
<script src="https://cdn.bootcdn.net/ajax/libs/axios/1.3.6/axios.js"></script>

<button class="btn4">点我使用axios发起GET请求</button>
<button class="btn5">点我使用axios发起POST请求</button>
<button class="btn6">点我使用axios发起Ajax请求</button>


// 设置请求base路径
axios.default.baseURL = 'http://127.0.0.1:5500'

// 绑定按钮触发事件发起get请求
const btn4 = document.querySelector(".btn4");
btn4.onclick = function () {
    // get请求第一个参数是url路径，可以写http://xxx.com/url注意跨越问题，也可以写/url但是会在当前的服务中查找这个路径的接口
    // get请求第二个参数是请求头和跟在url后面的参数设置
    axios.get('/url', {
        // params是跟在url后面的参数
        params: {
            id: 444,
            name: "张三"
        },
        // 设置请求头
        headers: {
            'Content-Type': "application/x-www-form-urlencoded; charset=UTF-8",
            a: 999
        },
    }).then(
        // 和Pormise一样接收两个回调函数，一个处理成功的请求另一个处理失败的请求
        (response) => {
            console.log("请求响应状态码", response.status)
            console.log("请求响应状态字符串", response.statusText)
            console.log("请求响应全部header头信息", response.headers)
            console.log("请求响应体信息", response.data)
        },
        (error) => {
            console.log("请求失败了", error.masage);
        }
    )
}

// 绑定按钮触发事件发起post请求
const btn5 = document.querySelector(".btn5");
btn5.onclick = function () {
    // post请求第一个参数是url路径，可以写http://xxx.com/url注意跨越问题，也可以写/url但是会在当前的服务中查找这个路径的接口
    // post请求第二个参数是请求body
    // post请求第三个参数是请求头和跟在url后面的参数设置
    axios.post('/url', {
        // 设置请求body
        username: "李四",
        password: "123456"
    },
        {// params是跟在url后面的参数
            params: {
                id: 555,
            },
            // 设置请求头
            headers: {
                'Content-Type': "application/json; charset=UTF-8",
                a: 888
            },
        }).then(
            // 和Pormise一样接收两个回调函数，一个处理成功的请求另一个处理失败的请求
            (response) => {
                console.log("请求响应状态码", response.status)
                console.log("请求响应状态字符串", response.statusText)
                console.log("请求响应全部header头信息", response.headers)
                console.log("请求响应体信息", response.data)
            },
            (error) => {
                console.log("请求失败了", error.masage);
            }
        )
}

// 绑定按钮触发事件发起ajax请求
const btn6 = document.querySelector(".btn6");
btn6.onclick = function () {

    axios({
        // 请求url路径，可以写http://xxx.com/url注意跨越问题，也可以写/url但是会在当前的服务中查找这个路径的接口
        url: '/url',
        // 设置请求的类型
        method: "POST",
        // 跟在url后面的参数设置
        params: {
            id: 666,
        },
        // 设置请求头
        headers: {
            'Content-Type': "application/json; charset=UTF-8",
            a: 666
        },
        // 设置请求body
        data: {
            username: "王舞",
            password: "666666"
        }

    }).then(
        // 和Pormise一样接收两个回调函数，一个处理成功的请求另一个处理失败的请求
        (response) => {
        console.log("请求响应状态码", response.status)
        console.log("请求响应状态字符串", response.statusText)
        console.log("请求响应全部header头信息", response.headers)
        console.log("请求响应体信息", response.data)
        },
        (error) => {
          console.log("请求失败了", error.masage);
        }
    )
}
~~~