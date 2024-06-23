# Vue计算属性
~~~html
<div id="root">
    姓：<input type="text" placeholder="请输入姓" v-model="firstname"><br/>
    名：<input type="text" placeholder="请输入名" v-model="lastname"><br/>
    method计算属性：<input type="text" placeholder="请输入姓名" v-model="fullname1()"><br/>
    computed计算属性：<input type="text" placeholder="请输入姓名" v-model="fullname2"><br/>
</div>

<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false

    var v = new Vue({
        el: "#root",
        data: {
            firstname: "张",
            lastname: "三"
        },

        methods: {
            // 通过methed计算属性，缺点是多次调用，并且无法做到修改时同步其他属性
            fullname1() {
                return this.firstname + "-" + this.lastname
            },
        },

        // 设置计算属性
        computed: {
            fullname2: {
                // 设置fullname2属性被读取时需要调用的get方法，并且有缓存
                // 只有初次读取和内部依赖的其他属性被修改之后get方法才会被调用，其他情况下会使用缓存
                get() {
                    return this.firstname + "-" + this.lastname
                },

                // 设置fullname2属性被修改时需要调用的set方法
                set(value) {
                    var arr = value.split("-")
                    this.firstname = arr[0]
                    this.lastname = arr[1]
                }
            }
        }
    })
</script>
~~~