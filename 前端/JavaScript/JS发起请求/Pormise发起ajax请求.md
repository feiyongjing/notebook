~~~javaScript
// 封装一个Promise对象用于异步发起Ajax请求
const getJSON = function (url) {
    // Promise构造器接收一个函数，函数的两个参数：resolve和reject是默认的
    // resolve函数在异步代码执行成功后手动调用，接收的参数是异步操作成功返回的结果
    // reject函数在异步代码执行失败后手动调用，接收的参数是异步操作失败返回的结果
    const promise = new Promise(function (resolve, reject) {

        const headler = function () {
            if (this.readyState !== 4) {
                // readyState存有 XMLHttpRequest 的状态。从 0 到 4 发生变化。
                // 0: 请求未初始化
                // 1: 服务器连接已建立
                // 2: 请求已接收
                // 3: 请求处理中
                // 4: 请求已完成，且响应已就绪
                return;
            }
            // status是http请求响应码
            // 请求成功手动调用resolve，并且传递请求成功返回的结果
            // 请求失败手动调用reject，并且传递失败失败返回的结果
            if (this.status === 200) {
                // 打印响应的Headers
                console.log(this.getAllResponseHeaders())
                resolve(this.response);
            } else {
                reject(new Error(this.statusText));
            }
        }
        // 创建一个Ajax请求
        const client = new XMLHttpRequest();
        // open设置请求方式、请求url
        client.open("GET", url);
        // 当XMLHttpRequest中的readyState发生变化时会调用onreadystatechange函数
        client.onreadystatechange = headler;
        // 设置响应头content-type
        client.responseType = "application/json; charset=utf-8";
        // 设置请求头
        client.setRequestHeader("accept", "application/json")
        // post请求需要添加content-type
        // client.setRequestHeader("content-type","application/json; charset=UTF-8")
        
        // onload是请求完成触发的事件，也就是回调函数
        // client.onload() = function () { console.log(JSON.parse(client.responseText)) }
        
        // 请求超时设置，超过10s就取消请求
        client.timeout=10000
        // 请求超时的回调函数
        client.ontimeout=function () { console.log("网络异常请稍后重试") }
        // 客户网络出现问题的回调函数
        client.ontimeout=function () { console.log("你的网络异常请稍后重试") }
        
        // 调用abort可以手动取消请求
        // client.abort()

        // 发送请求，如果是POST请求需要在send函数中添加body参数，当然需要设置对应的请求Header：content-type
        client.send();
        
        // 例如body传json
        // client.setRequestHeader("content-type","application/json; charset=UTF-8")
        // client.send('{"aaa":"123","bbb":"456"}');
        
        // 例如body传表单
        // client.setRequestHeader("content-type","application/x-www-form-urlencoded; charset=UTF-8")
        // client.send("aaa=123&bbb=456");
    })
    return promise;
}

// 由于大多数的网页有跨域拦截，所以这里的url要使用只能自己写一个web服务使用
// pormise.then对异步操作未来的结果进行回调处理，接收两个函数参数，
// 第一个函数接收resolve函数异步操作成功返回的结果，并对这个结果进行处理
// 第二个函数接收reject函数异步操作失败返回的结果，并对这个结果进行处理
// 无论是异步操作成功或者失败处理，两种处理内部都可以继续进行新的异步处理，而如果将新的异步处理pormise对象作为成功处理或失败处理函数的返回值，则then的返回值就是新的异步处理pormise对象，而这样做就可以在后面继续接then方法来处理新的异步处理的结果，做到外部的链式调用从而摆脱内部包裹的多层回调地狱
// 当然如果在成功处理或失败处理函数中没有进行新的的异步处理返回新的的异步处理pormise对象，这样默认会将成功处理或失败处理函数的实参作为返回值
getJSON("url").then(function (data) {
    console.log("请求成功请将响应的JSON渲染到页面上")
    console.log("请求成功json对象", data)
}, function (error) {
    console.log("请求失败json", error);
})
~~~
