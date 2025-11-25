# hello基础案例
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
  </head>
  <body>

    <div id="root"></div>
    
    <!-- 注意这里的script标签type属性是text/babel，而不是text/javascript -->
    <script type="text/babel">
    
      function MyApp() {
        // 注意不要写引号包裹标签，这是jsx的特殊写法
        // 使用括号包裹可以写多行html
        return (
            <h1 id='test'>
                <span>
                    Hello, world!
                </span>
            </h1>
        );
      }

      // 获取真实DOM节点   
      const container = document.getElementById('root');

      // ReactDOM.createRoot根据真实DOM创建虚拟DOM   
      const root = ReactDOM.createRoot(container);

      // 渲染虚拟DOM到页面
      root.render(<MyApp />);

    </script>
  </body>
</html>
~~~


