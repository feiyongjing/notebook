# 组件的refs属性（通过refs属性查找对应的元素）
- 字符串形式的ref属性
- 回调函数形式的ref属性
- createRef创建ref容器

### 字符串形式的ref属性设置例子（不建议使用，在未来的react版本中会移除）
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
  <div id="test1"></div>

  <script type="text/babel">
    class MyApp extends React.Component {

      showDate1 = ()=>{
        // 获取input输入标签，也可以直接通过 this.refs.test1 获取标签ref属性是test1的真实DOM节点标签
        const {test1} = this.refs
        alert(test1.value)
      }

      showDate2 = ()=>{
        alert( this.refs.test2.value)
      }

      render() {
        return (
          <div className='title'>
              <input ref="test1" type="text" placeholder="点击按钮弹窗提示数据"/>
              <button onClick={this.showDate1} >点击提示左侧数据</button>
              <input ref="test2" onBlur={this.showDate2} type="text" placeholder="失去焦点弹窗提示数据"/>
          </div>
        )
      };

    }

    ReactDOM.render(<MyApp />, document.getElementById('test1'));
  </script>
</body>

</html>
~~~

### 回调函数形式的ref属性设置例子
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
  <div id="test1"></div>

  <script type="text/babel">
    class MyApp extends React.Component {

      showDate1 = ()=>{
        // 由于在标签上设置的ref属性是一个回调函数，该回调函数会被react自动的进行调用
        // ref回调函数的参数就是该属性所在的标签节点，而在回调函数在给组件实例对象添加属性使其赋值该标签节点
        // 这样就可以直接通过组件实例对象的属性获取到对应标签节点了
        const {test1} = this
        alert(test1.value)
      }

      saveInput = (currentNode) => {this.test2 = currentNode}

      showDate2 = ()=>{
        alert( this.test2.value)
      }

      render() {
        return (
          <div className='title'>
              <input ref={(currentNode) => {this.test1 = currentNode}} type="text" placeholder="点击按钮弹窗提示数据"/>
              <button onClick={this.showDate1} >点击提示左侧数据</button>
              <input ref={this.saveInput}  onBlur={this.showDate2} type="text" placeholder="失去焦点弹窗提示数据"/>
          </div>
        )
      };

    }

    ReactDOM.render(<MyApp />, document.getElementById('test1'));
  </script>
</body>

</html>
~~~

### createRef创建ref容器设置例子（最新的api）
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
  <div id="test1"></div>

  <script type="text/babel">
    class MyApp extends React.Component {

      // 通过在组件实例对象上绑定通过React.createRef()创建的ref容器
      // 然后直接在标签上设置ref属性值是指定的ref容器，react会自动的将ref属性所在DOM标签绑定到ref属性值所在的ref容器中的current属性上
      // 需要注意的是一个ref容器只能绑定到一个DOM标签上，如果绑定了多个则只有最后一个生效
      myRef1=React.createRef();
      myRef2=React.createRef();

      showDate1 = ()=>{
        // 通过ref容器的current属性获取DOM标签
        alert(this.myRef1.current.value)
      }

      showDate2 = ()=>{
        alert(this.myRef2.current.value)
      }

      render() {
        return (
          <div className='title'>
              <input ref={this.myRef1} type="text" placeholder="点击按钮弹窗提示数据"/>
              <button onClick={this.showDate1} >点击提示左侧数据</button>
              <input ref={this.myRef2} onBlur={this.showDate2} type="text" placeholder="失去焦点弹窗提示数据"/>
          </div>
        )
      };

    }

    ReactDOM.render(<MyApp />, document.getElementById('test1'));
  </script>
</body>

</html>
~~~

# 不要过度的使用ref属性获取DOM对象，占用资源比较大，在一些情况下有其他获取DOM对象的方式

### 通过event事件对象获取发生事件的DOM对象
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
  <div id="test1"></div>

  <script type="text/babel">
    class MyApp extends React.Component {

      myRef1=React.createRef();

      showDate1 = ()=>{
        // 通过ref容器的current属性获取DOM标签
        alert(this.myRef1.current.value)
      }

      showDate2 = (event)=>{
        // 通过event.target获取发生事件的DOM标签
        alert(event.target.value)
      }

      render() {
        return (
          <div className='title'>
              <input ref={this.myRef1} type="text" placeholder="点击按钮弹窗提示数据"/>
              <button onClick={this.showDate1} >点击提示左侧数据</button>
              <input onBlur={this.showDate2} type="text" placeholder="失去焦点弹窗提示数据"/>
          </div>
        )
      };

    }

    ReactDOM.render(<MyApp />, document.getElementById('test1'));
  </script>
</body>

</html>
~~~








