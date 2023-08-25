
~~~typeScript
console.log('hello TypeScript')

// 设置变量的类型，在之后的使用中变量只能是指定的类型，否则编辑器报错或者是tsc命令编译报错
let a: number;

// 如果在ts中不声明变量类型，则以第一次赋值的类型为变量的类型，后续赋值其他类型变量会报错
let b = true;
// b=123

// ts可以强制限制参数的类型和数量，以及返回值的类型
function sum(m: number, n: number): number {
    return m + n
}

// 常见的类型有string、boolear、bumber、any、unknown、void、never、object、array、tuple

// any类型表示全部类型，即变量可以赋值任意类型，其实不设置类型就是隐式的any 
// 注意建议不使用any类型，原因是将any类型的变量赋值给其他类型的变量时会破坏其他变量的类型限制，导致被赋值的变量也跟着变为any类型
let d: any

// unknown类型标识未知类型，即变量的类型未知所以可以赋值任意的类型，但是与anl类型的区别是
// 将unknown类型的变量赋值直接给其他类型的变量时会直接报错，不允许将unknown类型赋值给具体类型的变量
// 或者是类型断言也可以直接赋值
let e: unknown = "de3"
let e1: string
// 当然提前对unknown类型的变量使用typeof关键字获取类型做类型判断，判断与被赋值的变量类型一致时可以赋值
// if(typeof e==='string'){
//     e1=e
// }
// 在编译器不知道的情况下对变量类型断言也可以直接赋值
// 方式一类型断言
// e1= e as string
// 方式二类型断言
// e1= <string> e

// void 表示undefined、或者是没有值
function fn(): void {
    // return
    // return undefined
}

// never 表示永远没有值，事实上就是没有值只有异常错误
function fn2(): never {
    throw new Error("报错了")
}

// object类型使用的少，原因是object对象的限制近乎于没有，
// 但是一般是使用{}包裹一些属性来限制变量对象中的属性（不能多也不能少）
// 例如如下限制变量对象必须有name属性，类型必须是字符串，而且还不能有多余的其他属性
let f: { name: string } = { name: "张三" }
let f1: { name: string, age: number } = { name: "张三", age: 18 }
// 可以使用&表示与进行连接
// let f1:{ name: string } & { age: number } ={name:"张三",age:18}
// 如果需要一些属性可有可无需要在属性后加个问号表示属性是可选的
let f2: { name: string, age?: number } = { name: "张三" }
// 如果需要一堆属性可有可无，设置[propName:string]: any 表示任意数量的属性，
// 其中propName是自定义的（写什么东西） ，而组合的[propName:string]表示任意数量任意属性名的属性，ang表示属性的值是任意类型
let f3: { name: string, [propName: string]: any } = { name: "张三" }

// 限制函数变量的参数和返回值
let fn3: (a: number, b: number) => number
fn3 = function (a: number, b: number): number {
    return a + b;
}

// 数组类型限制数组元素的类型
let g: string[] = ['1', 'a']
let g1: Array<string> = ['1', 'b']

// 元组(tuple)，其实就是固定数组长度且元素类型固定的数组
let h: [string, number]

// 枚举其实就是常量对象
enum Gender {
    Male = 0,
    Female = 1
}
console.log(Gender.Male)

// 联合类型，即变量只能赋值使用这几种类型
let y: string | number

// 设置变量的字面量常量，即该变量只能是这些常量
let x: "male" | "female"
// d=male  d可以是male和female的字符串，但是不能赋值其他的字符串数据

// 类型别名
type myType = 1 | 2 | 3 | 4 | '%'
let k: myType
// 等价于 let k= 1 | 2 | 3 | 4 | '%'


// ts与java都有类，类都是包含属性和方法，属性的getter和setter
// ts和java都有类的修饰符public、protected(当前类和子类)、private，但是不同的是java默认不加修饰符是当前包访问类、而ts则是public
// ts和java都有this、super、构造器、extends继承、abstract抽象类、interface接口、泛型，它们的作用是一样的
// ts和java都有静态属性和静态方法，它们的作用是一样的，但是ts的静态属性和方法子类也会继承
// ts和java都有常量修饰，java的常量使用final，ts类属性常量是readonly而其他常量是const修饰

class Preson {

    // 如果需要使用TS标准的getter、setter则属性名称必须是下划线开头
    // 使用TS标准的getter、setter后获取或赋值实例对象的属性时直接调用属性名会自动调用getter、和setter
    _name: string;
    
    constructor(name: string) {
        this._name = name
    }

    set name(value: string) {
        this._name = value
    }

    get name() {
        return this._name;
    }
}
let person= new Preson("张三")
// 自动调用_name的get方法
console.log(person.name)
// 自动调用_name的set方法
person.name="李四"
console.log(person.name)



~~~