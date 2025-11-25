# 组件的props属性（接收组件标签上传递的数据）
~~~html
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8" />
  <title>Hello World</title>
  <!-- react.development.js 是react的核心库需要到官网下载 -->
  <!--   <script src="https://unpkg.com/react@18/umd/react.development.js"></script> -->
  <script src="./js/react18/react.development.js"></script>

  <!-- react-dom.development.js 是提供操作DOM的react库需要到官网下载 -->
  <!-- <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script> -->
  <script src="./js/react18/react-dom.development.js"></script>

  <!-- babel.min.js 解析jsx语法代码转为js需要到官网下载 -->
  <!-- <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script> -->
  <script src="./js/react18/babel.min.js"></script>

  <!-- prop-types.js 是MyApp组件标签上传递属性到组件实例对象props属性上时的类型检查限制需要使用的库 -->
  <!-- <script src="https://cdnjs.cloudflare.com/ajax/libs/prop-types/15.6.0/prop-types.js"></script> -->
  <script src="./js/react18/prop-types.js"></script>

  <style>
    .title {
      font-size: 30px;
      background-color: aqua;
    }
  </style>
</head>

<body>
  <div id="test1"></div>
  <div id="test2"></div>
  <div id="test3"></div>

  <script type="text/babel">
    class MyApp extends React.Component {

      // 如果写在类外就需要手动 MyApp.propTypes={...}
      // 现在MyApp组件标签上传递属性到组件实例对象props属性上时的类型检查限制
      static propTypes = {
        name: PropTypes.string.isRequired,  // name属性必须是字符串并且是必传选项
        age: PropTypes.number,
        sex: PropTypes.string,
        speak: PropTypes.func, // speak属性必须是函数类型
      }

      // 如果写在类外就需要手动 MyApp.defaultProps={...}
      // 设置MyApp组件标签上传递属性到组件实例对象props属性上时没有传递属性时设置该属性的默认值
      static defaultProps = {
        age: 19,
        sex: '男'
      }

      render() {
        // 读取通过在组件对象上的props属性
        // 注意props中children默认是收集的标签体内容
        const { name, age, sex, children} = this.props
        return (
          <div className='title'>
            <h3>姓名：{name}</h3>
            <h3>年龄：{age}</h3>
            <h3>性别：{sex}</h3>
            <h3>爱好：{children}</h3>
          </div>
        )
      };

    }


    // 设置自定义的标签属性，最终这些属性会存储在组件对象的props属性上，并且默认会将标签体内容收集到props属性中children上
    ReactDOM.render(<MyApp name='张三' age={15} sex='男'>篮球</MyApp>, document.getElementById('test1'));

    const user2 = { name: '李四', age: 20, sex: '男' }
    ReactDOM.render(<MyApp {...user2} children="足球" />, document.getElementById('test2'));

    const user3 = { ...user2, name: '王舞', sex: '女' , children: "羽毛球"}
    ReactDOM.render(<MyApp {...user3} />, document.getElementById('test3'));
  </script>
</body>

</html>
~~~



