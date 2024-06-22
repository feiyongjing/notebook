# Vue如何做数据绑定
~~~html
<div id="root">
    <!-- 元素内容可以通过{{}}包裹Vue对象data中的属性双向绑定或者js表达式 -->
    <h1 >hello, {{name}}</h1>


    <!-- 元素的属性可以通过v-bind绑定属性值是Vue对象data中的属性或者js表达式，v-bind:可以简写为: -->
    <a v-bind:href="url">去百度</a>
    <!-- 注意v-bind是单向数据绑定，即data中的属性值发生变化影响元素属性值，元素属性值发生变化不影响data中的属性值 -->
    <input type="text" placeholder="单向数据绑定" v-bind:value="bind_test1" >
    <!-- v-model可以实现元素属性值和data中的属性值双向绑定
         但是v-model只能应用在表单类的元素上并且默认就是绑定的value属性值 
         所以v-model:value可以直接简写为 v-model
    -->
    <input type="text" placeholder="双向数据绑定" v-model:value="bind_test2" >
</div>


<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false
    // 常见vue对象
    var v=new Vue({
        // el属性是css选择器，只能选择绑定一个元素，不能绑定多个元素（当出现多个元素时只有第一个元素绑定成功）
        // 其实也可以在vue对象创建出来之后使用vue对象调用 $mount("#root")方法来指定 el 属性
        el: "#root",     
        // data提供el绑定元素的数据，包含它的后代元素
        // 其实dat也可以是一个函数，依靠该返回值靠提供数据（注意不要写箭头函数，这样它就不是vue对象的函数了）
        data: {          
            name: "张三",
            url: "https://www.baidu.com",
            bind_test1: "",
            bind_test2: ""
        }
    })
</script>
~~~


