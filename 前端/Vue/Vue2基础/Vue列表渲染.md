# Vue列表渲染
~~~html
<div id="root">
    <h3>姓名和年龄数字列表遍历</h3>
    <input type="text" placeholder="请输入姓名" v-model="keyWord">
    <button @click="sortType=1">按年龄升序排列</button>
    <button @click="sortType=2">按年龄降序排列</button>
    <button @click="sortType=0">按年龄原顺序排列</button>
    <button @click=updatepersons>点击将马冬梅替换成马老师</button>
    <ul>
        <!-- 数组遍历 -->
        <!-- 列表中的p in filPersons 代表对vue中的filPersons属性遍历，key需要绑定列表中的唯一标识ID，并且key必须要有 -->
        <!-- 如果是(a,b) in filPersons 也代表对vue中的filPersons属性遍历不过多的第二个参数是filPersons属性的index -->
        <!-- 如果不写key的绑定默认绑定index，注意使用filPersons属性的index作为key则在已有列表中除了末尾之外插入数据是会导致后面的数据错乱 -->
        <!-- 使用p of filPersons 也可以遍历vue中的filPersons属性 -->
        <li v-for="p in filPersons" :key="p.id">姓名：{{p.name}}，年龄：{{p.age}}，性别：{{p.sex}}</li>
    </ul>

    <h3>汽车对象信息遍历</h3>
    <ul>
        <!-- 对象属性遍历 -->
        <!-- 如果是(value,key) in car也代表对vue中的car属性遍历-->
        <!-- 注意第一个参数是对象的值，第二个参数是对象的属性名称 -->
        <li v-for="(value,key) in car">{{value}}:{{key}}</li>
    </ul>
    <div>姓名：{{student.name}}，性别：{{student.sex}}</div>
    <button @click=addAge>点击添加显示年龄</button>
    <button @click=delAge>点击删除显示的年龄</button>
    <div v-if="student.age">年龄：{{student.age}}</div>

    <h3>爱好</h3>
    <ul>
        <!-- 数组遍历 -->
        <li v-for="h in student.hobby">{{h}}</li>
    </ul>
    <button @click=updatehobby>点击将篮球替换成乒乓球</button>
</div>


<script src="../js/vue.js"></script> <!-- vue2.7.16的源码，你需要手动下载放入指定目录下 -->
<script type="text/javascript">
    Vue.config.productionTip = false


    const v = new Vue({
        el: "#root",

        data: {
            keyWord: '',
            sortType: 0,  // 排序方式：0代表原顺序排序、1代表生效排序，2代表降序排序
            persons: [
                { id: '001', name: "马冬梅", age: 28, sex: "女" },
                { id: '002', name: "周冬雨", age: 21, sex: "女" },
                { id: '003', name: "周杰伦", age: 24, sex: "男" },
                { id: '004', name: "温兆伦", age: 18, sex: "男" }
            ],
            // filPersons: [],
            car: {
                name: "奥迪A8",
                price: "70万RMB",
                color: "黑色"
            },
            student: {
                name: '张三',
                sex: '男',
                hobby: [
                    "篮球",
                    "跑步",
                    "电子游戏"
                ]
            }
        },

        methods: {

            updatepersons() {

                // 错误的写法
                // this.persons[0]="{ id: '001', name: '马老师', age: 28, sex: '女' }"
                // 数组中的元素不可以直接通过索引赋值直接修改，需要使用以下7个数组方法修改
                // push()、pop()、shift()、unshift()、splice()、sort()、reverse()
                // 数组方法修改正确的写法
                // this.persons.splice(0,1,"{ id: '001', name: '马老师', age: 28, sex: '女' }")
                // 但是如果数组元素是一个个对象，那么元素对象的属性是可以直接修改的，如下
                this.persons[0].name = "马老师"
            },

            // 默认没有年龄，调用方法添加年龄属性
            addAge() {
                // 方式1：给指定的VM添加属性和属性值
                Vue.set(this.student, 'age', 18)
                // 方式2：给指定的VM添加属性和属性值
                // this.$set(this.student,'age',18)
                // 注意以上两种方法都只能给VM的date属性下的对象属性里面添加属性
            },

            // 默认有年龄，调用方法删除年龄属性
            delAge() {
                // 方式1：给指定的VM删除属性
                Vue.delete(this.student, 'age')
                // 方式2：给指定的VM删除属性
                // this.$delete(this.student,'age')
                // 注意以上两种方法都只能给VM的date属性下的对象属性里面添加属性
            },


            updatehobby() {
                // 错误的写法
                // this.student.hobby[0]="乒乓球"
                // 数组中的元素不可以直接通过索引赋值直接修改，需要使用以下7个数组方法修改
                // push()、pop()、shift()、unshift()、splice()、sort()、reverse()
                // splice数组方法第一个参数定义新元素应该被添加的位置。第二个参数定义应该删除多个元素。第三个和之后的参数被当作插入的元素
                // 但是如果数组元素是一个个对象，那么元素对象的属性是可以直接修改的，如下
                // this.persons[0].name="马老师"
                this.student.hobby.splice(0, 1, "乒乓球")
            }


        },

        // 列表过滤和排序计算属性实现
        computed: {
            filPersons: {
                get() {
                    let arr = this.persons.filter((p) => {
                        return p.name.indexOf(this.keyWord) !== -1
                    })
                    // 年龄排序实现
                    if (this.sortType) {
                        arr.sort((p1, p2) => {
                            return this.sortType === 1 ? p1.age - p2.age : p2.age - p1.age
                        }
                        )
                    }
                    return arr;
                }
            },
        }

        // 列表过滤和排序watch实现
        // watch: {
        //     keyWord: {
        //         immediate: true,
        //         handler(newValue, oleValue) {
        //             let arr = this.persons.filter((p) => {
        //                 return p.name.indexOf(newValue) !== -1
        //             })

        //              // 年龄排序实现
        //              if (this.sortType) {
        //                 arr.sort((p1, p2) => {
        //                     return this.sortType === 1 ? p1.age - p2.age : p2.age - p1.age
        //                 }
        //                 )
        //             }
        //             this.filPersons = arr;
        //         }
        //     }
        // }
    })
</script>
~~~


