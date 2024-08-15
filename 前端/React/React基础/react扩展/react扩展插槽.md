# react扩展插槽
~~~js
import React, { Component, PureComponent } from 'react'


export default class A extends PureComponent {

    render() {

        return (
            <>
                <div>A组件</div>
                <hr />
                {/* 给子组件设置回调函数，用于子组件展示其他组件，并且子组件也可以通过参数传递数据给其他组件
                    这样修改回调函数就可以在子组件中展示不同的组件
                    在vue中这是通过插槽技术实现的
                 */}
                <B render={(userObj) => <C userObj={userObj} />} />
            </>
        )
    }

}


class B extends PureComponent {

    state = { name: "张三", age: 18 }

    render() {

        return (
            <>
                <div>B组件</div>
                <hr />
                {this.props.render(this.state)}
            </>
        )
    }

}



class C extends Component {
    render() {

        return (
            <>
                <div>C组件</div>
                <div>B组件中的用户名称是：{this.props.userObj.name}，年龄是：{this.props.userObj.age}</div>
            </>
        )
    }
}

~~~


