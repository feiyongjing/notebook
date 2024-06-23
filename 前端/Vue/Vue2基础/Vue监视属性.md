# Vue监视属性
~~~html
<div id="root">
    <h2>今天的天气很{{info}}</h2>
    <button @click="changeWeather">切换天气</button>

    <h2>numbers.a的值是{{numbers.a}}</h2>
    <button @click="numbers.a++">点我a+1</button>

    <h2>numbers.b的值是{{numbers.b}}</h2>
    <button @click="numbers.b++">点我b+1</button>
</div>


<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false

    var v = new Vue({
        el: "#root",
        data: {
            isHot: true,
            numbers: {
                a: 1,
                b: 2,
            }
        },

        methods: {
            changeWeather() {
                this.isHot = !this.isHot
            }
        },

        computed: {
            info: {
                get() {
                    return this.isHot ? "凉爽" : "炎热"
                },
            }
        },

        // 设置监视属性（计算属性也可以监视）
        // 其实也可以在vue对象创建出来之后使用vue对象调用 $watch("监视属性",{监视处理设置})方法来添加 watch 监视指定属性
        watch: {
            isHot: {
                immediate: true, // 设置初始化时hendler是否调用

                // 当被监视的属性发生变化时hendler会被调用
                handler(newValue, oldValue) {
                    console.log("isHot被修改了，新的值和旧的值是：", newValue, oldValue)
                }
            },

            // 如果是属性对象内部的属性监视需要加引号包裹
            // "numbers.a": {
            //     handler(newValue, oldValue) {
            //         console.log("numbers.a被修改了，新的值和旧的值是：", newValue, oldValue)
            //     }
            // },
            numbers: {
                deep: true, // 如果是属性对象内部的全部属性开启监视需要使用deep开启深度监视多层属性，默认只检测最外层属性

                handler(newValue, oldValue) {
                    console.log("numbers或者numbers内部属性被修改了，新的值和旧的值是：", newValue, oldValue)
                }
            }
        }
    })
</script>
~~~