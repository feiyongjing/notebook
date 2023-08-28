## while循环语句
~~~java
while(循环条件){
     循环语句；
}
~~~
---


## do {} while 循环语句
注意do {} while 循环语句第一次进入一定会执行循环语句然后在判断循环条件；
~~~java
do{
    循环语句；
}while(循环条件)；
~~~
---


## for循环语句
~~~java
for(int i=0;     //任何语句，包括空语句，进行初始化
i<10;               //返回boolean值，包括空语句，循环条件
i++;                 //任何语句，包括空语句，在每一次的循环执行循环语句之后执行的操作
){
    循环语句
}
~~~
---

## foreach循环语句
~~~java
for(T s : Itareble<T>的实现对象  ){
    循环语句
}
~~~
---

## break关键字
跳出包裹当前break的第一层循环

## continue关键字
跳过包裹当前continue的第⼀层循环中continue之后的其余语句，继续执行下一次循环

## 标号label，适用于break和continue
　　标号提供了一种简单的break语句所不能实现的控制循环的方法，当在循环语句中碰到break时，不管其它控制变量，都会终止。但是，当你嵌套在几层循环中想退出循环时又怎么办呢？break只退出一重循环，但你可以用标号label标出你想退出哪一个语句。规定标号label必需放在循环之前（意味着循环必需紧跟着标号）
~~~java
label:      //label是标号的命名
for(int i=0;i<100;i++){

    while(i<100){
        if(i==50){
            break label; //跳跃到标号
        }
    }

    System.out.println(i);
}
~~~
---

# switch语句

### 可以switch哪些东⻄？ 
- 基本数据类型
- enum 枚举
- String （JDK7+）

### switch的穿透
case 语句中的值的数据类型必须与变量的数据类型相同，而且只能是字符串常量或者常量

~~~java
switch(expression){
    case 1 : //expression的值是1执行语句1
          break;//语句1，如果语句中没有break语句则会穿透进行case 2的判断
    case 2 : //expression的值是2执行语句2
          break; //语句2
    //你可以有任意数量的case语句
    default :   //以上语句都不成功执行语句3
         break;  //语句3
}
~~~