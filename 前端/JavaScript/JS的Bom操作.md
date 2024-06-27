# BOM简介
Bom是指浏览器对象模型，即整个浏览器窗口是一个对象

# 常见的Bom对象
- window对象表示的是整个浏览器窗口，这是一个全局对象
- Navigator对象表示的是浏览器信息，通过这个对象可以识别不同的浏览器(实际内部只有userAgent属性还有用，其他属性没有意义了)
- Location对象代表的是当前浏览器的地址栏信息，即网址输入框信息，同时可以操作这个对象进行浏览器页面跳转
- History代表的是浏览器历史记录，可以通过这个对象操作浏览器的历史记录，但是由于隐私原因这个对象无法获取到具体的历史记录
- Screen代表的屏幕信息，可以通过这个对象获取用户显示器的相关信息
- window对象下也可以获取到Navigator、Location、History、Screen这些对象

## 
~~~js
// 通过navigator.userAgent可以判断浏览器类型
console.log(navigator.userAgent)

// history.back设置浏览器页面向前跳转
history.back()

// history.back设置浏览器页面向后跳转
history.forward()

// history.go设置浏览器页面向前或者向后跳转多个页面，正数表示向前跳转，负数表示向后跳转
history.go(2)

// location代表的是当前页面的完整url，通过一些属性可以获取协议、域名、端口
// 如果斜杠location表示的url就可以跳转到其他页面
console.log(location)
location.assign("https://www.baidu.com")  // 跳转到其他页面，和直接修改location的效果是一样的
location.reload()  // 表示重新加载页面，即刷新网页，如果传递参数true表示清除缓存强制刷新
~~~

# 浏览器localStorage存储
~~~html
<div>
    <h2>浏览器localStorage存储（一般最大是5MB）的数据只有手动清除（清除浏览器缓存或者手动调用clear）才会消失</h2>
    <button onclick="saveData()">点我向浏览器localStorage写数据</button><br />
    <button onclick="getData()">点我从浏览器localStorage获取数据并弹窗</button><br />
    <button onclick="removeByDataKey()">点我从浏览器localStorage删除一个数据</button><br />
    <button onclick="clearData()">点我从浏览器localStorage清除所有数据</button><br />
</div>

<script type="text/javascript">
    function saveData() {
        localStorage.setItem("user1", '{"name":"张三", "age": 18}')
        localStorage.setItem("user2", '{"name":"李四", "age": 20}')
    }

    function getData() {
        alert(localStorage.getItem("user1")+localStorage.getItem("user2"))
    }

    function removeByDataKey() {
        localStorage.removeItem("user1")
    }

    function clearData() {
        localStorage.clear()
    }
</script>
~~~


# 浏览器localStorage存储
~~~html
<div>
    <h2>浏览器sessionStorage存储的数据会随着浏览器窗口的关闭而清除</h2>
    <button onclick="saveData()">点我向浏览器sessionStorage写数据</button><br />
    <button onclick="getData()">点我从浏览器sessionStorage获取数据并弹窗</button><br />
    <button onclick="removeByDataKey()">点我从浏览器sessionStorage删除一个数据</button><br />
    <button onclick="clearData()">点我从浏览器sessionStorage清除所有数据</button><br />
</div>

<script type="text/javascript">
    function saveData() {
        sessionStorage.setItem("user1", '{"name":"张三", "age": 18}')
        sessionStorage.setItem("user2", '{"name":"李四", "age": 20}')
    }

    function getData() {
        alert(sessionStorage.getItem("user1")+sessionStorage.getItem("user2"))
    }

    function removeByDataKey() {
        sessionStorage.removeItem("user1")
    }
    
    function clearData() {
        sessionStorage.clear()
    }
</script>
~~~















