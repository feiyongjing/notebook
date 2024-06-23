# Vue如何进行事件绑定
~~~html

<div id="root">
    <!-- v-on进行事件绑定函数，v-on: 可以直接简写为@ 
         注意如果在使用时没有传递参数就会在函数内部有一个默认的event事件对象
         当然一旦传递了参数则需要手动在调用方中使用$event占位表示事件对象是那个参数
    -->
    <button v-on:click="showinfo1">点击出现弹窗</button><br/>
    <button @click="showinfo2(21, $event)">点击出现弹窗</button><br/>

    <!-- 事件修饰符：
         prevent用于阻止默认事件
         stop用于阻止事件冒泡
         once用于事件只触发一次
         captrue用于使用事件的捕获模式
         self用于只有event.target是当前元素时才触发事件
         passive用于事件的默认行为立即执行，无需等待事件绑定函数回调执行完毕
    -->
    <button @click.prevent="showinfo1">点击出现弹窗</button><br/>

    <!-- 键盘事件绑定 
         @keyup.enter中@表示v-on事件绑定，keyup表示键盘按下抬起事件，enter表示键盘的回车键，其他的键盘按键表示如下
         delete表示删除键
         esc表示退出键
         space表示空格键
         tab表示换行键(按下这个键输入框会失去焦点不会触发keyup键盘按下抬起事件，比较特殊需要配合 keydown 事件表示直接按下键盘就触发事件)
         up表示向上键
         down表示向下键
         left表示向左键
         right表示向右键
         如果这些键位表示不够可以在事件处理函数中手动获取事件对象的keyCode判断键盘按键
    -->
    <input type="text" @keyup.enter="showinfo3" style="width: 100%;" placeholder="键盘事件敲下输入文本按下回车出现弹框">
</div>


<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false
    
    var v=new Vue({
        el: "#root",     
        data: {},

        // 设置函数变量属性，可以在html中给事件绑定函数
        methods: {
            // 函数变量属性的设置是这样showinfo: function(){} （注意不要写箭头函数，箭头函数的this不确定）如下例子是简写
            // 注意如果在使用时没有传递参数就会在函数内部有一个默认的event事件对象
            // 当然一旦传递了参数则需要手动在调用方中使用$event占位表示事件对象是那个参数
            showinfo1(){
                alert("hello world")
            },

            showinfo2(num, event){
                console.log(num, event)
                alert("hello world")
            },

            showinfo3(){
                alert("hello world")
            },
        },
    })
</script>
~~~






