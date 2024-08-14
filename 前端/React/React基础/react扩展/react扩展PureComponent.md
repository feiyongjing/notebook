# react扩展PureComponent
- 优化组件不必要的渲染，即state和props数据不被修改就不进行render渲染

# 案例：如果输入框的内容和显示的内容一致，然后进行点击修改数据，执行setState函数修改数据是不会进行无用的渲染的
~~~js
import React, { Component, PureComponent } from 'react'


export default class A extends PureComponent {


    state = { name: "张三", age: 18 }

    updateName = () => {
        this.setState({name: this.inputNode.value})
    }

    render() {
        // 开发环境下 React.StrictMode 包裹App组件进行检查会导致reder和一些生命周期在组件渲染和更新都会被调用两次，来帮助我们发现副作用错误
        console.log('A组件进行render渲染')  
        const { name, age } = this.state
        return (
            <>
                <div>A组件</div>
                <div>A组件中的用户名称是：{name}，年龄是：{age}</div>
                <input type='text' ref={c => this.inputNode = c} placeholder='请输入新的用户名称' />
                <button onClick={this.updateName}>点我修改用户名称</button>
                <hr />
                <B userObj={{name, age}}/>
            </>
        )
    }

    // 方式一
    // 通过shouldComponentUpdate判断state和props是否被修改，如果没有修改就返回false不调用reand重新渲染组件
    // 缺点是每个组件都需要写shouldComponentUpdate这个生命周期钩子函数，并且state和props中的数据多时进行比较非常麻烦
    // shouldComponentUpdate(nextProps, nextState) {
    //     console.log("旧的state",this.state)
    //     console.log("新的state",nextState)
    //     console.log("旧的props",this.props)
    //     console.log("新的props",nextProps)

    //     console.log(this.state.name!==nextState.name)
    //     return this.state.name!==nextState.name
    //   }
}


// 方式二：组件继承PureComponent，而不是原来的Component
// 这样会直接默认重写shouldComponentUpdate生命周期钩子函数，默认比较state和props它们是否是新的对象（比较新旧state和props的地址）
// 即必须是新的对象才会修改数据进行页面的渲染，如果只是修改旧的对象内部数据是不生效的
// 所以一般都是编写 this.setState({name: 'xxx'}) 这样创建新的对象进行修改数据
class B extends PureComponent {

    render() {
        console.log('B组件进行render渲染')
        const {name, age} = this.props.userObj

        return (
            <>
                <div>B组件</div>
                <div>B组件获取A组件用户名称：{name}，年龄是：{age}</div>
                
            </>
        )
    }
}
~~~


