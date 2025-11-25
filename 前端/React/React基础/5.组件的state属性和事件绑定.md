# 组件的state属性（数据绑定）和事件绑定

### 复杂写法（不推荐使用）
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
    // 类组件必须要求类继承 React.Component，类名就是组件名称
    class MyApp extends React.Component {

      // 构造器的作用是初始化state属性和解决原型上的函数中的this指向问题
      // props参数是组件标签上传递的自定义属性
      constructor(props) {
        super(props)
        // state属性设置需要绑定的数据
        this.state = {
          isHot: false,
          wind: "微风"
        }

        // 将原型上的changeWeather函数中的this与实例对象绑定，用于解决原型上的changeWeather函数中的this指向问题
        // 默认原型上的changeWeather函数中this指向的是undefined
        this.changeWeather = this.changeWeather.bind(this)
      }

      render() {
        const { isHot, wind } = this.state
        return (
          <div className='title'>
            <h1 onClick={this.changeWeather}>今天天气很{isHot ? "凉爽" : "炎热"}，{wind}</h1>
            <button onClick={this.changeWeather}>切换天气</button>
          </div>
        )
      };

      // 这是原型函数
      changeWeather() {
        // this.state.isHot=!this.state.isHot // 错误的写法：state属性不能直接修改
        const isHot = this.state.isHot
        this.setState({ isHot: !isHot }) // 严重注意：state属性只能通过setState函数更新（即只更新有编写的state内部属性）
      }

    }

    const container = document.getElementById('root');
    const root = ReactDOM.createRoot(container);
    root.render(<MyApp />);
  </script>
</body>

</html>
~~~


### 简化写法（推荐使用）
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
    // 类组件必须要求类继承 React.Component，类名就是组件名称
    class MyApp extends React.Component {

      // state属性设置需要绑定的数据
      state = {
        isHot: false,
        wind: "微风"
      }

      // 注意标签事件绑定的 onXxx 中on后面的首字母需要大写，
      // 这使用的不是原生的Dom事件，而是React的自定义合成事件
      // React的事件处理是委托给最外层的元素
      // 通过event.targer可以获取到发生事件的Dom元素对象
      render() {
        const { isHot, wind } = this.state
        return (
          <div className='title'>
            <h1 onClick={this.changeWeather}>今天天气很{isHot ? "凉爽" : "炎热"}，{wind}</h1>
            <button onClick={this.changeWeather}>切换天气</button>
          </div>
        )
      };

      // 事件函数都定义为箭头函数
      changeWeather = ()=> {
        // this.state.isHot=!this.state.isHot // 错误的写法：state属性不能直接修改
        const isHot = this.state.isHot
        this.setState({ isHot: !isHot }) // 严重注意：state属性只能通过setState函数更新（即只更新有编写的state内部属性）
      }

    }

    const container = document.getElementById('root');
    const root = ReactDOM.createRoot(container);
    root.render(<MyApp />);
  </script>
</body>

</html>
~~~

### 常见事件绑定
- onChange事件，这是input独有的输入数据修改事件
- checked事件，这是checkbox类型input标签的数据绑定
- onClick事件，鼠标点击事件
- onKeyDown事件，按下键盘事件
- onMouseEnter事件，鼠标进入标签事件
- onMouseLeave事件，鼠标离开标签事件







