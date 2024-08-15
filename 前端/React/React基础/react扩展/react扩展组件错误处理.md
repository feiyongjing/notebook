# react扩展组件错误处理
~~~js
import React, { Component, PureComponent } from 'react'


export default class A extends PureComponent {

    state = {
        hasError: false
    }

    // 特殊的生命周期函数，一旦后代组件出错就会触发，返回值是新的state
    static getDerivedStateFromError(error) {
        console.log("出错了", error)
        return {
            hasError: true
        }

    }

    // 特殊的生命周期函数，组件加载出错页面渲染失败后调用，一般是在这里面统计错误然后调用其他后端API通知有bug
    componentDidCatch(){
        console.log("组件加载出错，页面渲染失败")
    }

    render() {

        return (
            <>
                <div>A组件</div>
                <hr />
                {/* 判断子组件是否出错一旦出错就会显示错误标签信息或者错误组件来替代子组件的位置 */}
                {/* 注意在开发模式下错误组件信息只是一闪而过，然后显示代码错误信息，
                这是正常的，这会提醒你有错误，而在生产打包后是真正的一直显示错误组件信息 */}
                { this.state.hasError ? <h2>您的网络出错请稍后重试</h2> : <B />}
            </>
        )
    }

}


class B extends PureComponent {

    state = {
        // userObj: { name: "张三", age: 18 } 
        // 后端返回数据类型和字段错误
        ccc: ["哈哈"]
    }

    render() {

        return (
            <>
                <div>B组件</div>
                <div>B组件中的用户名称是：{this.state.xxx}，年龄是：{this.state.userObj.age}</div>
            </>
        )
    }

}
~~~


