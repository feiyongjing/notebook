# JS的编写位置
- 直接在一些标签属性上写JS代码
- 在script标签中写JS代码
- 通过script标签引入外部JS代码(推荐写在body标签的末尾)

### 方式一：直接在一些标签属性上写JS代码
~~~html
<button onclick="alert('hello world');">点击按钮</div>
~~~

### 方式二：在script标签中写JS代码
~~~html
<div>js打印日志到浏览器的console控制台中和弹窗显示hello world</div>
<script>
    // 打印日志到浏览器的控制台中
    console.log("hello world");
    // 弹窗显示hello world
    alert("hello world");
</script>
~~~

### 方式三：通过script标签引入外部JS代码(推荐写在body标签的末尾)
~~~html
<body>
    <div>js打印日志到浏览器的console控制台中和弹窗显示hello world</div>
    <script src="../js/hello.js"></script>
</body>
~~~
~~~js
// 打印日志到浏览器的控制台中
console.log("hello world");
// 弹窗显示hello world
alert("hello world");
~~~


