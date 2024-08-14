# react扩展Fragment组件标签
~~~js
import React, { Fragment } from 'react'

export default function Count() {

    return (
        // jsx中要求组件只能编写一个根标签，但是实际有些情况下不希望组件包裹多余的div标签
        // 解决方案1：使用空标签，在实际渲染到页面中时会去除空标签
        // <>
        //     <div>哈哈哈</div>
        // </>

        // 解决方案2：使用Fragment标签，在实际渲染到页面中时会去除Fragment标签
        // 如果组件列表中的元素，则Fragment还支持写key属性（其他属性不支持写），而空标签就不支持写key属性
        <Fragment key="001">
            <div>哈哈哈</div>
        </Fragment>
    )
}
~~~


