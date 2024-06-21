## JS的基本数据类型和对象
~~~js
// js5有六种数据类型 分别是数字、字符串、布尔、对象(object)、null、undefined 函数类型，js6有添加了Symbol和BigInt
// 基本数据类型数字、字符串、布尔如下，另外双引号和单引号包裹的都是字符串，布尔类型可以将false写成0、true写成1
// 16进制的数字是0x开头，8进制的数字是0开头，2进制的数字是0b开头
// 数字有个奇怪的值是NaN，所有参与NaN进行算术运算的值都是NaN
// 函数和变量的声明在执行代码时会被最先执行
var age = 24;
var name = "张三";
var flag = true;

// 复合数据类型对象(object)，例子如下
var user1 = {
    age: 24,
    name: "张三",
    flag: true
}
// 手动创建对象并且添加、删除、获取对象的属性
var user2 = new Object();  // 也可以写 var obj={} 
user2.age=18;
user2.flag=true;
delete user2.flag;
console.log(user2.age)    
console.log(user2["age"])   
console.log("age" in user2)   // 判断对象上是否有指定名称的属性

// 直接声明函数
function fun1(obj1, obj2){
    console.log(obj1)
    console.log(obj2)
}
fun1("哈哈1","哇哇1")

// 设置函数变量
var fun2 = function(obj1, obj2){
    console.log(obj1)
    console.log(obj2)
}
fun2("哈哈2","哇哇2")

// null和undefined是特殊类型，null一般代表对象为空，undefined一般代表数值没有
// 一般如果声明了变量但是未赋值这个变量就是undefined或者是null，大部分请求下是undefined

// typeof 用于判断基本数据类型
console.log(typeof 24)              
console.log(typeof "张三")   
console.log(typeof true)        
console.log(typeof user1)  
~~~

## JS数组
~~~js
// 创建数组对象，Array构造函数可以设置1或者多个参数，如果只有一个参数并且是数字表示的是数组的长度，多个参数表示的是数组元素的值
// 也可以使用 var a = [1,2,3] 或者 var a = Array.of(10, 20, 30) 创建数组，这样设置1或者多个参数都表示的是数组元素的值
var a = new Array() 
// 设置数组长度，如果不设置就是实际使用的最大索引作为数组长度
a.length = 10

a[3] = "哈哈"

console.log(a.length)
console.log(a.toString())
// 数组常用的方法 push、pop、unshift、shift
// push方法向数组末尾添加1或者多个元素，返回新的数组长度
// pop方法删除数组最后1个元素，返回被删除的元素
// unshift方法向数组头部添加1或者多个元素，返回新的数组长度
// shift方法删除数组头部第1个元素，返回被删除的元素
// concat方法连接两个数组，返回新的数组
// join方法设置使用指定字符串作为分隔符将数组中的元素全部放入一个字符串返回
// reverse方法翻转数组
// 

var b = ["123","xas","哈哈","456","789"]
// slice根据数组索引截取子数组返回（不影响原数组），包含第一个参数索引元素，不包含第二个参数索引元素
// 负数索引表示数组末尾（最后一个元素负数索引是-1）
var c = b.slice(1, -1)
console.log(c.toString())

// splice替换数组的子数组，并返回被替换的子数组，第一个参数表示开始位置的索引，第二个参数表示截取删除元素数量，第三个和后面参数是新的子数组元素
// 负数索引表示数组末尾（最后一个元素负数索引是-1）
var d = b.splice(1, 1)
console.log(d.toString())
console.log(b.toString())

// 读取数组，使用3个点快速读取而不用循环
console.log(...[1, 2, 3])
// 数组快速合并
console.log(...[...[1, 2, 3], ...[4, 5, 6]])

// 伪数组对象是指类似数组，但是事件不是真正的数组，例子如下
var user1 = {
    0: "李四光",
    1: "20",
    2: "男",
    length: 3,
}
console.log("打印伪数组对象user1：", user1["0"]);

// Array.from将伪数组转换成真正的数组（其实真正的数组相当与java的List集合），真正的数组拥有添加元素等方法
var user1List = Array.from(user1);
user1List.push("篮球")
console.log("打印真正的数组对象", user1List);
~~~

## JS的Set
~~~js
// Set数据结构，其实就是Java的HashSet,不能放重复的数据
var s = new Set();
Array.of(155, 155, 223, 35, 488, 599).forEach(x => s.add(x));
console.log("打印Set：", s);
// set转数组
console.log("set转数组：", Array.of(...s));
console.log("set转数组：", [...s]);
// 字符串xaaabbbccccdddd字符去重
console.log("字符串xaaabbbccccdddd字符去重：", [...new Set("xaaabbbccccdddd")].join(""));
// 字符串"5"和数字5的数据类型不同，所以它们不是重复的数据可以同时在一个Set中
console.log("字符串'5'和数字5的数据类型不同，所以它们不是重复的数据可以同时在一个Set中", ...new Set(["5", 5]));
~~~

## JS的强制类型转换
~~~js
var a = 24;
var b = "15";
var c = true;
var d = "false";

console.log(typeof String(a))   
// 正常的数字字符串可以转换为数字
// 包含非数字的字符串会转换为NaN
// 空字符串和空格字符串会转换为0 
// 布尔值false转换为0、true转换为1 
// null转换为0，undefined转换为NaN  
console.log(typeof Number(b))   
console.log(typeof String(c))    
// 数字0、NaN、空字符串、null、undefined转换为false，其他的都是true
console.log(typeof Boolean(d))  
~~~

## JS运算符
~~~js
// 比较运算符：==  ===  !=  !==
console.log('10=="10"数值比较，输出结果' + (10 == "10"))                           // 输出true
console.log('10==="10"严格比较，即比较数值也比较类型，输出结果' + (10 === "10"))    // 输出false

console.log('10!="10"数值比较，输出结果' + (10 != "10"))                           // 输出false
console.log('10!=="10"严格比较，即比较数值也比较类型，输出结果' + (10 !== "10"))    // 输出true

// 注意!取反运算只要以下6个值是true其他的都是false
console.log("false取反"+!false);
console.log("0取反"+!0);
console.log("null取反"+!null);
console.log("undefined取反"+!undefined);
console.log("空字符串取反"+!"");
console.log("NaN取反"+!NaN);
~~~

## JS循环
~~~js
// 常见循环和java循环写法一致
// 循环变量对象属性
var user = {
    age: 24,
    name: "张三",
    flag: true
}

// 循环获取的a是对象的属性名称
for(var a in user){
    console.log(user[a]);
}

var a = ["123","xas","哈哈"]
// 数组的forEach函数式编程
// 函数的第一个参数是元素值
// 函数的第一个参数是数组元素索引
// 函数的第三个参数是数组
a.forEach(function(value, index, obj){
    console.log(value)
})
~~~

## JS原型对象(其实是java的class)
~~~js
// 首字母大写的函数是构造函数
function User(){
}

// User.prototype获取class的原型对象，然后在原型对象上添加属性
// 后续所有调用User方法创建的User对象，从这些User对象上获取的原型对象(注意是所有的User共享的同一个原型对象)都可以获取到原型属性
User.prototype.a=1

var user1=new User()
var user2=new User()

// 实例对象上获取到原型对象就是Class的原型对象
console.log(user1.__proto__ === User.prototype)

// 从实例对象上调用__proto__可以获取到原型对象，然后继续获取原型对象的属性
console.log(user1.__proto__.a)
console.log(user2.__proto__.a)

// 调用实例对象上的属性
// 如果该实例没有该属性则会去它的原型对象上去取这个属性（如果找不到就继续向上找原型对象的属性，直到Object的原型也找不到就undefined）
console.log(user1.a)
console.log(user2.a)

// 修改原型对象会影响所有的实例对象
user1.__proto__.a=2
console.log(user1.__proto__.a)
console.log(user2.__proto__.a)

// 对象的hasOwnProperty可以检查是否包含指定的属性（不会检查对象的原型对象属性）
// 通过in检查对象是否包含指定的属性（会检查对象的原型对象属性）
user1.age=18
console.log(user1.hasOwnProperty("age"))
console.log(user1.hasOwnProperty("a"))
console.log("age" in user1)
console.log("a" in user1)

// hasOwnProperty方法是原型对象的原型对象携带的（如果找不到就继续向上找原型，直到Object的原型）
console.log(user1.__proto__.__proto__.hasOwnProperty("hasOwnProperty"))
~~~

## JS通过原型对象重写方法
~~~js
function User(age) {
    this.age=age
}
// 通过原型对象重写toString方法
User.prototype.toString = function () {
    return "年龄是：" + this.age;
}

var user1 = new User(10)
var user2 = new User(18)
console.log(user1.toString())
console.log(user2.toString())
~~~

## JS的this
~~~js
function User(age) {
    // 一般情况下this是指函数的调用者
    this.age=age
}

var fun1 = function(){
    return this.age
}

var a= new User(12)
var b= new User(15)

// 函数类型变量通过apply和call方法也可以调用该函数，并且传入的第一个参数在函数内部会被当成this
// call方法会直接将第一个参数之后的全部参数按照顺序交给函数对应的型参执行
// apply方法会将第二个参数（类型一定是数组），将数组中的元素按照顺序作为函数对应的型参执行
console.log(fun1.apply(a))
console.log(fun1.call(b))
~~~

## JS函数参数
~~~js
var fun1 = function () {
    // 函数没有设置参数，但是可以通过arguments伪数组获取到参数
    console.log(arguments[0])
    console.log(arguments[1])
}

fun1("张三", 18)
fun1("李四", 13)
~~~

## JS正则
~~~js
// 创建正则对象，第一个参数是正则表达式，第二个参数是匹配模式（i表示忽略大小写，g表示全局匹配）
var reg = new RegExp("a", "ig")

var a="a"

// 正则对象的test方法可以验证字符串是否匹配
console.log(reg.test(a))
~~~

## JS字符串相关
~~~js
// 遍历字符串
var username = "xxasdasdw"
console.log("遍历字符串", username);
for (const i of username) {
    console.log(i);
}

// 字符串使用反引号可以包变量
var bilibili = "https://www.bilibili.com/"
var html = `<a href="${bilibili}">跳转到bilibili</a>`

// charAt方法可以获取字符串中指定索引位置的字符
// charCodeAt方法可以获取字符串中指定索引位置字符的Unicode编码
// concat方法可以连接多个字符串，返回新的字符串
// indexOf方法可以检查字符串中是否包含指定的子字符串，从前向后查找返回该子字符串的第一次出现位置的索引，没有就返回-1，可以指定第二参数设置从哪个索引位置开始检查
// lastindexOf方法可以检查字符串中是否包含指定的子字符串，从后向前查找返回该子字符串的第一次出现位置的索引，没有就返回-1
// slice方法截取子字符串返回，包含第一个参数索引字符，不包含第二个参数索引字符，负数索引表示末尾字符（最后一个字符负数索引是-1）
// subslice方法截取子字符串返回，包含第一个参数索引字符，不包含第二个参数索引字符，不能写负数索引（写了会默认是0），如果第二参数小于第一参数则默认执行时会交换它们的位置
// subslice方法截取子字符串返回，第一个参数设置开始索引，第二参数表示截取的字符数量
// split方法参数分隔字符串拆分字符串为字符串数组，数组中的元素没有分隔的字符串
~~~

## ES6的let变量和const常量
~~~js
/**
 * var是函数级别的作用域，有变量提升，先使用后声明不会报错只是变量是undefined，并且同一作用域同名变量可以多次声明
 * let是块级左右域（花括号级别作用域），let同一作用域同名变量只可以声明一次，
 * let没有变量提升，即不会将let变量自动提升，即let变量必须先声明后使用
 */
var a = [];
for (var i = 0; i < 10; i++) {
    a[i] = function () {     // 这里只是函数赋值到数组元素并没有执行函数
        console.log(i)
    }
}
a[6](); // 会打印10，原因是由于var是函数基本作用域，所以只有一个i，在所有循环执行完时i=10，所以执行函数打印i是10

var b = [];
for (let i = 0; i < 10; i++) {
    b[i] = function () {     // 这里只是函数赋值到数组元素并没有执行函数
        console.log(i)
    }
}
b[6](); // 会打印6，原因是由于是函数基本作用域，所以有10个i，所以执行第7个函数打印i是6

// const声明常量，声明时必须赋值，const是块级左右域（花括号级别作用域），
// const同一作用域同名变量只可以声明一次，const没有变量提升，即不会将const变量自动提升
const eq = "常量"
console.log("const", eq)
~~~

## ES6的对象解构赋值和对象扩展
~~~js
var user = {
    name: "张三",
    age: 20
}
// 将user对象中的属性直接拿出来，而不是使用user.name等获取user对象的属性
var { name, age } = user;
console.log("user对象解构赋值的属性name是", name)
console.log("user对象解构赋值的属性age是", age)

// 对象的扩展
var name = "盖伦"
var ae = "age"
var user2 = {
    // name: name,
    name, // 如果是上面引用外面的变量就可以简写
    // age: "20",
    [age]: "20", // 对象的属性名引用外面的数据和上面的age: "20"是一样的效果
    // getName:function(){
    //     console.log("打印名称:",this.name);
    // }
    // 对象中的函数可以简写
    getName() {
        console.log("打印名称:", this.name);
    }
}
~~~

## ES6的箭头函数
~~~js
// 箭头函数，其实就是java的拉姆达表达式
var sum = (x, y) => x + y;
// 注意如果是返回对象需要使用圆括号包裹返回值
var sum1 = (x, y) => ({ x, y });
// 注意对于普通函数来说this是指向的函数运行时所在的对象，而箭头函数则是指向函数运行时上层作用域中的this
~~~

## 将变量与对象的属性绑定，两者变化一个另一个也会随之变化
~~~js
// 将变量与对象的属性绑定，两者变化一个另一个也会随之变化
let number = 18
let person = {
    name: "张山",
    sex: "男"
    //    age:number  使用这个方式设置对象的属性是无法随着nunber变量的变化而变化
}

Object.defineProperty(person, "age", {
    value: number,
    enumerable: true,    //控制属性是否允许枚举遍历，默认是false
    writable: true,      //控制属性是否允许被修改，默认是false
    configurable: true,  //控制属性是否允许被删除，默认是false

    // 定义该属性的getter方法，当有人读取该属性时会自动的调用这个方法获取属性的值
    // 即当number被修改后，age属性也会变化
    get() {
        console.log("有人读取了age属性")
        return number;
    },

    // 定义该属性的setter方法，当有人修改该属性时会自动的调用这个方法修改属性的值
    // 即当age属性被修改后，number也会变化
    set(value) {
        console.log("有人修改了age属性")
        number = value;
    }

})

console.log(Object.keys(person))
~~~

