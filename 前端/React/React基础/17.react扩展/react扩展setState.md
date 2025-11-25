# react扩展setState

### Count组件案例如下
~~~js
import React, { Component } from 'react'

export default class Count extends Component {

    state = { count: 0 }

    add = () => {
        
        // setState第一个参数第一种写法，写对象，对象内部有需要修改的state属性
        // setState函数的第二个参数是state修改完成之后的回调函数（一般是省略第二个参数），即setState函数修改state是异步完成的
        // this.setState({ count: this.state.count + 1 }, () => {
        //     console.log("state真正的修改完成，count值是：", this.state.count)
        // })
        // console.log("state还没有修改完成，count值是：", this.state.count)


        // setState第一个参数第二种写法，写函数，函数的返回值对象内部有需要修改的state属性
        // 写函数默认接收state和props
        this.setState((state, props)=>{
            return {count:state.count+1}
        }, () => {
            console.log("state真正的修改完成，count值是：", this.state.count)
        })
    }

    render() {

        return (
            <div>
                <h2>Count组件</h2>
                <h4>当前值是{this.state.count}</h4>
                <button onClick={this.add}>+</button>&nbsp;


            </div>
        )
    }
}
~~~



