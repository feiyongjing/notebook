# Vue过滤器
~~~html
 <div id="root">
     <!-- 过滤器和Linux命令行的管道符相似，都是将管道符前的标准输出当成管道符后的标准输入 -->
     <!-- 过滤器并没有改变原始VM中的数据，并且在标签属性上只能使用v-bind单项绑定使用过滤器，而无法使用v-model双向绑定使用过滤器 -->
     <h3>现在的时间格式化后是：{{time | timeFormater("YYYY-MM-DD HH:mm:ss")}}</h3>
     <h3>现在的时间格式化后的年月日是：{{time | timeFormater | timeFormaterDate}}</h3>
 </div>


 <script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
 <script type="text/javascript" src="https://cdn.bootcdn.net/ajax/libs/dayjs/1.11.7/dayjs.min.js"></script><!-- 时间转换库-->
 <script type="text/javascript">
     Vue.config.productionTip = false

     // 设置全局的过滤器，注意必须要在所有的VM实例创建之前设置才行
     // Vue.filter('timeFormater',function(value,str="YYYY-MM-DD HH:mm:ss"){
     //     return dayjs(value).format(str)
     // })
     const timeVm = new Vue({
         el: "#root",
         data: {
             time: Date.now()
         },

         // 方法方式格式化时间
         methods: {
             getFmtTime() {
                 return dayjs(Date.now()).format("YYYY-MM-DD HH:mm:ss")
             }
         },

         // 计算属性方式格式化时间
         computed: {
             fmtTime() {
                 return dayjs(Date.now()).format("YYYY-MM-DD HH:mm:ss")
             }
         },

         // 过滤器方式格式化时间
         // 注意这是局部的过滤器只有当前Vm管理标签范围内可以使用
         filters: {
             // 过滤器的第一个参数默认是管道符前的标准输出
             // str="YYYY-MM-DD HH:mm:ss"是当该参数不传递时的默认参数
             timeFormater(value, str = "YYYY-MM-DD HH:mm:ss") {
                 return dayjs(value).format(str)
             },

             timeFormaterDate(value, str = "YYYY-MM-DD") {
                 return dayjs(value).format(str)
             }
         }
     })
 </script>
~~~


