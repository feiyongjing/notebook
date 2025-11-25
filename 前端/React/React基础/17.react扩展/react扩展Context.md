# react扩展Context
- Context用于父组件方便快捷的向后代组件传递数据

# 案例如下：A组件向C组件传递数据和函数
~~~js
import React, { Component, createContext } from 'react'

// 通过createContext 创建context来存储数据，当然也可以传递参数对象表示初始化存储的数据
const UserContext = createContext({})

export default class A extends Component {


    state = { name: "张三", age: 18 }

    updateName = (name) => {
        this.setState({ name })
    }

    render() {
        const { name, age } = this.state
        return (
            <>
                <div>A组件</div>
                <div>A组件中的用户名称是：{name}，年龄是：{age}</div>
                <hr />
                {/* 通过 xxxContext.Consumer 标签包裹组件，并且设置value来向后代组件传递需要的数据 */}
                <UserContext.Provider value={{ ...this.state, updateName: this.updateName }}>
                    <B />
                </UserContext.Provider>

            </>
        )
    }
}

class B extends Component {
    render() {
        return (
            <>
                <div>B组件</div>
                <hr />
                <C />
            </>

        )
    }
}

class C extends Component {

    // 方式一：设置 static contextType 的属性是从哪个Context获取数据
    // 然后就可以直接通过从 this.context 获取Context中的数据，由于使用了this，所以只有类组件可以使用这种方式
    static contextType = UserContext

    render() {

        return (

            // <>
            //     <div>C组件</div>
            //     <div>C组件获取A组件用户名称：{this.context.name}</div>
            //     <input type='text' ref={c => this.inputNode = c} placeholder='请输入新的用户名称' />
            //     <button onClick={() => this.context.updateName(this.inputNode.value)}>点我修改用户名称</button>
            // </>

            // 方式二：直接使用 xxxContext.Consumer 标签包裹内容，在内容中可以通过 value 获取Context中的数据
            // 这种方式函数式组件和类组件都可以使用，但是就是写起来有些麻烦
            <>
                <div>C组件</div>
                <div>C组件获取A组件用户名称：
                    <UserContext.Consumer>
                        {
                            value => (
                                <>
                                    {value.name}，年龄是：{value.age}
                                    <input type='text' ref={c => this.inputNode = c} placeholder='请输入新的用户名称' />
                                    <button onClick={() => value.updateName(this.inputNode.value)}>点我修改用户名称</button>
                                </>
                            )
                        }
                    </UserContext.Consumer>
                </div>
            </>
        )
    }
}


~~~


