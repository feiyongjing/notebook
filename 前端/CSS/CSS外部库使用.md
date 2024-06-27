### animate动画库使用，使用参考官网：https://animate.style
~~~html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="x-ua-compatible" content="ie=edge">
    <title>JS</title>

    <!-- curl https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css > animate.min.css 
         上述命令在bash中下载css
    -->
    <link rel="stylesheet" href="../css/animate.min.css" />
</head>

<body>
    <style>
        div {
            height: 500px;
            background-color: aquamarine;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        h2 {
            font-size: 60px;
            /* 设置动画：animate__rotateIn 去除前缀*/
            animation: rotateIn 2s 0.5s infinite alternate;
        }
    </style>

    <div>
        <h2>哈哈</h2>
    </div>
</body>

</html>
~~~




