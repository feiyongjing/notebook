# react扩展Hook
- 主要目的是能在函数式组件中使用state和其他react特性


### stateHook使用
- 解决了函数式组件获取和修改state
~~~js
import React,{useState} from 'react'

export default function Count() {

    // useState函数初始化state数据，注意是state数据其中的一个属性
    // 参数是这个属性的初始化值
    // 返回值是个数组，有两个元素，第一个元素是state数据的这个属性值，第二个参数是修改state数据中这个属性的函数
    const [count, setStateCount]=useState(0)
    const [name, setStateName]=useState("张三")

    const add =()=>{
        // 第一种修改方式：参数不是函数，直接将参数作为修改值
        // setStateCount(count+1)

        // 第一种修改方式：参数是函数，函数的参数默认是元数据，将函数的返回值作为修改值
        setStateCount( oldCount => oldCount+1)
    }

    const updateName =()=>{
        setStateName("李四")
    }

    return (
        <div>
            <h2>Count组件</h2>
            <h4>名称：{name}</h4>
            <button onClick={updateName}>名称变为李四</button>&nbsp;
            <h4>当前值是{count}</h4>
            <button onClick={add}>值加1</button>&nbsp;

        </div>
    )
}
~~~

### effectHook使用
- 解决了函数式组件生命周期的设置
~~~js
import React, { useState, useEffect } from 'react'
import ReactDOM from 'react-dom/client';

import {root} from '../../index'

export default function Count() {

    const [count, setStateCount] = useState(0)

    const add = () => { setStateCount(count + 1) }

    const death = (event) => {
        // 卸载组件，react18之后无法卸载单独的组件了，而是直接卸载根节点
        // ReactDOM.unmountComponentAtNode(document.getElementById('test'))
        root.unmount() 
      }

    // useEffect函数接收两个参数，第一个参数是回调函数，第二个参数是个数组
    // 注意回调函数被调用的时机是由第二个参数决定的
    // 第二个参数是设置监视哪些state中的属性，例如监视count
    // 如果第二个参数不传是监视state中的全部属性，但是如果传递空数组是不监视state中的任何属性
    // 如果第二个参数是监视state中的属性，回调函数相当于 componentDidMount 和 componentDidUpdate 生命周期钩子的结合
    // 如果第二个参数传递空数组是不监视state中的任何属性，回调函数相当于 componentDidMount 生命周期钩子的结合
    // 另外第一个参数回调函数返回的函数是 componentWillUnmount 生命周期钩子函数
    useEffect(() => {

        console.log("哈哈")

        // 开启定时器
        let timer = setInterval(() => { setStateCount(count + 1) }, 1000)

        return ()=>{
            console.log('组件即将解除时的挂载生命周期钩子函数执行')
            // 清除定时器
            clearInterval(timer)
        }
    }, [count])

    return (
        <div id="test">
            <h2>Count组件</h2>
            <h4>当前值是{count}</h4>
            <button onClick={add}>值加1</button>&nbsp;
            <button onClick={death}>点我卸载组件</button>
        </div>
    )
}
~~~

### refHook使用
- 解决了函数式组件获取标签
~~~js
import React, { useRef,createRef } from 'react'

export default function Count() {

    // 调用useRef函数或者createRef函数都会返回一个ref容器
    const myRef = useRef()
    // const myRef = createRef()
    
    const show =()=>{
        // 从ref容器取出标签内容
        alert(myRef.current.value)
    }

    return (
        <div id="test">
            {/* 标签绑定ref容器 */}
            <input type='text' ref={myRef} placeholder='请输入' />
            <button onClick={show}>点我弹窗显示值</button>
        </div>
    )
}
~~~







