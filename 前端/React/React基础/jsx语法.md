# jsx是什么？
- 全称Javascript XML，是react定义的一种类似于xml的JS扩展语法：js + xml

### jsx的语法规则
~~~html
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8" />
  <title>Hello World</title>
  <!-- react.development.js 是react的核心库 -->
  <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <!-- react-dom.development.js 是提供操作DOM的react库 -->
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>

  <!-- babel.min.js 解析jsx语法代码转为js -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <style>
    .title {
      font-size: 30px;
      background-color: aqua;
    }
  </style>
</head>

<body>

  <div id="root"></div>
  <!-- 注意这里的script标签type属性是text/babel，而不是text/javascript -->
  <script type="text/babel">

    const myId = "test"
    const myData = ["java", "c", "c++", "python", "go", "javascript", "typescript"]

    function MyApp() {
      // jsx语法规则
      // 1. 编写虚拟DOM的html时不需要引号包裹标签，使用括号包裹可以写多行html
      // 2. 标签混入标签属性和标签内容需要使用变量时，变量需要使用花括号{}包裹变量
      // 3. 标签中指定样式的类名不使用class属性表示，而是使用className属性表示
      // 4. 标签内部的style属性不能像正常的html编写，jsx的内联样式编写格式是 style={{key:value}}
      // 5. 编写虚拟DOM的html时只有一个根标签
      // 6. 标签首字母如果是小写，就会将标签转为正常的html标签（无法转化就报错），如果首字母是大写，react就会当成组件进行渲染
      return (
        <h1 id={myId} className='title'>
          <ol>
            {myData.map((item, index) => {
              return <li key={index}>{item}</li>
            })}
          </ol>
        </h1>
      );
    }

    const container = document.getElementById('root');
    const root = ReactDOM.createRoot(container);
    root.render(<MyApp />);

  </script>
</body>

</html>
~~~





