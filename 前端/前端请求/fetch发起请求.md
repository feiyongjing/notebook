~~~
<button class="btn7">点我使用fetch函数发起请求</button>

// 绑定按钮触发事件发起请求
const btn7 = document.querySelector(".btn7");
btn7.onclick = function () {

    fetch('http://xxx.com/url?id=2233',{
        // 设置请求的类型
        method: "POST",
     
        // 设置请求头
        headers: {
            'Content-Type': "application/json; charset=UTF-8",
            a: 666
        },
        // 设置请求body，如果需要传递json类型的对象请搜索一下
        body: 'username=王舞&password=666666'

    }).then(response => {
        // 以字符串返回响应体信息
        // return response.text();
        // 以json对象返回响应体信息
        return response.json();
    }).then(response => {
        console.log("打印响应体的json对象",response)
    })
}


 
~~~