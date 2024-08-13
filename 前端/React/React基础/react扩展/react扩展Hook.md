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

