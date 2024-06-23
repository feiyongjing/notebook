# Vue条件渲染
~~~html
<style>
    .test0 {
        width: 50px;
        height: 50px;
        background-color: blueviolet;
    }

    .test1 {
        width: 100px;
        height: 100px;
        background-color: palegreen;
    }

    .test2 {
        width: 200px;
        height: 200px;
        background-color: orchid;
    }

    .test3 {
        width: 300px;
        height: 300px;
        background-color: cornflowerblue;
    }
</style>
<div id="root">
    <!-- 通过v-show的布尔值（也可以是表达式结果是布尔类型的表达式），true表示显示元素，false表示隐藏元素 -->
    <div class="test1" v-show="true">显示元素一</div>
    <div class="test1" v-show="false">隐藏元素一</div>


    <!-- 通过v-if或v-else-if的布尔值（也可以是表达式结果是布尔类型的表达式），true表示显示元素，false表示隐藏元素 
         v-else-if和v-else属性所在元素的前一个兄弟元素必须包含v-if或者v-else-if
         v-else-if和v-else属性都是前面的兄弟元素判断v-if或者v-else-if不成立时进行的继续判断
         注意v-if和v-else-if是由对应的布尔值，而v-else没有对应的布尔值，表示前面的判断不通过就显示当前这个元素
    -->
    <h2>n的值是{{n}}</h2>
    <button @click="changeN">点我n+1</button>
    <div :class="test" v-if="n === 1">{{n}}</div>
    <div :class="test" v-else-if="n === 2">{{n}}</div>
    <div :class="test" v-else-if="n === 3">{{n}}</div>
    <div :class="test" v-else>{{n}}</div>
</div>


<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false
    
    var v = new Vue({
        el: "#root",
        data: {
            test: "test0",
            n: 0,
        },
        methods: {
            changeN() {
                this.n += 1
                if (this.n > 3) {
                    this.test = "test0"
                } else {
                    this.test = "test" + this.n
                }
            }
        },
    })
</script>
~~~