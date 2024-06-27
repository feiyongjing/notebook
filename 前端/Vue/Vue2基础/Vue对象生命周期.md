# Vue对象生命周期
- 八个生命周期：beforeCreate、created、beforeMount、mounted、beforeUpdate、updated、beforeDestroy、destroyed
~~~html
<div id="root">
    <!-- 注意这里的{opacity}是简写，原来的css应该是{opacity:1}，由于与Vue对象实例中的属性名称一致所有可以简写 -->
    <h3 :style="{opacity}">欢迎学习Vue</h3>
    <button @click="stop">点我停止变化</button>
</div>


<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false

    let v = new Vue({
        el: "#root",
        data: {
            opacity: 1,
        },

        methods: {
            stop() {
                // 调用Vue对象实例的该方法Vue对象实例就会自杀
                this.$destroy()
            }
        },

        // 数据代理还未开始，无法通过Vue对象实例获取data中的数据，methods中的方法
        beforeCreate() {
            console.log("beforeCreate")
            // console.log(this)
            // 卡断点
            // debugger
        },

        // 数据监测和数据代理之后，可以通过Vue对象实例获取data中的数据，methods中的方法
        created() {
            console.log("created")
            // console.log(this)
            // 卡断点
            // debugger
        },

        // 已经生成了虚拟DOM，但是在beforeMount中对DOM的操作都是操作的真实DOM但是这个真实DOM最终会被内存中的虚拟DOM覆盖导致不生效
        beforeMount() {
            console.log("beforeMount")
            // console.log(this)
            // 卡断点
            // debugger
        },

        // 常用的生命周期：Vue完成模板的解析并把初始的真实DOM元素放入页面后（挂载完毕）调用mounted，此时操作的真实DOM是生效的
        mounted() {
            console.log("mounted")
            setInterval(() => {
                this.opacity -= 0.01
                if (this.opacity <= 0) this.opacity = 1
            })
        },

        // 数据发生更新时调用，但是此时数据是新的，但是页面上的数据还是是旧的
        beforeUpdate() {
            console.log("beforeUpdate")
            // console.log(this)
            // 卡断点
            // debugger
        },

        // 数据更新完成后调用，但是此时数据和页面上的数据都是新的
        updated() {
            console.log("updated")
            // console.log(this)
            // 卡断点
            // debugger
        },

        // 常用的生命周期：当Vue对象实例调用$destroy()后会调用beforeDestroy，此时Vue对象实例的data、methods都是可用的，
        // 一般在此阶段关闭定时器、取消订阅消息、解绑自定义事件等收尾操作
        // 在beforeDestroy调用Vue对象实例的data、methods修改数据是生效的，
        // 但是由于已经进入销毁流程了，之后不会调用beforeUpdate、updated、mounted所以数据不会同步到页面了
        beforeDestroy() {
            console.log("beforeDestroy")
            // console.log(this)
            clearInterval(this.timer)
            // 卡断点
            // debugger
        },

        // Vue对象实例销毁完毕后调用destroyed
        destroyed() {
            console.log("destroyed")
            // 卡断点
            // debugger
        },
    })
</script>
~~~








